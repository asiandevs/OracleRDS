# AWS Cloud Migration – Fine-Grained Auditing 

##  Overview

Fine-Grained Auditing (FGA) allows administrators to audit access to specific data based on conditions. This feature is essential for monitoring sensitive data and high-risk actions while reducing the volume of logs by focusing only on relevant events.

## Amazon RDS and Oracle FGA

Amazon RDS for Oracle supports the Oracle Fine-Grained Auditing (FGA) feature.  
However, FGA events stored in `SYS.FGA_LOG$` and accessed via `DBA_FGA_AUDIT_TRAIL` are **not** accessible through standard RDS log access.

Reference: [Security Auditing in Amazon RDS for Oracle: Part 1](https://aws.amazon.com/blogs/database/part-1-security-auditing-in-amazon-rds-for-oracle/)

##  Required RDS Configuration

Ensure your RDS Oracle instance uses a custom parameter group with the following settings:

```sql
audit_sys_operations = TRUE
audit_trail = DB
```

> **Note**: Audit is database-level

---

##  Audit Configuration – Pluggable/tenant Database (e.g., JANGLA  is a pluggable database here)

```sql
SQL> alter session set container=JANGLA;
SQL> show parameter audit;
```

Example output:

| Name | Type | Value |
|------|------|-------|
| audit_file_dest | string | /u01/app/oracle/admin/CHUMKI/adump |
| audit_sys_operations | boolean | TRUE |
| audit_trail | string | DB |
| unified_audit_systemlog | string | LOCAL3.INFO |

---

##  FGA Policy Names

```sql
SQL> select distinct FGA_POLICY_NAME from UNIFIED_AUDIT_TRAIL;
```

FGA_POLICY_NAME appears under the default owner and default tablespace.

---

##  Audit Table Partitioning

```sql
select owner, table_name, interval, partitioning_type, partition_count, def_tablespace_name 
from dba_part_tables where owner='AUDSYS';
```

### Output:

| OWNER | TABLE_NAME | INTERVAL | PARTITION | COUNT | DEF_TABLESPACE_NAME |
|-------|------------|----------|-----------|-------|---------------------|
| AUDSYS | AUD$UNIFIED | INTERVAL '1' MONTH | RANGE | 1048575 | SYSAUX |

---

## Fine-Grained Auditing Capability

```sql
select * from v$option where lower(parameter) like '%audit%' order by 1;
```

| PARAMETER | VALUE | CON_ID |
|-----------|-------|--------|
| Fine-grained Auditing | TRUE | 0 |
| Unified Auditing | FALSE | 0 |

---

## List Enabled Audit Policies

```sql
SELECT * 
FROM audit_unified_enabled_policies 
ORDER BY policy_name, entity_name;
```

### Distinct Policies:

```sql
select distinct policy_name from audit_unified_enabled_policies;
```

- AUD_POL_ANY_SCHEMA_DDL  
- AUD_POL_SCHEMA_DDL  
- ORA_ACCOUNT_MGMT  
- ORA_SECURECONFIG  

---

##  Create Policies on RDS

### Create `AUD_POL_ANY_SCHEMA_DDL`

```sql
create audit policy "AUD_POL_ANY_SCHEMA_DDL" privileges ...;
```

(Include all relevant privileges listed in original document.)

**Enable Policy:**

```sql
audit policy aud_pol_any_schema_ddl except '<LIST_OF_USERS>';
```

### Create `AUD_POL_SCHEMA_DDL`

```sql
create audit policy "AUD_POL_SCHEMA_DDL" privileges ...;
```

**Enable Policy:**

```sql
audit policy aud_pol_schema_ddl except '<LIST_OF_USERS>';
```

---

##  Default RDS Policies

RDS includes:

- ORA_LOGON_FAILURES (All Users)
- ORA_SECURECONFIG (All Users)

Update ORA_SECURECONFIG to use `EXCEPT` clause:

```sql
NOAUDIT policy ORA_SECURECONFIG;
audit policy ORA_SECURECONFIG except '<LIST_OF_USERS>';
```
---

## Enable ORA_ACCOUNT_MGMT

```sql
audit policy ORA_ACCOUNT_MGMT except '<LIST_OF_USERS>';
```

---

## Other Audit SQL Utilities

### List Policies

```sql
select distinct policy_name from audit_unified_enabled_policies;
```

### Generate DDL for Policy

```sql
SELECT to_char(dbms_metadata.get_ddl('AUDIT_POLICY', '<policy_name>')) FROM dual;
```

### Disable Audit Policy

```sql
NOAUDIT POLICY <policy_name>;
```

### Drop Audit Policy

```sql
DROP audit policy <policy_name>;
```

---

## CloudWatch Integration for Extended Log Retention

### Default Retention Limitation

- RDS audit logs retained only for **7 days**
- Cannot be extended on instance directly
- Audit & trace files share the same retention config

---

### CloudWatch Logs Solution

**Benefits:**

- Durable storage
- Log analytics
- Alerts & metrics
- Customizable retention

**Implementation:**

- Configure Oracle RDS to publish logs to CloudWatch
- Logs stream format: `/aws/rds/instance/my_instance/audit`

---

### Audit Trail Configuration

Set `audit_trail` to one of:

- none
- os
- db [, extended]
- xml [, extended]

---

### Best Practices

- Adjust retention per compliance needs
- Monitor storage costs
- Implement log analysis strategy
- Document configurations

---

##  Exporting Audit Data

To export audit tables using **Data Pump**:

```bash
userid='system/<password>'
directory=...
full=yes
include=audit_trails
dumpfile=audit.dmp
```

Reference: [Oracle Support Doc ID 2709550.1](https://support.oracle.com/epmos/faces/DocumentDisplay?parent=SrDetailText&sourceId=3-40295847021&id=2709550.1)
**Require Oracle Support Account**

---


