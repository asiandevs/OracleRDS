Migrating Oracle PDB database to AWS RDS
•	Prerequisites
•	Export Source data from the source Oracle database
•	Create Target Oracle RDS 
o	Export Schemas data
o	Export Metadata
o	Generate scripts from metadata dump
o	Create tablespace scripts from the source database
o	Create Synonym script from the source database
o	Transfer Required files to S3
o	Transfer Required files from S3 to EC2
•	Import Database to Oracle RDS 
o	Execute scripts to target Oracle RDS
o	Download the dump files from Amazon S3 to the RDS instance
o	External File system
o	Execute Datapump import
o	Compile Invalid objects
o	Clean up Oracle dumps from the RDS
 

Prerequisites
•	An active AWS account
•	The required permissions to create roles in AWS Identity and Access Management (IAM) and for an Amazon S3 multipart upload
•	The required permissions to export data from the source database
•	Create an S3 bucket
•	Create the IAM role and assign policies
•	Create the target Amazon RDS for Oracle DB instance and associate the Amazon S3 integration role
Create an RDS for Oracle multitenant container database using the following AWS CloudFormation template.
Note: The CloudFormation template creates a Bring Your Own License (BYOL) RDS for Oracle instance.
 
https://aws.amazon.com/blogs/database/migrate-your-oracle-pluggable-database-to-amazon-rds-for-oracle/
•	Get the VPC and subnet details for the database
•	Get the necessary tags information
•	Collect the source database capacity
o	database size
o	Tablespace size
o	grant privileges
o	list of schemas - which need to be exported
o	get the database memory usage [ SGA + PGA ] - from DBA_HIST_SGASTAT
o	roles
o	profiles
o	list of database links
o	synonyms
o	listener port - 1529 / 1530
Below commands are specific to D1AUS database. Please make sure to modify the scripts according to the respective application and database requirements
Export Source data from the source Oracle database
 
Create Target Oracle RDS
Create an Oracle RDS
Export Schemas data
Connect to your source database
# Get the PDB list:
SQL> select con_id, name, open_mode from v$pdbs;

# Connect to source PDB
SQL> ALTER SESSION SET CONTAINER = <PDBName>;

#Verify
SQL> SHOW CON_NAME;

#Verify DATA_PUMP_DIR exists on both on-prem and RDS.
SELECT * FROM DBA_DIRECTORIES;
Connect to the source PDB database using TNS and perform an export of the schemas, that are in scope of the migration, using Oracle data pump
i. Contents of the par file
cat exp_d1aus.par
USERID='system/<password>@dt005alnxaur-vip.itsmyhome.net.au:1529/D1AUS'
DUMPFILE=exp_bkp_D1AUS_%U.dmp
LOGFILE=exp_bkp_D1AUS.log
DIRECTORY=dp_dir
SCHEMAS=SCHEMA1, SCHEMA2, SCHEMA3
FILESIZE=8G
CLUSTER=NO
PARALLEL=4
COMPRESSION=ALL
CONTENT=ALL
METRICS=YES
LOGTIME=ALL
FLASHBACK_TIME="to_timestamp(to_char(sysdate,'YYYY-MM-DD HH24:MI:SS'),'YYYY-MM-DD HH24:MI:SS')"
EXCLUDE=STATISTICS

To execute:
expdp parfile=exp_d1aus.par
Export Metadata
select distinct 'select dbms_metadata.get_ddl(''PROFILE'','''||profile||''')||'';'' from dual;'
from dba_profiles
where profile not in ('QUALYS_PROFILE');

i. Contents of the par file
cat exp_d1aus_metadata.par
USERID='system/<password>@Hostname:<Port>/<SID>'
DUMPFILE=exp_db01_md.dmp
LOGFILE=exp_db01_md.log
DIRECTORY=dp_dir
CONTENT=METADATA_ONLY
METRICS=Y
FULL=y

ii. To execute:
expdp parfile=exp_db01_metadata.par
Generate scripts from metadata dump
Create scripts for grant, profile, role and user from the exported metadata dump
i. Set database environment first
cd /mnt/Helia_DB_NFSmount/D1AUS
export ORACLE_PDB_SID=D1AUS
ii. Execute to generate SQL scripts
impdp \"/ as sysdba\" full=y DIRECTORY=dp_dir DUMPFILE=exp_db01_md.dmp SQLFILE=D1AUS_GRANTS.sql INCLUDE=GRANT
impdp \"/ as sysdba\" full=y DIRECTORY=dp_dir DUMPFILE=exp_db01_md.dmp SQLFILE=D1AUS_PROFILES.sql INCLUDE=PROFILE
impdp \"/ as sysdba\" full=y DIRECTORY=dp_dir DUMPFILE=exp_db01_md.dmp SQLFILE=D1AUS_ROLES.sql INCLUDE=ROLE
impdp \"/ as sysdba\" full=y DIRECTORY=dp_dir DUMPFILE=exp_db01_md.dmp SQLFILE=D1AUS_USERS.sql INCLUDE=USER

Create tablespace scripts from the source database
Connect to the source database and create database tablespace, temporary tablespaces and users tablespace quota
Amazon RDS only supports Oracle Managed Files (OMF) for data files, log files, and control files. When you create data files and log files, you can't specify the physical file names. Oracle RDS provided tablespaces are bigfile tablespaces by default. After connecting to the PDB, all are bigfile tablespaces excluding the TEMP tablespace.
select tablespace_name, bigfile
from   dba_tablespaces
order by 1;
-- Users Tablespaces 
set pause off
set lines 120
set pages 9999
set heading off
set feedback off
set showmode off
set echo off
set verify off
set serverout on size unlimited
set termout off
spool createtablesqtech.sql
select 'create tablespace '|| tablespace_name ||' datafile size 4G autoextend on next 1G maxsize unlimited;' from dba_data_files 
where tablespace_name in ('ts1', 'ts2', 'ts3','TEMP', 'TOOLS', 'USERS') ORDER BY tablespace_name;
spool off
exit
/

-- Temporary tablespaces
set pause off
set lines 120
set pages 9999
set heading off
set feedback off
set showmode off
set echo off
set verify off
set serverout on size unlimited
set termout off
spool createtemptables.sql
select 'create temporary tablespace '|| tablespace_name ||' tempfile size 4G autoextend on next 1G maxsize unlimited;' 
from dba_temp_files ORDER BY tablespace_name;
spool off
exit
/
-- tablespace Quota
spool create_ts_quota.sql
select distinct 'alter user '||OWNER||' quota unlimited on '||TABLESPACE_NAME||';' from dba_segments;
spool off
exit
/
Create Synonym script from the source database
set heading off
set feedback off
set pages 0
set linesize 300
spool mig_cre_synonyms.sql
select 'CREATE OR REPLACE '||decode(OWNER,'PUBLIC','PUBLIC ')
||'SYNONYM "'
||decode(OWNER,'PUBLIC','',OWNER)
||decode(OWNER,'PUBLIC','','".')
||decode(OWNER,'PUBLIC','','"')
||SYNONYM_NAME||'"'
||' FOR "'
||TABLE_OWNER||'"."'
||TABLE_NAME
||'";'
from dba_synonyms
where table_owner in (
select username from dba_users where username in (
'SCHEMA1',
'SCHEMA2',
'SCHEMA3')
)
and owner='PUBLIC'
;
spool off
Transfer Required files to S3
Transfer exported dump files to Amazon S3
There is a NFS attached with the current on-premise Linux database servers
/mnt/amity_DB_NFSmount
Now from there load the data to the S3 bucket
$ cd /mnt/amity_DB_NFSmount
$ sftp -i s3.pem poc-user@20.333.21.39
sftp> put D1AUS/exp_bkp_db01_* oracledump
Validate files are loaded to S3
 
Transfer Required files from S3 to EC2
Transfer all generated scripts ( tablespaces, users, profile, role, grant) to target EC2
[oracle@dt005alnxaur monowar_migscript]$ scp *.sql oracle@20.333.21.24:OracleDump/CHUMKI
oracle@20.333.21.24's password:
validate
 
Import Database to Oracle RDS
Execute scripts to target Oracle RDS
i) connect to the target oracle rds database through EC2
[oracle@ip-20.333-21-24 OracleDump]$ sqlplus 'pdbadmin@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=xxxxxx.ap-southeast-2.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=CHUMKI)))'

SQL*Plus: Release 21.0.0.0.0 - Production on Tue Jan 14 01:43:47 2025
Version 21.12.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.

Enter password:
Last Successful login time: Tue Jan 14 2025 01:19:50 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.25.0.0.0

SQL> show con_name

CON_NAME
------------------------------
CHUMKI
SQL>
ii) Create required tablespaces, profile, roles, users, grants
-- tablespaces
spool tablespaces.txt
@createtablesqtech.sql
@createtemptables.sql
spool off

-- profile
spool profiles.txt
@CHUMKI_PROFILES.sql
spool off

-- roles
spool roles.txt
@CHUMKI_ROLES.sql
spool off

-- users
spool usercreate.txt
@CHUMKI_USERS.sql
spool off

-- grant
spool grant.txt
@CHUMKI_GRANTS.sql
spool off

-- Tablespace Quota
spool tablespacequota.txt
@create_ts_quota.sql
spool off

-- Synonym
spool synonym.txt
@mig_cre_synonyms.sql
spool off

Grant script need to execute again after full import.
Download the dump files from Amazon S3 to the RDS instance
Make sure your RDS instance has enough storage space to accommodate the dump files. You can monitor the storage space from CloudWatch metrics, and prevent Amazon RDS from running out of space, by creating cloudwatch alarm on RDS storage .
SELECT rdsadmin.rdsadmin_s3_tasks.download_from_s3(
      p_bucket_name    =>  'poc-oracle-rds-testing-sdfr1286',
      p_s3_prefix      =>  'oracledump/exp_bkp_db01_01.dmp',
      p_directory_name =>  'DATA_PUMP_DIR')
AS TASK_ID FROM DUAL;

-- Note: Do for each dump file (wild card like "exp_bkp_db01_%u.dmp" did not work)
Verify the status of the file you uploaded to the RDS for Oracle instance with the following SQL query using the task id from the above output:
SELECT text from table(rdsadmin.rds_file_util.read_text_file('BDUMP' , 'dbtask-<task_id>.log'));
After the SQL query output shows the file downloaded successfully, you can list the data pump file in the RDS for Oracle database with the following query:
select * from table(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) order by filename;

Sample Output:
FILENAME                              TYPE         FILESIZE MTIME
------------------------------------- ---------- ---------- ---------
2B8D8B8F52B10C83E0630100007F99C1/     directory        4096 13-JAN-25
exp_bkp_db01_01.dmp                  file       5813813248 13-JAN-25
exp_bkp_db01_02.dmp                  file       5016264704 13-JAN-25
exp_bkp_db01_03.dmp                  file       5178421248 13-JAN-25
exp_bkp_db01_04.dmp                  file       5638828032 13-JAN-25
External File system
Create external file system for creating External database tables
Execute Datapump import
We are using Oracle dtapump to import data.
First we connect to the target Oracle RDS database from an EC2 and execute impdp
i) Content of the .par file
cat imp_db01_users.par
USERID='pdbadmin@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=dev-cdb.cbce36oemola.ap-southeast-2.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=CHUMKI)))'
DUMPFILE=exp_bkp_db01_%U.dmp
LOGFILE=imp_db01_users.log
DIRECTORY=DATA_PUMP_DIR
PARALLEL=4
#COMPRESSION=ALL
CONTENT=ALL
METRICS=YES
LOGTIME=ALL
SCHEMAS=SCHEMA1,
SCHEMA2,
SCHEMA3
ii) execute datapump import
export ORACLE_HOME=/u03/app/oracle/product/client19c
export PATH=$ORACLE_HOME/bin:$PATH
which impdp

impdp parfile=imp_db01_users.par
See the progress
-- List of datapmup jobs
select * from dba_datapump_jobs;

-- From API 
SELECT text FROM table(rdsadmin.rds_file_util.read_text_file('DATA_PUMP_DIR','imp_db01_users.log'));
-- where imp_db01_users.log is the log file name defined during import job

-- Queries taking longer time - use session longops to monitor progress

SELECT b.username, a.sid, b.opname, b.target, a.module,
            round(b.SOFAR*100/b.TOTALWORK,0) || '%' as "%DONE", b.TIME_REMAINING,
            to_char(b.start_time,'YYYY/MM/DD HH24:MI:SS') start_time,
            to_char(b.last_update_time,'YYYY/MM/DD HH24:MI:SS') last_update_time,
            to_char(b.sql_exec_start,'YYYY/MM/DD HH24:MI:SS') sql_exec_start
     FROM v$session_longops
     b, v$session a
     WHERE a.sid = b.sid      ORDER BY 9 desc;

-- Querying V$SESSION_LONGOPS and V$DATAPUMP_JOB views:-     
SELECT sl.sid, sl.serial#, sl.sofar, sl.totalwork, dp.owner_name, dp.state, dp.job_mode
     FROM v$session_longops sl, v$datapump_job dp
     WHERE sl.opname = dp.job_name
     AND sl.sofar != sl.totalwork;
When completed, execute grant script again, compile objects and validate
-- grant
spool grant01.txt
@CHUMKI_GRANTS.sql
spool off
-- compile invalid obejcts
SET LINES 180
SET ECHO OFF
SET HEA OFF
SET FEEDBACK OFF
 
SPOOL /home/oracle/OracleDump/fixinvalid.sql
select distinct('exec SYS.UTL_RECOMP.RECOMP_SERIAL('''||owner||''');') from dba_objects WHERE status = 'INVALID'
/
spool off
 
SET FEEDBACK ON
SET ECHO ON
@/home/oracle/OracleDump/fixinvalid.sql
-- check invalid objects
-- set lines 200 
set pages 999
col c1 heading 'owner' format a15
col c2 heading 'name' format a40
col c3 heading 'type' format a10
ttitle 'Invalid|Objects'

select owner c1, object_type c3, object_name c2 from dba_objects where status != 'VALID' order by owner, object_type;

select owner, count(*),object_type from  dba_objects where status<>'VALID' group by owner, object_type order by 1;

Grant additional Privileges based on the requirements
		

This is for CHUMKI database. Required privileges will be granted according to the application.
grant dba to OWNER1;

grant select on dba_users to secowner;

Grant select on sys.dba_role_privs to owner1  with grant option;
Grant select on sys.dba_roles to secowner with grant option; 
grant execute on sys.dbms_pipe to owner2; 
grant execute on sys.dbms_alert to owner2;
grant select on sys.v_$session to owner1;
grant execute on SYS.UTL_MAIL to owner2;
grant execute on sys.dbms_lock to owner2;


grant execute on pkgowner.p_decision to dataowner with grant option;
grant execute on pkgowner.p_documents to dataowner with grant option;
grant execute on pkgowner.p_entry to dataowner with grant option;
grant execute on pkgowner.p_cst to dataowner with grant option;
grant execute on pkgowner.p_lmi to dataowner with grant option;
-- Password Verify function 

begin
    rdsadmin.rdsadmin_password_verify.create_verify_function(
        p_verify_function_name => 'ora19c_stig_verify_function', 
        p_min_length           => 16, 
        p_min_uppercase        => 1, 
		p_min_lowercase        => 1,
        p_min_digits           => 1, 
        p_min_special          => 1);
end;
/

ALTER PROFILE DEFAULT LIMIT PASSWORD_VERIFY_FUNCTION ora19c_stig_verify_function;

BEGIN
  rdsadmin.rdsadmin_util.grant_sys_object(
    p_obj_name  => 'AUD$',         -- Note: just AUD$, not SYS.AUD$
    p_grantee   => 'owner2',     -- Replace with actual username
    p_privilege => 'SELECT',
    p_grant_option => FALSE        -- Add this parameter
  );
END;
/


Compile Invalid objects
select owner, object_type, object_name from dba_invalid_objects;

SPOOL /tmp/compileinvalidobj.sql
select distinct('exec SYS.UTL_RECOMP.RECOMP_SERIAL('''||owner||''');') from dba_objects WHERE status = 'INVALID'
/
SPOOL OFF
 
@/tmp/compileinvalidobj.sql
#### Few EXAMPLE Command ####
# Create tablespace
CREATE TABLESPACE TS01 DATAFILE SIZE 1G AUTOEXTEND ON MAXSIZE Unlimited;
#Setting the default tablespace
EXEC rdsadmin.rdsadmin_util.alter_default_tablespace(tablespace_name => 'ts01');
#Setting the default temporary tablespace
EXEC rdsadmin.rdsadmin_util.alter_default_temp_tablespace(tablespace_name => 'temp01');
#Changing the global name of a database
EXEC rdsadmin.rdsadmin_util.rename_global_name(p_new_global_name => 'new_global_name');
Clean up Oracle dumps from the RDS
After successfully migrated database to RDS, remove unnecessary dumps from the RDS
-- Find the loaded dump/s from RDS
SQL> SELECT * FROM TABLE(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) ORDER by mtime;

-- delete unwanted dump file/s from rds
SQL> EXEC UTL_FILE.FREMOVE('DATA_PUMP_DIR','exp_db01_04.dmp');
