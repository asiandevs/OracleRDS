# AWS RDS Oracle: Issues, Features, and Solutions

## Connection Issues
- [RDS Oracle Connection Errors](https://repost.aws/knowledge-center/rds-oracle-connection-errors)

## Modify SID Name
- [Change RDS Oracle SID Instance](https://repost.aws/knowledge-center/rds-oracle-change-sid-instance)

## Huge Pages
- [Huge Pages in Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Concepts.HugePages.html)

## Database Features
### Overview of RDS for Oracle CDBs
- Database init parameters and options
- Unsupported features at the PDB level (supported at the CDB level):
  - Option groups (applies to all PDBs in the CDB instance)
  - Parameter groups (all parameters are derived from the parameter group of the CDB instance)
- [Reference](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Concepts.CDBs.html#Oracle.Concepts.single-tenant-limitations)

## In-Transit Encryption
- Enable SSL encryption by adding the Oracle SSL option to the option group.
- RDS uses a second port for SSL connections, allowing both clear text and SSL communication.
- [Oracle Secure Sockets Layer - Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Concepts.SSL.html)
- [Connecting to RDS Oracle using SSL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html)
- [Jive Documentation: SSL/TLS Connection](https://www.jivesoftware.com/)

## Oracle RAC on AWS
- AWS RDS does not support Oracle RAC features.
- [Migration from Oracle RAC to AWS Alternatives](https://aws.amazon.com/blogs/database/migrate-from-oracle-rac-to-aws-alternatives-on-aws/)

## Read Replica
### Observations
- 3-7 minutes lag in Latest Restore Time (LRT)
- Achieving zero RPO requires Multi-AZ or Physical Standby Database

### Available Solutions
1. **Oracle Read Replicas (Enterprise Edition Required)**
   - **Read-only mode** (requires Active Data Guard license)
   - **Mounted mode** (no Active Data Guard license required, cost-effective DR solution)
2. **AWS Database Migration Service (DMS) or Oracle GoldenGate**
   - [AWS DMS](https://aws.amazon.com/dms/)
   - [Oracle GoldenGate](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.OracleGoldenGate.html)

### Limitations
- RDS for Oracle supports Data Guard read replicas for Oracle 19c and 21c CDBs in single-tenant configuration only.
- [Oracle Read Replicas Overview](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-read-replicas.overview.html#oracle-read-replicas.overview.modes)

## Different Create Dates in RDS
- Variations due to DBCA templates.
- Different Oracle versions can result in different "created" timestamps.

## OMS Issue
- Agent could not communicate back to OMS.
- Verify OMS host resolution:
  ```sql
  SELECT UTL_INADDR.get_host_address('OMS_Host_Name') FROM dual;
  ```
- [Troubleshooting OEM Agent](https://repost.aws/knowledge-center/rds-oracle-oem-agent-errors)
- [RDS OEM Agent Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Options.OEMAgent.html)

## Incompatible Parameter Status
- Causes:
  - DB instance scaled to an incompatible instance type.
  - Engine upgraded, causing incompatibilities.
- Solutions:
  - Identify and reset incompatible parameters.
  - Use default parameter groups temporarily.
- [Fixing Incompatible Parameters](https://repost.aws/knowledge-center/rds-incompatible-parameters)

## Stopping a Running Backup
- Not possible to stop an ongoing backup job.
- To disable future automated backups:
  - [Getting an Existing ARN](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Tagging.ARN.html#USER_Tagging.ARN.Getting)

## Restore Database
- Restore a DB instance to a specific point in time without modifying the source instance.
- [Point-in-Time Restore](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)

## Flashback Technology
- RDS for Oracle supports:
  - Flashback Table
  - Flashback Query
  - Flashback Transaction
- Does **not** support Flashback Database.
- [Flashback Database Alternatives](https://aws.amazon.com/blogs/database/alternatives-to-the-oracle-flashback-database-feature-in-amazon-rds-for-oracle/)

## AWS DMS and Oracle
- DMS does not support Oracle LogMiner in a PDB environment.
- Solution:
  - Use Binary Reader:
    ```
    Above error is observed due to a limitation with Oracle database as source for DMS. 

As per the limitation “Oracle LogMiner doesn't support connections to a pluggable database (PDB). To connect to a PDB, access the redo logs using Binary Reader.”
[+]https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.Oracle.html#CHAP_Source.Oracle.Limitations 

To use Binary Reader to access the redo logs, add the following extra connection attributes on the endpoint   
useLogMinerReader=N;useBfile=Y;

The steps for adding extra connection attributes are as follows:

1. Open the AWS DMS console, and then select the AWS Region that the endpoint is in.
2. From the navigation pane, choose Endpoints, and then select the endpoint that you want to modify.
3. Choose Actions, and then choose Modify.
4. Expand the Endpoint settings section, and then select the checkbox for “Use endpoint connection attributes”.
5. Under “Extra connection attributes” heading add “useLogMinerReader=N;useBfile=Y;”
6. Choose Save.

[+]How can I add or modify endpoint settings for AWS DMS endpoints?
https://repost.aws/knowledge-center/dms-extra-connection-attributes 

    ```
  - [DMS Extra Connection Attributes](https://repost.aws/knowledge-center/dms-extra-connection-attributes)

## External Directory Setup on RDS

- atively we can not integrate FSx and RDS oracle as there is no option group available.
- Replication steps:-
  
```
1) Created an RDS oracle instance of same version- ' 19.0.0.0.ru-2025-01.rur-2025-01.r1'
2) Created an FSx file system.
3) Created an EC2 (amazon linux) instance to communicate between RDS and FSx.
4) Created a new directory and mounted the FSX filesystem to that directory.

mkdir -p /Fsxtst

ec2-user@ip-172-xx-1-xxx ~]$ df -h
Filesystem                                               Size  Used Avail Use% Mounted on
devtmpfs                                                 4.0M     0  4.0M   0% /dev
tmpfs                                                    475M     0  475M   0% /dev/shm
tmpfs                                                    190M  448K  190M   1% /run
/dev/xvda1                                               8.0G  1.9G  6.2G  23% /
tmpfs                                                    475M     0  475M   0% /tmp
/dev/xvda128                                              10M  1.3M  8.7M  13% /boot/efi
fs-xxxxxxxxxxxx.amazonaws.com:/fsx/   64G   54M   64G   1% /Fsxtst
tmpfs                                                     95M     0   95M   0% /run/user/1000

5) Installed Oracle client in the EC2 instance and connected to the RDS instance .

ec2-user@ip-172-31-1-182 ~]$ sqlplus admin@database-2.xxxxxxxxxx.rds.amazonaws.com:1521/ABCDE

SQL*Plus: Release 21.0.0.0.0 - Production on Tue Mar 18 07:54:10 2025
Version 21.9.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.

Enter password: 
Last Successful login time: Tue Mar 18 2025 07:53:39 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.26.0.0.0

6) When tried to create an Oracle directory using the OS level FSx filesystem directory, it throws an error as mentioned below.

SQL> create directory test as '/Fsxtst';
create directory test as '/Fsxtst'
*
ERROR at line 1:
ORA-04088: error during execution of trigger 'RDSADMIN.RDS_DDL_TRIGGER2'
ORA-00604: error occurred at recursive SQL level 1
ORA-20900: Invalid path used for directory: /Fsxtst
ORA-06512: at "RDSADMIN.RDSADMIN_TRIGGER_UTIL", line 714
ORA-06512: at line 1
ORA-06512: at line 12
```
-- Solution
```

With the output shared, I could see that the path '/oracle/gatewaymm' dont have writer access 'w'. So could you please provide write access to the absolute path of the directory and try again?

Current setup:-
------------------
[oracle@ip-10-16-144-167 oracle]$ ls -ld /oracle
drwxr-xr-x 3 oracle oracle 6144 Mar 20 03:16 /oracle
[oracle@ip-10-16-144-167 oracle]$ ls -ld /oracle/gatewaymm
drwxr-xr-x 3 oracle oracle 6144 Mar 20 22:03 /oracle/gatewaymm

Action need to be performed:-
-------------------------------------
chmod 777 /oracle
chmod 777 /oracle/gatewaymm
cd /oracle/gatewaymm
chmod 777 *

After this you should be able to see the 'w' in the permissions
"drwxrwxrwx" instead of "drwxr-xr-x"
```
===================
Replication step:-
===================

1) Mounting the EFS and Creating necessary directories on EC2.

sudo mkdir -p /efsdir
sudo mount -t efs -o tls fs-0ed404ff2374e6534:/ /efsdir

[ec2-user@ip-172-31-1-11 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           475M     0  475M   0% /dev/shm
tmpfs           190M  516K  190M   1% /run
/dev/xvda1      8.0G  1.6G  6.4G  21% /
tmpfs           475M     0  475M   0% /tmp
/dev/xvda128     10M  1.3M  8.7M  13% /boot/efi
tmpfs            95M     0   95M   0% /run/user/1000
127.0.0.1:/     8.0E     0  8.0E   0% /efsdir

sudo mkdir /efsdir/datapump --> Creating directory for storing files to create an external table.

2) Checking OS level permissions:

[ec2-user@ip-172-31-1-11 datapump]$ ls -ld /efsdir/datapump
drwxrwxrwx. 3 root root 6144 Mar 20 15:00 /efsdir/datapump


3) Creating a dummy txt file:

echo -e 10, Boston\\n20, Denver\\n30, Toronto > /efsdir/datapump/basketball_teams.txt

[ec2-user@ip-172-31-1-11 datapump]$ pwd
/efsdir/datapump
[ec2-user@ip-172-31-1-11 datapump]$ ls -ltrh
total 28K
-rwxrwxrwx. 1 ec2-user ec2-user   34 Mar 20 14:44 basketball_teams.txt

4) Connecting to the RDS Oracle using sqlplus to create oracle directory:

BEGIN
    rdsadmin.rdsadmin_util.create_directory_efs(
    p_directory_name => 'DATAUPLOAD1',
    p_path_on_efs => '/rdsefs-fs-0ed404ff2374e6534/datapump');
END;
/

set pages 999 lines 999;
col DIRECTORY_PATH format a50;
col DIRECTORY_NAME format a30;
col owner format a20;
select * from dba_directories;SQL> SQL> SQL> SQL> 

OWNER		     DIRECTORY_NAME		    DIRECTORY_PATH				       ORIGIN_CON_ID
-------------------- ------------------------------ -------------------------------------------------- -------------
SYS		     OPATCH_INST_DIR		    /rdsdbbin/oracle/OPatch					   0
SYS		     RDS$TEMP			    /rdsdbdata/tmp						   0
SYS		     DATAUPLOAD1		    /rdsefs-fs-0ed404ff2374e6534/datapump			   0
SYS		     JAVA$JOX$CUJS$DIRECTORY$	    /rdsdbbin/oracle/javavm/admin/				   0
SYS		     DATA_PUMP_DIR		    /rdsdbdata/datapump 					   0
SYS		     ADUMP			    /rdsdbdata/admin/ORCL/adump 				   0
SYS		     RDS$DB_TASKS		    /rdsdbdata/dbtasks						   0
SYS		     OPATCH_SCRIPT_DIR		    /rdsdbbin/oracle/QOpatch					   0
SYS		     OPATCH_LOG_DIR		    /rdsdbbin/oracle/rdbms/log					   0
SYS		     BDUMP			    /rdsdbdata/log/trace					   0


col FILENAME format a20;
SELECT * FROM TABLE(rdsadmin.rds_file_util.listdir(p_directory => 'DATAUPLOAD1'));

FILENAME	     TYPE	  FILESIZE MTIME
-------------------- ---------- ---------- ---------
datapump/	     directory	      6144 20-MAR-25
basketball_teams.txt file		34 20-MAR-25

Note: Yes this step is working for me while replciating.


5) Creating a external table with the dummy text file:

CREATE TABLE basketball_teams (
  id         NUMBER,
  team_name  VARCHAR2(50)
 )
ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY DATAUPLOAD1
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    DNFS_DISABLE
    FIELDS TERMINATED BY ','
    MISSING FIELD VALUES ARE NULL
    (id,team_name)
  )
  LOCATION ('basketball_teams.txt')
)
PARALLEL
REJECT LIMIT UNLIMITED;  

Table created. 

SQL> select * from basketball_teams;

	ID TEAM_NAME
---------- -----------------
	10  Boston
	20  Denver
	30  Toronto
 
```
Created EC2 instance 
Created RDS Oracle instance 
Created EFS filesystem 
Configured security group settings between EC2 and EFS 
Created Option group for EFS and Integrated with RDS 
Configured security group setting between RDS and EFS
```
==========================
It is working as expected.
==========================
```
```
Title: "Maximizing Audit Log Retention in Amazon RDS for Oracle: A Comprehensive Guide" 

By default, Amazon RDS maintains audit files for just seven days – a limitation that can be challenging for organizations requiring longer retention periods for compliance or security purposes. In this blog post, let's explore how to effectively manage and extend your RDS Oracle audit log retention using Amazon CloudWatch Logs. 

Understanding the Basics:

The default 7-day retention period for audit files cannot be modified directly on the RDS instance

Audit files and trace files share the same retention configuration

After seven days, Amazon RDS automatically deletes older audit files

The CloudWatch Solution 

To retain audit logs beyond the 7-day limit, Amazon CloudWatch Logs offers an excellent solution. Here's why it's beneficial:

Highly durable storage

Advanced analysis capabilities

Custom alarm creation

Metric visualization

Flexible retention periods

Implementation Process:

Configure your RDS for Oracle instance to publish log data to CloudWatch Logs

Each Oracle database log is published as a separate stream in the format:
/aws/rds/instance/my_instance/audit

Configuring Audit Trails 

Set the audit_trail parameter to one of these values:

none

os

db [, extended]

xml [, extended]

Important Considerations 

CloudWatch Logs Retention:

By default, logs are stored indefinitely

Retention periods can be customized as needed

Modification can be done through CloudWatch Logs console

Archivelog Retention:

Default value is 0 (immediate purging after creation)

Doesn't affect Point-in-Time Recovery capabilities

RDS maintains archived redo logs externally based on backup retention period

Best Practices 

Regularly review and adjust retention periods based on compliance requirements

Monitor CloudWatch Logs storage costs

Implement appropriate log analysis strategies

Maintain documentation of retention configurations

This solution provides organizations with the flexibility to maintain audit logs for extended periods while leveraging CloudWatch's powerful analysis capabilities. 

For detailed implementation steps and additional information, refer to AWS's official documentation on Oracle log access and CloudWatch Logs management.

Remember: Proper audit log management is crucial for maintaining security compliance and ensuring operational transparency in your database environment.

 You can refer the following documentation to enable exporting the logs to CloudWatch,
 	[+] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.Oracle.html#USER_LogAccess.Oracle.PublishtoCloudWatchLogs 


Please note that by default CloudWatch logs are stored indefinitely, however you can modify the retention period of this log group by referring the following documentation,
 	[+] Change log data retention in CloudWatch Logs - https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html#SttingLogRetention 


Archivelog log retention specifies the duration in hours before archive/redo log files are automatically deleted. As for the archive log retention hours, I would like to inform you that the default value would be 0 and this indicates that the archive logs are purged after their creation, however this will not have any impact on your Point In time restores as when the archived log retention period expires, RDS for Oracle removes the archived redo logs from your DB instance. To support restoring your DB instance to a point in time, Amazon RDS retains the archived redo logs outside of your DB instance based on the backup retention period.  


You can read more on archive log retention from the following documentation, 
	[+] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RetainRedoLogs.html 

 
```
- Conclusion: as RDS is an managed service, if anything need to be done/required OS level activity those will be provided as part of option group. All those integration through option group provided necessary OS level permission by providing a predefined oracle stored procedure and functions. Thus whenever you are trying to create directory using EFS integration, the predefined procedure/function have access to underlying OS filesystem.

- So it is always recommended to use either S3 or EFS for additional export/import/ETL process.

---
```
set pagesize 3000
set lines 2000
set head off
set feedback off
spool user_backups.sql
select 'alter user '||username||' profile '||profile||';' from dba_users;
select 'alter user '|| name ||' identified by values '''||decode(spare4,null,password,spare4)||''';' sql 
from sys.user$ 
where name in ('SVC_WS_AUDIT', 'UNDERWRITER');
```
