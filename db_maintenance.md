# Amazon RDS: Restricted Commands and Administrative Tasks

In Amazon RDS, you are not allowed to execute the following commands:

- `ALTER DATABASE`
- `ALTER SYSTEM`
- `GRANT ANY ROLE/PRIVILEGE`
- `DROP/CREATE ANY DIRECTORY`

This is because AWS manages the RDS instance and restricts access to these commands. Instead, AWS provides packages under the `rdsadmin` schema to help you perform common DBA tasks without needing to execute the above commands.
Run the following SQL command in your Oracle RDS instance using a SQL client like SQL*Plus or SQL Developer or EC2.

### Database Name

```sql
exec rdsadmin.rdsadmin_util.rename_global_name(p_new_global_name => 'NEWDB');
```
---
### Sessions Management

#### Check Active Sessions
```sql
SELECT substr(s.INST_ID || '|' || s.USERNAME || '| ' || s.sid || ',' || s.serial# || ' |' || substr(s.MACHINE,1,22) || '|' || substr(s.MODULE,1,18),1,69) AS "INS|USER|SID,SER|MACHIN|MODUL",
       substr(s.status || '|' || round(w.WAIT_TIME_MICRO / 1000000) || '|' || LAST_CALL_ET || '|' || to_char(LOGON_TIME,'ddMon HH24:MI'),1,40) AS "ST|WAITD|ACT_SINC|LOGIN",
       substr(w.event,1,24) AS "EVENT",
       s.SQL_ID || '|' || round(w.TIME_REMAINING_MICRO / 1000000) AS "CURRENT SQL|REMIN_SEC",
       s.FINAL_BLOCKING_INSTANCE || '|' || s.FINAL_BLOCKING_SESSION AS "I|BLK_BY"
FROM   gv$session s, gv$session_wait w
WHERE  s.USERNAME IS NOT NULL 
AND    s.sid = w.sid 
AND    s.STATUS = 'ACTIVE'
ORDER BY "I|BLK_BY" DESC, w.event, "INS|USER|SID,SER|MACHIN|MODUL", "ST|WAITD|ACT_SINC|LOGIN" DESC, "CURRENT SQL|REMIN_SEC";
```
#### Disconnect a Session
```sql
EXEC rdsadmin.rdsadmin_util.disconnect(<SID>, <SERIAL#>, 'IMMEDIATE');
```
#### Cancel Statement
```sql
EXEC rdsadmin.rdsadmin_util.cancel(<SID>, <SERIAL#>, '<SQLID>');
```
#### Enable Restricted Sessions
```sql
EXEC rdsadmin.rdsadmin_util.restricted_session(p_enable => true);
```
```sql
SELECT logins FROM v$instance;
```
#### Disable Restricted Sessions
```sql
EXEC rdsadmin.rdsadmin_util.restricted_session(p_enable => false);
```
```sql
SELECT logins FROM v$instance;
```
#### Kill a Sessions
```sql
begin
rdsadmin.rdsadmin_util.kill( sid => sid, serial => serial_number);
end;
/
```
#### Kill session - immediately (kill the process). [overriding the value IMMEDIATE in 3rd parameter with value PROCESS]:
```sql
begin
rdsadmin.rdsadmin_util.kill( sid => sid, serial => serial_number, method => PROCESS);
end;
/
```
---

## Users Management

### grant select on sys objects to users:
```sql
begin
rdsadmin.rdsadmin_util.grant_sys_object( p_obj_name => ‘V_$SESSION', p_grantee => 'USERNAME', p_privilege => 'SELECT');
end; /
```
```
BEGIN
  rdsadmin.rdsadmin_util.grant_sys_object(
    p_obj_name  => 'DBMS_STANDARD',         -- Note: just AUD$, not SYS.AUD$
    p_grantee   => 'DB_TEST',     -- Replace with actual username
    p_privilege => 'EXECUTE',
    p_grant_option => FALSE        -- Add this parameter
  );
END;
/
```

### Grant Permission on SYS Tables/Views with the grant option
```sql
EXEC rdsadmin.rdsadmin_util.grant_sys_object(p_obj_name  => 'V_$SESSION', p_grantee => 'USER1', p_privilege => 'SELECT', p_grant_option => true);
```

### Revoke Permission from a User on SYS Object
```sql
EXEC rdsadmin.rdsadmin_util.revoke_sys_object(p_obj_name  => 'V_$SESSION', p_revokee  => 'USER1', p_privilege => 'SELECT');
```

### Grant SELECT on Dictionary
```sql
GRANT SELECT_CATALOG_ROLE TO user1;
```

### Grant EXECUTE on Dictionary
```sql
GRANT EXECUTE_CATALOG_ROLE TO user1;
```
### Revoking SELECT or EXECUTE Privileges on SYS Objects:
```sql
begin
rdsadmin.rdsadmin_util.revoke_sys_object(
p_obj_name => ‘V_$SESSION‘,
p_revokee => ‘USER1‘,
p_privilege => ‘SELECT‘);
end;
/
```
### create a custom function to verify passwords by using the Amazon RDS procedure and assign:

```sql
begin
 rdsadmin.rdsadmin_password_verify.create_verify_function(
 p_verify_function_name => 'amity_stig_verify_function', 
 p_min_length => 12, 
 p_min_uppercase => 1, 
 p_min_digits => 1, 
 p_min_special => 1,
 p_disallow_at_sign => true);
end;
/
alter profile Default LIMIT PASSWORD_VERIFY_FUNCTION amity_stig_verify_function

```
---
### Tablespaces
Amazon RDS only supports Oracle Managed Files (OMF) for data files, log files, and control files. When you create data files and log files, you can’t specify the physical file names. By default, the tablespace created is a bigfile tablespace.

#### To create a smallfile tablespace, you need to mention the “smallfile” keyword after create in your syntax.
```
create smallfile tablespace janglast datafile size 100M autoextend on maxsize 1G;
```
```
create smallfile temporary tablespace tempsmall01;
```

Better not to use smallfile tablespaces as no option resize smallfile tablespaces with Amazon RDS for Oracle but to add a datafile to a smallfile tablespace.
```
alter tablespace janglast resize 4g;
```
```
alter tablespace janglast add datafile size 4g;
```
drop tablespace janglast including contents and datafiles;

#### Set DEFAULT TABLESPACE:
```sql
EXEC rdsadmin.rdsadmin_util.alter_default_tablespace(tablespace_name => 'example');
```

#### Resize TEMPORARY tablespace:
```sql
EXEC rdsadmin.rdsadmin_util.resize_temp_tablespace('TEMP','4G');
```
#### Tablespace and Storage Management:
```sql
SELECT tablespace_name, 
       round((tablespace_size8192)/(10241024)) Total_MB, 
       round((used_space8192)/(10241024)) Used_MB, 
       round(used_percent,2) "%Used" 
FROM dba_tablespace_usage_metrics;
```
#### Object Management:
```sql
SELECT SEGMENT_NAME, 
       TABLESPACE_NAME, 
       SEGMENT_TYPE, 
       ROUND(SUM(BYTES/1024/1024)) OBJECT_SIZE_MB 
FROM SYS.DBA_SEGMENTS 
WHERE OWNER = upper('SPRINT_STAGE1') 
GROUP BY SEGMENT_NAME, TABLESPACE_NAME, SEGMENT_TYPE;
```
---

### Directory/S3 Management

#### Create a Directory `DATA_PUMP_DIR` Object in Oracle RDS

```sql
BEGIN
  rdsadmin.rdsadmin_util.create_directory(
    p_directory_name  => 'DATA_PUMP_DIR',
    p_directory_path  => 'DATA_PUMP_DIR'
  );
END;
/
```
#### Upload a file to S3 bucket:
```sql
SELECT rdsadmin.rdsadmin_s3_tasks.upload_to_s3( 
    p_bucket_name => '<bucket_name>', 
    p_prefix => '<file_name>', 
    prefix => '', 
    p_directory_name => 'DATA_PUMP_DIR') 
AS TASK_ID 
FROM DUAL;
```

#### Download all files from an S3 bucket:
```sql
SELECT rdsadmin.rdsadmin_s3_tasks.download_from_s3( 
    p_bucket_name => 'my-bucket', 
    p_directory_name => 'DATA_PUMP_DIR') 
AS TASK_ID 
FROM DUAL;
```

#### Download specific files from an S3 bucket:
```sql
SELECT rdsadmin.rdsadmin_s3_tasks.download_from_s3( 
    p_bucket_name => 'my-bucket', 
    p_s3_prefix => 'export_files/', 
    p_directory_name => 'DATA_PUMP_DIR') 
AS TASK_ID 
FROM DUAL;
```

#### Copy the Dump File from S3 to RDS

Use the `rdsadmin.rdsadmin_s3_tasks.download_from_s3` procedure to copy the dump file from S3 to the `DATA_PUMP_DIR` directory on the RDS instance:

```sql
BEGIN
  rdsadmin.rdsadmin_s3_tasks.download_from_s3(
    p_bucket_name     => 'your-bucket-name',
    p_s3_prefix       => 'folder-name/',
    p_directory_name  => 'DATA_PUMP_DIR'
  );
END;
/
```

#### Show All Files Under `DATA_PUMP_DIR`
```sql
SELECT * FROM TABLE(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) ORDER BY mtime;
```

#### View Logs Using `rds_file_util.read_text_file`:

To view the log file (e.g.,import_multi_file.log), use the following procedure to read it in chunks:

```sql
DECLARE
  v_text VARCHAR2(4000);
BEGIN
  v_text := rdsadmin.rds_file_util.read_text_file(
    p_directory_name => 'DATA_PUMP_DIR',
    p_filename       => 'import_multi_file.log'
  );
  DBMS_OUTPUT.PUT_LINE(v_text);
END;
/
```

#### Read a Log File Under `BDUMP`
```sql
SELECT text FROM TABLE(rdsadmin.rds_file_util.read_text_file('BDUMP','dbtask-<taskid>.log'));
```

#### Delete a Directory
> **Note:** Deleting a directory will not delete the underlying files. You need to delete them manually before removing the directory.

1. List all files under the directory:
```sql
SELECT * FROM TABLE(RDSADMIN.RDS_FILE_UTIL.LISTDIR('BKP_DIR')) ORDER BY mtime;
```

2. Delete files one by one:
```sql
EXEC utl_file.fremove('DATA_PUMP_DIR','export_RA%U.dmp');
```

3. Delete the directory:
```sql
EXEC rdsadmin.rdsadmin_util.drop_directory(p_directory_name => 'BKP_DIR');
```

4. Rename a file:
```sql
EXEC UTL_FILE.FRENAME('DATA_PUMP_DIR', '<Original_filename>', 'DATA_PUMP_DIR', '<New_filename>', TRUE);
```
---
### Enable/Disable Force Logging:
```sql
EXEC rdsadmin.rdsadmin_util.force_logging(p_enable => true );
```
```
EXEC rdsadmin.rdsadmin_util.force_logging(p_enable => false);
```

### Flush Shared Pool:
```sql
EXEC rdsadmin.rdsadmin_util.flush_shared_pool;
```
### Flush Buffer Cache:
```sql
EXEC rdsadmin.rdsadmin_util.flush_buffer_cache;
```
### Purge Recyclebin:
```sql
exec rdsadmin.rdsadmin_util.purge_dba_recyclebin;
```
### Logfile group management
Adding Online Redo Logs An Amazon RDS DB instance running Oracle starts with four online redo logs, 128 MB each. here 500M and '3' example only
Drop each inactive log using the group number. For that do log switch and check point and then delete and recreate with right sizing.

To add additional redo logs, use the Amazon RDS procedure rdsadmin.rdsadmin_util.add_logfile.
```
exec rdsadmin.rdsadmin_util.add_logfile(p_size => '500M');
```
```
select GROUP#,bytes/1024/1024,STATUS from v$log;
```
```
exec rdsadmin.rdsadmin_util.drop_logfile(grp => 3);
```

### Force a Checkpoint and Switch log:
```sql
EXEC rdsadmin.rdsadmin_util.checkpoint;
```
```
EXEC rdsadmin.rdsadmin_util.switch_logfile;
```
### For a different user than the rdsadmin, grant system privileges
```
GRANT EXECUTE ON rdsadmin.rdsadmin_util TO DB_DEVELOPER;
```
### View REDOLOG switches per hour:
```sql
SELECT to_char(first_time,'YYYY-MON-DD') day, 
       to_char(sum(decode(to_char(first_time,'HH24'),'00',1,0)),'9999') "00" 
FROM v$log_history 
WHERE first_time > sysdate-1 
GROUP BY to_char(first_time,'YYYY-MON-DD') 
ORDER BY 1 ASC;
```
### Add/Drop REDO LOG Group:
```sql
EXEC rdsadmin.rdsadmin_util.add_logfile(p_size => '1G');
```
```
EXEC rdsadmin.rdsadmin_util.drop_logfile(1);
```
### Checking and Updating Archive Log Retention:
```sql
set serveroutput on;
EXEC rdsadmin.rdsadmin_util.show_configuration;
```
```
EXEC rdsadmin.rdsadmin_util.set_configuration(name => 'archivelog retention hours', value => '24');
```
#### To keep logs for a longer period of time for a replication process:
```sql
exec rdsadmin.rdsadmin_util.set_configuration('archivelog retention hours',48);
```
#### To see expired logs and their delete status:
```sql
select trunc(completion_time,'DD'), deleted, count(*)
from v$archived_log
group by trunc(completion_time,'DD'), deleted
order by 1 desc;
```

### Add Supplemental Log:
```sql
EXEC rdsadmin.rdsadmin_util.alter_supplemental_logging('ADD','ALL');
```

### Change Database Timezone (Here setting up as Sydney):
```sql
alter session set nls_date_format='DD-MON-YY HH24:MI:SS';
select sysdate from dual;

select systimestamp from dual;
SELECT dbtimezone FROM DUAL;

EXEC rdsadmin.rdsadmin_util.alter_db_time_zone(p_new_tz => 'Australia/Sydney');
```
Restart database to reflect the change.

### Gather Statistics:
```sql
BEGIN 
  DBMS_STATS.GATHER_DATABASE_STATS(
    ESTIMATE_PERCENT => DBMS_STATS.AUTO_SAMPLE_SIZE, 
    CASCADE => TRUE, 
    degree => DBMS_STATS.AUTO_DEGREE, 
    GATHER_SYS => TRUE, 
    OPTIONS => 'GATHER STALE'
  ); 
END;
/
```

### Re-Compile invalid objects:
```sql
EXEC SYS.UTL_RECOMP.recomp_parallel(4);
```
```
EXECUTE SYS.UTL_RECOMP.RECOMP_SERIAL();
```

### Disable the CORRUPTION SKIPPING on the corrupted object:

```sql
BEGIN
  rdsadmin.rdsadmin_dbms_repair.skip_corrupt_blocks (
    schema_name => '&corrupted_Object_Owner',
    object_name => '&corrupted_object_name',
    object_type => rdsadmin.rdsadmin_dbms_repair.table_object,
    flags => rdsadmin.rdsadmin_dbms_repair.noskip_flag);
END;
/

SELECT skip_corrupt 
FROM dba_tables 
WHERE owner = UPPER('&corrupted_Object_Owner') 
AND table_name = UPPER('&corrupted_object_name');
```

### Finally DROP the repair tables:

```sql
EXEC rdsadmin.rdsadmin_dbms_repair.drop_repair_table;
EXEC rdsadmin.rdsadmin_dbms_repair.drop_orphan_keys_table;
```
---
### Auditing:

#### Enable Auditing for all the privileges on SYS.AUD$ table:

```sql
EXEC rdsadmin.rdsadmin_master_util.audit_all_sys_aud_table;
EXEC rdsadmin.rdsadmin_master_util.audit_all_sys_aud_table(p_by_access => true);
```

#### Disable Auditing on SYS.AUD$ table:

```sql
EXEC rdsadmin.rdsadmin_master_util.noaudit_all_sys_aud_table;
```
---
### RMAN Tasks:

#### incremental backup of the current tenant database
```sql
BEGIN
    rdsadmin.rdsadmin_rman_util.backup_tenant_incremental(
        p_owner               => 'SYS', 
        p_directory_name      => 'DATATRN',
        p_level               => 1,
        p_parallel            => 2,  
        p_section_size_mb     => 10,
        p_tag                 => 'MY_INCREMENTAL_BACKUP',
        p_rman_to_dbms_output => FALSE);
END;
/
```
#### backs up all archived redo logs
```sql
BEGIN
    rdsadmin.rdsadmin_rman_util.backup_archivelog_all(
        p_owner               => 'SYS', 
        p_directory_name      => 'DATATRN',
        p_parallel            => 2, 
        p_tag                 => 'LOG_BACKUP_EAYM',
        p_rman_to_dbms_output => FALSE);
END;
/
```

#### Validate the database for Physical/Logical corruption on RDS:

```sql
BEGIN
    rdsadmin.rdsadmin_rman_util.validate_tenant(
        p_validation_type     => 'PHYSICAL+LOGICAL', 
        p_parallel            => 4,  
        p_section_size_mb     => 10,
        p_rman_to_dbms_output => FALSE);
END;
/
```
p_rman_to_dbms_output parameter is set to FALSE, the RMAN output is written to a file in the BDUMP directory.

```sql
SELECT * FROM table(rdsadmin.rds_file_util.listdir('BDUMP')) where filename like '%rds-rman%' ;  
```
To view the files in the BDUMP directory, run the following SELECT statement.
```sql
SELECT text FROM table(rdsadmin.rds_file_util.read_text_file('BDUMP','rds-rman-validate-DATABASE-2025-02-01.22-09-52.150775000.txt'));
 ```           

#### Enable BLOCK CHANGE TRACKING on RDS:

```sql
COL STATUS   FORMAT A8
COL FILENAME FORMAT A60
SELECT STATUS, FILENAME FROM V$BLOCK_CHANGE_TRACKING;
```
```
EXEC rdsadmin.rdsadmin_rman_util.enable_block_change_tracking;
```

#### Disable BLOCK CHANGE TRACKING on RDS:

```sql
EXEC rdsadmin.rdsadmin_rman_util.disable_block_change_tracking;
```
#### To delete expired archive logs, set the first flag to true:
```sql
BEGIN
rdsadmin.rdsadmin_rman_util.crosscheck_archivelog(
p_delete_expired => TRUE,
p_rman_to_dbms_output => FALSE);
END;
/
```
#### Crosscheck and delete expired ARCHIVELOGS:

```sql
EXEC rdsadmin.rdsadmin_rman_util.crosscheck_archivelog(p_delete_expired => TRUE, p_rman_to_dbms_output => TRUE);
```

---
### Oracle Scheduler Jobs Management:

#### List all jobs:

```sql
SELECT OWNER||'.'||JOB_NAME "OWNER.JOB_NAME",
       ENABLED, STATE, FAILURE_COUNT,
       TO_CHAR(LAST_START_DATE,'DD-Mon-YYYY hh24:mi:ss TZR') LAST_RUN,
       TO_CHAR(NEXT_RUN_DATE,'DD-Mon-YYYY hh24:mi:ss TZR') NEXT_RUN,
       REPEAT_INTERVAL,
       EXTRACT(DAY FROM LAST_RUN_DURATION) ||':'||
       LPAD(EXTRACT(HOUR FROM LAST_RUN_DURATION),2,'0')||':'||
       LPAD(EXTRACT(MINUTE FROM LAST_RUN_DURATION),2,'0')||':'||
       LPAD(ROUND(EXTRACT(SECOND FROM LAST_RUN_DURATION)),2,'0') "DURATION(d:hh:mm:ss)"
FROM dba_scheduler_jobs 
ORDER BY ENABLED, STATE, "OWNER.JOB_NAME";
```

#### List AUTOTASK INTERNAL MAINTENANCE WINDOWS:

```sql
SELECT WINDOW_NAME, TO_CHAR(WINDOW_NEXT_TIME,'DD-MM-YYYY HH24:MI:SS') NEXT_RUN,
       AUTOTASK_STATUS STATUS, WINDOW_ACTIVE ACTIVE, 
       OPTIMIZER_STATS, SEGMENT_ADVISOR, SQL_TUNE_ADVISOR 
FROM DBA_AUTOTASK_WINDOW_CLIENTS;
```

#### FAILED JOBS IN THE LAST 24H:

```sql
SELECT JOB_NAME, OWNER, LOG_DATE, STATUS, ERROR#, RUN_DURATION 
FROM DBA_SCHEDULER_JOB_RUN_DETAILS 
WHERE STATUS='FAILED' AND LOG_DATE > SYSDATE-1 
ORDER BY JOB_NAME, LOG_DATE;
```

#### Current running jobs:

```sql
SELECT j.RUNNING_INSTANCE INS,
       j.OWNER||'.'||j.JOB_NAME ||' | '||SLAVE_OS_PROCESS_ID||'|'||j.SESSION_ID "OWNER.JOB_NAME|OSPID|SID",
       s.FINAL_BLOCKING_SESSION "BLKD_BY", ELAPSED_TIME, CPU_USED,
       SUBSTR(s.SECONDS_IN_WAIT||'|'||s.WAIT_CLASS||'|'||s.EVENT,1,45) "WAITED|WCLASS|EVENT", S.SQL_ID
FROM dba_scheduler_running_jobs j, gv$session s
WHERE j.RUNNING_INSTANCE=S.INST_ID(+)
AND j.SESSION_ID=S.SID(+)
ORDER BY "OWNER.JOB_NAME|OSPID|SID", ELAPSED_TIME;
```

#### Disable a job owned by SYS:

```sql
EXEC rdsadmin.rdsadmin_dbms_scheduler.disable('SYS.CLEANUP_NON_EXIST_OBJ');
```

#### Modify the repeat interval of a job:

```sql
BEGIN
  rdsadmin.rdsadmin_dbms_scheduler.set_attribute(
    name      => 'SYS.CLEANUP_NON_EXIST_OBJ',
    attribute => 'repeat_interval',
    value     => 'freq=daily;byday=FRI,SAT;byhour=20;byminute=0;bysecond=0');
END;
```
#### Flash shared pool
```
exec rdsadmin.rdsadmin_util.flush_shared_pool
/
```
