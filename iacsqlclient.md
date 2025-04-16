
#!/bin/bash
set -euo pipefail

# ------------------------------------------------------------------------------
# Script: snapshot_restore.sh
#
# Description:
#   This script automates the process of:
#     1. Finding the latest RDS snapshot for a given environment (DB instance ID).
#     2. Updating a specified CloudFormation stack with the latest snapshot ID.
#     3. Waiting for the stack update to complete.
#     4. Retrieving the RDS instance ID from the stack outputs.
#     5. Waiting for the RDS instance to be in an 'available' state.
#     6. Renaming tenant databases on the RDS instance based on provided mappings.
#
# Usage:
#   ./snapshot_restore.sh \
#     --env <DBInstanceIdentifier> \
#     --stack <CloudFormationStackName> \
#     --rename <OLD1:NEW1,OLD2:NEW2,...>
#
# Example:
#   ./snapshot_restore.sh \
#     --env qat-cdb \
#     --stack oracle-rds-stg-instance \
#     --rename QAUS:SAUS,QTECH:STECH
#
# Requirements:
#   - AWS CLI with access to RDS and CloudFormation
#   - jq (for parsing stack parameters)
#   - Bash 3.2+ (works on macOS)
#   - gdate (if on macOS; provided via 'coreutils')
#
# Notes:
#   - The script exits immediately if any command fails.
#   - All renames are applied sequentially after the instance becomes available.
# ------------------------------------------------------------------------------

echo "Script started at: $(date)"

# Default to empty
SRC_ENV=""
STACK_NAME=""
TENANT_RENAMES_LIST=""

# Parse arguments
while [[ $# -gt 0 ]]; do
  case $1 in
    --srcenv)
      SRC_ENV="$2"
      shift 2
      ;;
    --stack)
      STACK_NAME="$2"
      shift 2
      ;;
    --rename)
      TENANT_RENAMES_LIST="$2"
      shift 2
      ;;
    *)
      echo "Unknown argument: $1"
      exit 1
      ;;
  esac
done

# Validate required inputs
if [[ -z "$SRC_ENV" || -z "$STACK_NAME" || -z "$TENANT_RENAMES_LIST" ]]; then
  echo "Usage: $0 --srcenv <SRC_ENV> --stack <STACK_NAME> --rename <OLD1:NEW1,OLD2:NEW2,...>"
  exit 1
fi

# Convert comma-separated list to array
IFS=',' read -r -a TENANT_RENAMES <<< "$TENANT_RENAMES_LIST"

# Disable the AWS CLI Pager
export AWS_PAGER=""

# Find the latest snapshot
LATEST_SNAPSHOT=""
LATEST_SNAPSHOT_TIME=""
LATEST_TIME=0

DATE_CMD=$(command -v gdate || command -v date)

# Get snapshots in a safe line-by-line format
SNAPSHOTS=$(aws rds describe-db-snapshots \
  --db-instance-identifier "$SRC_ENV" \
  --query "DBSnapshots[*].[DBSnapshotIdentifier,SnapshotCreateTime]" \
  --output text)

# Read line by line
while IFS=$'\t' read -r SNAP_ID SNAP_TIME_HUMAN; do

  # Skip if SNAP_TIME_HUMAN is "None" or empty
  if [[ -z "$SNAP_TIME_HUMAN" || "$SNAP_TIME_HUMAN" == "None" ]]; then
    continue
  fi

  SNAP_TIME=$($DATE_CMD -d "$SNAP_TIME_HUMAN" +%s 2>/dev/null)

  # Skip if date command failed (invalid date format)
  if [[ -z "$SNAP_TIME" ]]; then
    continue
  fi

  # DEBUG: List the SNAPS available being searched for latest one
#  echo "$SNAP_ID,$SNAP_TIME_HUMAN,$SNAP_TIME"

  if (( SNAP_TIME > LATEST_TIME )); then
    LATEST_TIME=$SNAP_TIME
    LATEST_SNAPSHOT=$SNAP_ID
    LATEST_SNAPSHOT_TIME=$SNAP_TIME_HUMAN
  fi
done <<< "$SNAPSHOTS"

echo "Latest RDS snapshot for $SRC_ENV is: $LATEST_SNAPSHOT at $LATEST_SNAPSHOT_TIME"

# Parameters to update in the stack
PARAM_TO_UPDATE="DBSnapshotIdentifier"
PARAM_VALUE="$LATEST_SNAPSHOT"

# Get current parameters and convert them to CLI-compatible format
PARAMS=$(aws cloudformation describe-stacks --stack-name "$STACK_NAME" \
  --query "Stacks[0].Parameters" --output json | \
  jq -r --arg key "$PARAM_TO_UPDATE" --arg val "$PARAM_VALUE" \
    '[.[] |
      if .ParameterKey == $key then
        {ParameterKey: .ParameterKey, ParameterValue: $val}
      else
        {ParameterKey: .ParameterKey, UsePreviousValue: true}
      end]')

# Convert JSON to CLI args
PARAM_CLI_ARGS=$(echo "$PARAMS" | jq -r '.[] |
  if .UsePreviousValue then
    "ParameterKey=\(.ParameterKey),UsePreviousValue=true"
  else
    "ParameterKey=\(.ParameterKey),ParameterValue=\(.ParameterValue)"
  end' | paste -sd " " -)

echo "Updating stack $STACK_NAME with $PARAM_TO_UPDATE=$PARAM_VALUE..."

aws cloudformation update-stack \
  --stack-name "$STACK_NAME" \
  --use-previous-template \
  --parameters $PARAM_CLI_ARGS \
  --capabilities CAPABILITY_NAMED_IAM

if [ $? -ne 0 ]; then
  echo "Stack update command failed. Exiting."
  exit 1
fi

echo "Waiting for stack update to complete..."
aws cloudformation wait stack-update-complete \
  --stack-name "$STACK_NAME"

if [ $? -eq 0 ]; then
  echo "✅ Stack update completed successfully."
else
  echo "❌ Stack update failed or was rolled back."
  exit 1
fi

NEW_INSTANCE_ID=$(aws cloudformation describe-stacks \
			--stack-name "$STACK_NAME" \
			--query "Stacks[0].Outputs[?OutputKey=='RDSInstanceId'].OutputValue" \
			--output text)


for PAIR in "${TENANT_RENAMES[@]}"; do
  OLD_TENANT="${PAIR%%:*}"
  NEW_TENANT="${PAIR##*:}"
  echo "Renaming tenant DB from $OLD_TENANT to $NEW_TENANT on instance $NEW_INSTANCE_ID..."

  aws rds modify-tenant-database --region ap-southeast-2 \
    --db-instance-identifier "$NEW_INSTANCE_ID" \
    --tenant-db-name "$OLD_TENANT" \
    --new-tenant-db-name "$NEW_TENANT"

  if [ $? -ne 0 ]; then
    echo "❌ Failed to rename $OLD_TENANT to $NEW_TENANT"
    exit 1
  fi

  echo "Waiting for RDS instance $NEW_INSTANCE_ID to become available..."

  sleep 25

  aws rds wait db-instance-available \
    --region ap-southeast-2 \
    --db-instance-identifier "$NEW_INSTANCE_ID"

  if [ $? -eq 0 ]; then
    echo "✅ RDS instance is available."
  else
    echo "❌ RDS instance is not available after waiting. Exiting."
    exit 1
  fi
done

echo "Script completed at: $(date)"

===============refresh/execute_sql_scripts.sh

version: 0.2

env:
  variables:
    ORACLE_HOME: "/opt/oracle/instantclient"
    LD_LIBRARY_PATH: "/opt/oracle/instantclient"
    TNS_ADMIN: "/opt/oracle/instantclient"
    #PATH: "/opt/oracle/instantclient:$PATH"
    ORACLE_VERSION: "19.18.0.0.0"
    ORACLE_ZIP_BASIC: "instantclient-basic-linux.x64-19.18.0.0.0dbru.zip"
    ORACLE_ZIP_SQLPLUS: "instantclient-sqlplus-linux.x64-19.18.0.0.0dbru.zip"
    ORACLE_ZIP_TOOLS: "instantclient-tools-linux.x64-19.18.0.0.0dbru.zip"
    ORACLE_DOWNLOAD_URL: "https://download.oracle.com/otn_software/linux/instantclient/1918000"

phases:

  install:
    commands:
      - set
      - ls -lRt
      - aws sts get-caller-identity
      # Install required packages
      - echo "Installing required packages..."
      - yum install -y libaio unzip wgetgcc
      # python3-devel

      # Create Oracle directories
      - mkdir -p $ORACLE_HOME
      - mkdir -p $TNS_ADMIN

      # Download and install Oracle Instant Client
      - echo "Downloading Oracle Instant Client..."
      - cd /tmp
      - wget -q $ORACLE_DOWNLOAD_URL/$ORACLE_ZIP_BASIC
      - wget -q $ORACLE_DOWNLOAD_URL/$ORACLE_ZIP_SQLPLUS
      - wget -q $ORACLE_DOWNLOAD_URL/$ORACLE_ZIP_TOOLS

      # Extract Oracle Instant Client
      - echo "Extracting Oracle Instant Client..."
      - unzip -q $ORACLE_ZIP_BASIC -d /opt/oracle
      - unzip -q $ORACLE_ZIP_SQLPLUS -d /opt/oracle
      - unzip -q $ORACLE_ZIP_TOOLS -d /opt/oracle
      - mv /opt/oracle/instantclient_19_18/* $ORACLE_HOME/
      - rm -rf /opt/oracle/instantclient_19_18

      # Configure Oracle client
      - echo "Configuring Oracle client..."
      - echo $ORACLE_HOME > /etc/ld.so.conf.d/oracle-instantclient.conf
      - ldconfig

      # Verify installation
      - echo "Verifying SQL*Plus installation..."
      - $ORACLE_HOME/sqlplus -version

  build:
    commands:
      # Test SQL*Plus connectivity (optional - uncomment if needed)
      - echo "Testing SQL*Plus connectivity..."
      - echo "exit" | sqlplus username/password@SAUS.ITSMYHOME.NET.AU

  post_build:
    commands:
      - echo "Build completed successfully"

      =============>refresh/execute_sql_scripts.sh

      #!/bin/bash

# Configuration
SCRIPTS_DIR="./sql"
SECRET_NAME="arn:aws:secretsmanager:ap-southeast-2:977099011956:secret:/rds/oracle/stg-cdb/credentials-yY0uYw"  # Replace with your AWS Secrets Manager secret name
REGION="ap-southeast-2"              # Replace with your AWS region

# Function to display usage information
usage() {
    echo "Usage: $0 [options]"
    echo "Options:"
    echo "  -b, --db DBNAME        Database Service Name to connect with"
    echo "  -d, --directory DIR    Directory containing SQL scripts (default: ./sql)"
    echo "  -s, --secret NAME      AWS Secrets Manager secret name"
    echo "  -r, --region REGION    AWS region (default: us-east-1)"
    echo "  -h, --help             Display this help message"
    exit 1
}

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case "$1" in
        -b|--db)
            DB_NAME="$2"
            shift 2
            ;;
        -d|--directory)
            SCRIPTS_DIR="$2"
            shift 2
            ;;
        -s|--secret)
            SECRET_NAME="$2"
            shift 2
            ;;
        -r|--region)
            REGION="$2"
            shift 2
            ;;
        -h|--help)
            usage
            ;;
        *)
            echo "Unknown option: $1"
            usage
            ;;
    esac
done

# Validate required parameters
if [ -z "$SECRET_NAME" ]; then
    echo "Error: Secret name is required"
    usage
fi

# Check if scripts directory exists
if [ ! -d "$SCRIPTS_DIR/$DB_NAME" ]; then
    echo "Error: Scripts directory '$SCRIPTS_DIR/$DB_NAME' does not exist"
    exit 1
fi

echo "Retrieving database credentials from AWS Secrets Manager..."
SECRET_JSON=$(aws secretsmanager get-secret-value \
    --secret-id "$SECRET_NAME" \
    --region "$REGION" \
    --query SecretString \
    --output text)

# Check if secret retrieval was successful
if [ $? -ne 0 ]; then
    echo "Error: Failed to retrieve secret from AWS Secrets Manager"
    exit 1
fi

# Extract database credentials from the secret
DB_USERNAME=$(echo $SECRET_JSON | jq -r '.username // .user // .dbuser')
DB_PASSWORD=$(echo $SECRET_JSON | jq -r '.password // .pass')
DB_HOST=$(echo $SECRET_JSON | jq -r '.host // .hostname // .endpoint')
DB_PORT=$(echo $SECRET_JSON | jq -r '.port // "1521"')

# Validate extracted credentials
if [ -z "$DB_USERNAME" ] || [ -z "$DB_PASSWORD" ] || [ -z "$DB_HOST" ] ]; then
    echo "Error: Failed to extract all required database credentials from the secret"
    echo "Make sure your secret contains username, password, host, and sid fields"
    exit 1
fi

# Create a connection string for SQLPlus
CONNECTION_STRING="${DB_USERNAME}/${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_SID}"

echo "Executing SQL scripts from directory: $SCRIPTS_DIR/$DB_SID"

# Count total scripts for progress reporting
TOTAL_SCRIPTS=$(find "$SCRIPTS_DIR/$DB_SID" -name "*.sql" | wc -l)
CURRENT_SCRIPT=0

# Process each SQL script in the directory
find "$SCRIPTS_DIR" -name "*.sql" -type f | sort | while read -r script; do
    CURRENT_SCRIPT=$((CURRENT_SCRIPT + 1))
    SCRIPT_NAME=$(basename "$script")

    echo "[$CURRENT_SCRIPT/$TOTAL_SCRIPTS] Executing: $SCRIPT_NAME"

    # Create a temporary file with WHENEVER SQLERROR EXIT SQL*Plus directive
    TEMP_SQL=$(mktemp)
    cat > "$TEMP_SQL" << EOF
WHENEVER SQLERROR EXIT SQL.SQLCODE
SET ECHO ON
SET FEEDBACK ON
SET SERVEROUTPUT ON SIZE 1000000
SET TIMING ON
SPOOL ${script}.log
-- Original script: $SCRIPT_NAME
$(cat "$script")
SPOOL OFF
EXIT
EOF

    # Execute the script using SQLPlus
    $ORACLE_HOME/sqlplus -S "$CONNECTION_STRING" @"$TEMP_SQL"

    # Check the exit status
    if [ $? -eq 0 ]; then
        echo "✓ Successfully executed: $SCRIPT_NAME"
    else
        echo "✗ Error executing: $SCRIPT_NAME"
        echo "Check ${script}.log for details"
        exit 1
    fi

    # Clean up temporary file
    rm -f "$TEMP_SQL"
done

echo "All SQL scripts executed successfully!"
exit 0


====>
refresh/sql/STECH/1_stech_refresh_change_user_passwords_new.sql
alter user DB_DEPLOYER identified by values 'S:435AF696F519AD836EF94Cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx1271281E957712D7D87';

refresh/buildspec-sqlplus.yml

refresh/buildspec-sqlplus.yml
