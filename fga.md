AWS Cloud Migration
/
Fine-grained Auditing (Production database - JANGLA)


Owned by Monowar Mukul

Last updated: just a moment ago

Fine-Grained Auditing (FGA) allows administrators to audit access to specific data based on conditions. This feature is essential for monitoring sensitive data and high-risk actions, reducing the volume of logs by focusing only on relevant events.

Amazon RDS for Oracle database log files - Amazon Relational Database Service The Oracle audit files provided are the standard Oracle auditing files. Amazon RDS supports the Oracle fine-grained auditing (FGA) feature. However, log access doesn't provide access to FGA events that are stored in the SYS.FGA_LOG$ table and that are accessible through the DBA_FGA_AUDIT_TRAIL view.

Security auditing in Amazon RDS for Oracle: Part 1 | Amazon Web Services 

In RDS : make sure custom parameter group has 
audit_sys_operation=TRUE and 

audit trail=DB


Audit is database level


SQL> alter session set container=JANGLA;
Session altered.
SQL> show parameter audit
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /u01/app/oracle/admin/CHUMKI/
                                                 adump
audit_sys_operations                 boolean     TRUE
audit_syslog_level                   string
audit_trail                          string      DB
unified_audit_common_systemlog       string
unified_audit_sga_queue_size         integer     1048576
unified_audit_systemlog              string      LOCAL3.INFO
SQL> select distinct FGA_POLICY_NAME from UNIFIED_AUDIT_TRAIL;
FGA_POLICY_NAME
--------------------------------------------------------------------------------
 

FGA_POLICY_NAME

It is under default owner and default tablespace



-- on-premise n-premise 
SQL> select owner,table_name,interval,partitioning_type,partition_count,def_tablespace_name from dba_part_Tables where owner='AUDSYS';
OWNER      TABLE_NAME      INTERVAL             PARTITION PARTITION_COUNT DEF_TABLESPACE_NAME
---------- --------------- -------------------- --------- --------------- ------------------------------
AUDSYS     AUD$UNIFIED     INTERVAL '1' MONTH   RANGE             1048575 SYSAUX
 



RDS : 
OWNER      TABLE_NAME    INTERVAL                PARTITION PARTITION_COUNT DEF_TABLESPACE_NAME
--------- --------------- ------------------------------
AUDSYS     AUD$UNIFIED   INTERVAL '1' MONTH     RANGE             1048575 SYSAUX
Fine-gained auditing is enabled



-- on-premise
-- on-premise
SQL> select * from v$option where lower(parameter) like '%audit%' order by 1;
PARAMETER                      VALUE                              CON_ID
------------------------------ ------------------------------ ----------
Fine-grained Auditing          TRUE                                    0
Unified Auditing               FALSE                                   0


RDS:
PARAMETER                                                        VALUE                                                                CON_ID
---------------------------------------------------------------- ---------------------------------------------------------------- ----------
Fine-grained Auditing                                            TRUE                                                                      0
Unified Auditing                                                 FALSE                                                                     0
Unified Auditing Policies



-- on-premise
-- on-premise
COLUMN POLICY_NAME     FORMAT A30                                 
COLUMN ENABLED_OPTION  FORMAT A20                                 
COLUMN ENTITY_NAME     FORMAT A35                                 
COLUMN ENTITY_TYPE     FORMAT A10                                 
COLUMN SUCCESS FORMAT A10  
set pages 200
set lines 2000
SELECT * 
FROM audit_unified_enabled_policies 
order by policy_name,entity_name;
~
POLICY_NAME                    ENABLED_OPTION       ENTITY_NAME                         ENTITY_TYP SUCCESS    FAI
------------------------------ -------------------- ----------------------------------- ---------- ---------- ---



---list the distinct policies in on-premise system
select distinct policy_name
from audit_unified_enabled_policies;
AUD_POL_ANY_SCHEMA_DDL
AUD_POL_SCHEMA_DDL
ORA_ACCOUNT_MGMT
ORA_SECURECONFIG
Most of them are Oracle Default schemas related. There are four policies other than oracle default schemas as below : 

RDS


---create one by one in RDS
----CREATE POLICY AUD_POL_ANY_SCHEMA_DDL;
create audit policy "AUD_POL_ANY_SCHEMA_DDL" privileges create any cluster,
                                                        alter any cluster,
                                                        drop any cluster,
                                                        create any index,
                                                        alter any index,
                                                        drop any index,
                                                        create any synonym,
                                                        drop any synonym,
                                                        create any view,
                                                        drop any view,
                                                        create any sequence,
                                                        alter any sequence,
                                                        drop any sequence,
                                                        drop any role,
                                                        alter any role,
                                                        create any trigger,
                                                        alter any trigger,
                                                        drop any trigger,
                                                        create any materialized view,
                                                        alter any materialized view,
                                                        drop any materialized view,
                                                        create any directory,
                                                        drop any directory,
                                                        create any type,
                                                        alter any type,
                                                        drop any type,
                                                        create any indextype,
                                                        alter any indextype,
                                                        drop any indextype,
                                                        create any context,
                                                        drop any context,
                                                        create any rule set,
                                                        alter any rule set,
                                                        drop any rule set,
                                                        create any rule,
                                                        alter any rule,
                                                        drop any rule,
                                                        drop any sql profile,
                                                        alter any sql profile,
                                                        create any sql profile,
                                                        drop any sql translation profile;
---Enable audit
audit policy aud_pol_any_schema_ddl
    except 'ANONYMOUS',
'APPQOSSYS',
'AUDSYS',
'DBSFWUSER',
'DBSNMP',
'DIP',
'GGSYS',
'GSMADMIN_INTERNAL',
'GSMCATUSER',
'GSMUSER',
'OUTLN',
'REMOTE_SCHEDULER_AGENT',
'SYS',
'SYS$UMF',
'SYSBACKUP',
'SYSDG',
'SYSKM',
'SYSRAC',
'SYSTEM',
'XDB',
'XS$NULL';
--validate 
select *
  from audit_unified_enabled_policies
 where policy_name = 'AUD_POL_ANY_SCHEMA_DDL';


---create policy AUD_POL_SCHEMA_DDL
create audit policy "AUD_POL_SCHEMA_DDL" privileges create table,
                                                    create synonym,
                                                    create view,
                                                    create sequence,
                                                    create database link,
                                                    create public database link,
                                                    drop public database link,
                                                    create procedure,
                                                    create trigger,
                                                    create materialized view,
                                                    create type,
                                                    create indextype,
                                                    create job actions create index,
                                                                       drop index,
                                                                       alter index,
                                                                       drop table,
                                                                       alter sequence,
                                                                       alter table,
                                                                       drop sequence,
                                                                       drop synonym,
                                                                       drop view,
                                                                       alter procedure,
                                                                       alter trigger,
                                                                       drop trigger,
                                                                       drop procedure,
                                                                       drop type,
                                                                       alter type,
                                                                       create type body,
                                                                       alter type body,
                                                                       drop type body,
                                                                       alter view,
                                                                       create function,
                                                                       alter function,
                                                                       drop function,
                                                                       create package,
                                                                       alter package,
                                                                       drop package,
                                                                       create package body,
                                                                       alter package body,
                                                                       drop package body,
                                                                       drop indextype,
                                                                       alter indextype,
                                                                       alter synonym;
---Enable audit for the policy AUD_POL_SCHEMA_DDL
audit policy aud_pol_schema_ddl
    except 
'ANONYMOUS',
'APPQOSSYS',
'AUDSYS',
'DBSFWUSER',
'DBSNMP',
'DIP',
'GGSYS',
'GSMADMIN_INTERNAL',
'GSMCATUSER',
'GSMUSER',
'OUTLN',
'REMOTE_SCHEDULER_AGENT',
'SYS$UMF',
'SYS',
'SYSBACKUP',
'SYSDG',
'SYSKM',
'SYSRAC',
'SYSTEM',
'XDB',
'XS$NULL';
--validate 
select *
  from audit_unified_enabled_policies
 where policy_name = 'AUD_POL_SCHEMA_DDL';
/


RDS: Below two policies created by default. We need to enable ORA_SECURECONFIG with EXCEPT option as default is ALL USERS but on-premise has with EXCEPT option.
POLICY_NAME                    ENABLED_OPTION       ENTITY_NAME                         ENTITY_TYP SUCCESS    FAI
------------------------------ -------------------- ----------------------------------- ---------- ---------- ---
ORA_LOGON_FAILURES             BY USER              ALL USERS                           USER       NO         YES
ORA_SECURECONFIG               BY USER              ALL USERS                           USER       YES        YES


--Enable Audit for the policy ORA_ACCOUNT_MGMT
audit policy ORA_ACCOUNT_MGMT
    except 
'ANONYMOUS',
'APPQOSSYS',
'AUDSYS',
'DBSFWUSER',
'DBSNMP',
'DIP',
'GGSYS',
'GSMADMIN_INTERNAL',
'GSMCATUSER',
'GSMUSER',
'OUTLN',
'REMOTE_SCHEDULER_AGENT',
'SYS$UMF',
'SYS',
'SYSBACKUP',
'SYSDG',
'SYSKM',
'SYSRAC',
'SYSTEM',
'XDB';
--validate 
select *
  from audit_unified_enabled_policies
 where policy_name = 'ORA_ACCOUNT_MGMT';


--disable the audit which is configured for all the users
NOAUDIT policy ORA_SECURECONFIG;
---Enable audit for the policy ORA_SECURECONFIG
audit policy ORA_SECURECONFIG
    except 
'ANONYMOUS',
'APPQOSSYS',
'AUDSYS',
'DBSFWUSER',
'DBSNMP',
'DIP',
'GGSYS',
'GSMADMIN_INTERNAL',
'GSMCATUSER',
'GSMUSER',
'OUTLN',
'REMOTE_SCHEDULER_AGENT',
'SYS$UMF',
'SYS',
'SYSBACKUP',
'SYSDG',
'SYSKM',
'SYSRAC',
'SYSTEM';
--validate 
select *
  from audit_unified_enabled_policies
 where policy_name = 'ORA_ACCOUNT_MGMT';
 

Other SQLs to manage Audit (sample SQLs)



---list the distinct policies in on-premise system
select distinct policy_name
  from audit_unified_enabled_policies;
AUD_POL_ANY_SCHEMA_DDL
AUD_POL_SCHEMA_DDL
ORA_ACCOUNT_MGMT
ORA_SECURECONFIG
--to get the ddl use the below 
SELECT to_char(dbms_metadata.get_ddl('AUDIT_POLICY', 'audit policy name'))
FROM dual;
to diable audit poicy
 NOAUDIT POLICY <<audit policy name>>;
To drop audit policy 
 DROP audit policy <<audit policy name>>;
