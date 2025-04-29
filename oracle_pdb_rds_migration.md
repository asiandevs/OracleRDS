#  Oracle PDB to AWS RDS Migration Runbook

---

```
+-----+----------------------------------------+-----------------------------------------------------------+------------------------------------------------+
| Step| Action                                 | Commands/Scripts                                           | Comments                                       |
|-----+----------------------------------------+-----------------------------------------------------------+------------------------------------------------|
| 1   | Collect source DB details              | SQL scripts (SGA, schemas, tablespaces)                    | Capture memory, users, roles, DB links         |
| 2   | Create AWS Infrastructure              | CloudFormation template, manual setup                     | RDS instance, EC2 client, S3 bucket            |
| 3   | Create IAM Role for RDS S3 integration  | AWS CLI or Console                                         | Trust policy + AmazonRDSFullAccess + S3Access  |
| 4   | Verify PDB access                      | ALTER SESSION SET CONTAINER; SHOW CON_NAME;                | Confirm connection to correct PDB             |
| 5   | Validate Directories                   | SELECT * FROM DBA_DIRECTORIES;                             | Ensure DATA_PUMP_DIR exists                    |
| 6   | Export schema with expdp               | expdp parfile=exp.par                                      | Use parallelism, FLASHBACK_TIME optional      |
| 7   | Upload dump to S3                      | AWS CLI: aws s3 cp *.dmp s3://bucket-name/                  | Ensure encryption and correct region          |
| 8   | Download dump on RDS                   | EXEC rdsadmin.rdsadmin_s3_tasks.download_from_s3            | Downloads to DATA_PUMP_DIR on RDS              |
| 9   | Prepare target RDS                     | SQL scripts for tablespaces, users, roles, synonyms        | Adapt to RDS limitations (no manual file naming)|
| 10  | Import schema with impdp               | impdp parfile=imp.par                                      | Monitor progress (DBA_DATAPUMP_JOBS, V$SESSION)|
| 11  | Reapply grants and privileges          | SQL grant scripts                                          | Ensure app users have correct access           |
| 12  | Compile invalid objects                | EXEC SYS.UTL_RECOMP.RECOMP_SERIAL('<SCHEMA>');             | Recompile invalid packages, views, etc.        |
| 13  | Validate migration                     | Compare object counts, synonyms, DB links                  | Cross-check source and target DBs              |
| 14  | Cleanup dump files on RDS              | EXEC rdsadmin.rdsadmin_util.delete_file('DATA_PUMP_DIR/');  | Free up storage                                |
| 15  | Final review and backup                | Manual checks and create final snapshot                    | Take RDS snapshot for rollback protection      |
+-----+----------------------------------------+-----------------------------------------------------------+------------------------------------------------+
```

---


## 1. Overview
This document outlines step-by-step procedures to migrate an Oracle PDB database from on-premises (or other environments) to Amazon RDS for Oracle.

---

## 2. Prerequisites
- Active AWS account
- Permissions:
  - IAM: Create roles, attach policies
  - RDS: Create/modify DB instances
  - S3: Upload/download files (multipart)
- Database access: Export permissions on source Oracle DB
- Setup:
  - Create S3 bucket
  - Create IAM role with RDS-S3 access
  - Create Target RDS for Oracle Multitenant (PDB) using CloudFormation  
    [AWS Example Template](https://aws.amazon.com/blogs/database/migrate-your-oracle-pluggable-database-to-amazon-rds-for-oracle/)
- Collect information:
  - VPC, subnet, security groups
  - Tags
  - Source DB sizing:
    - Database size
    - Tablespace size
    - Memory (SGA + PGA) from `DBA_HIST_SGASTAT`
    - List of schemas to migrate
    - Privileges, roles, profiles, DB links, synonyms
    - Listener port (e.g., 1529 / 1530)

---

## 3. Export Source Data
- Verify connection to source PDB:
  ```sql
  SELECT con_id, name, open_mode FROM v$pdbs;
  ALTER SESSION SET CONTAINER = <PDBName>;
  SHOW CON_NAME;
  ```
- Confirm `DATA_PUMP_DIR` directory exists:
  ```sql
  SELECT * FROM DBA_DIRECTORIES;
  ```
- Export schema data using **expdp**:
  - Sample `exp_db.par` content (adjust as per source DB)

- Export database metadata separately.

---

## 4. Prepare Target RDS
- Create RDS for Oracle instance.
- Create EC2 instance (client) to interact with RDS.
- Transfer scripts/dumps via S3.

---

## 5. Generate Supporting Scripts
- **Tablespaces creation** (bigfile tablespaces)
- **Temp tablespaces** creation
- **User quota assignment**
- **Synonyms**
- **Profiles, roles, users**
- **Privileges (grants)**

---

## 6. Migrate Files to Target
- Upload dumps to **Amazon S3**.
- Download from **S3 to RDS** using `rdsadmin.rdsadmin_s3_tasks.download_from_s3`.

---

## 7. Import to Target RDS
- Connect to RDS through EC2.
- Execute all generated SQL scripts:
  - Tablespaces, users, roles, profiles, quotas, synonyms
- Import using **impdp** with parallelism:
  - Import schemas
  - Monitor using:
    ```sql
    SELECT * FROM dba_datapump_jobs;
    SELECT * FROM v$session_longops;
    ```

---

## 8. Post Import Tasks
- Execute GRANT scripts again.
- Compile invalid objects:
  ```sql
  EXEC SYS.UTL_RECOMP.RECOMP_SERIAL('<SCHEMA>');
  ```
- Validate:
  - Number of objects per schema
  - Synonyms
  - Database links

---

## 9. Cleanup
- Remove dump files from RDS storage (to free space):
  ```sql
  EXEC rdsadmin.rdsadmin_util.delete_file('DATA_PUMP_DIR/filename.dmp');
  ```

---

# Notes
- **RDS Restrictions**:
  - No SYSDBA access
  - No OS-level access
  - OMF enforced (no manual file naming)
  - Limited `CREATE DIRECTORY`
- **Security**:
  - Validate IAM roles, policies
  - Audit export/import activities
- **Performance**:
  - Use sufficient EC2 instance size
  - Monitor RDS CPU and Storage through CloudWatch
  - Enable Enhanced Monitoring for RDS
- **Best Practices**:
  - Perform test migration before production
  - Use **Pre/Post migration checklist**
  - Perform backups before and after migration
  - Always validate schema counts, invalid objects

---
# Quick legend:
- **expdp** = Oracle Data Pump Export
- **impdp** = Oracle Data Pump Import
- **S3** = Amazon Simple Storage Service / Intermediate landing zone.
- **RDS** = Amazon Relational Database Service
- **PDB** = Oracle Pluggable Database
- **SFTP Server**: Entry point for dump file uploads from users.
- **EC2 Instance**: Session Manager enabled, runs Oracle datapump utilities.
- **EFS**: Acts as an RDS Oracle external directory.


Awesome — here's a simple **ASCII-style architecture diagram** showing the components and how they interact during your Oracle RDS migration with SFTP, S3, EC2, EFS, and RDS:

---

![image](https://github.com/user-attachments/assets/317a53be-e5c7-4ac5-bf56-d9b8cffe4401)


---

# Why this design is strong:
- No SSH keys required (Session Manager).
- Full control over files (SFTP uploads, copy to EFS).
- Enterprise-grade security (IAM roles, S3/EFS encryption).
- Deep visibility for troubleshooting (inside EC2 instance).
- Reusability: same SFTP+EFS infra can serve multiple DB migrations!


---
