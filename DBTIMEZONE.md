# 🌏 Managing Oracle RDS Time Zone Settings on AWS

When working with Amazon RDS for Oracle, it's important to understand how **time zone configuration** affects your application. Whether you're adjusting the time zone for reporting, compliance, or regional business logic, AWS provides two mechanisms to set the time zone—each with distinct behavior and implications.

In this post, we’ll walk through:

- The difference between **host-level** and **database-level** time zone changes
- How to apply each change
- Gotchas to avoid (especially when modifying an existing instance)

---

## 🕰️ Two Ways to Set Time Zone in Oracle RDS

There are **two supported methods** to change the time zone on an Oracle RDS instance:

### ✅ Option 1: Using an Option Group (System-Level Time Zone)

- This modifies the **host-level** time zone.
- Affects **all system-level operations**, including `SYSDATE`, `SYSTIMESTAMP`, and default date values.

### ✅ Option 2: Using PL/SQL to Alter the DB Time Zone (Schema-Level)

- Changes the **database time zone** for certain **timestamp data types**.
- Does **not** affect `SYSDATE` or the system’s underlying clock.

### 🔍 Key Difference

| Feature                  | Option Group (TIMEZONE Option) | `alter_db_time_zone` (PL/SQL) |
|--------------------------|-------------------------------|-------------------------------|
| Changes SYSDATE/SYSTIMESTAMP | ✅ Yes                       | ❌ No                        |
| System-wide impact       | ✅ Yes                       | ❌ Limited to timestamp types |
| Requires reboot          | ✅ Yes                       | ✅ Yes                       |
| Can be reversed easily   | ❌ No (see note below)       | ✅ Yes (but not system time)  |

📖 **AWS Documentation:**
- [Setting DB Time Zone in RDS Oracle](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.TimeZoneSupport.html)
- [RDS Oracle TIMEZONE Option](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.Timezone.html)

---

## 🧪 Example: Step-by-Step Time Zone Configuration

### 🔍 Check the Default DB Time Zone

```sql
SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
UTC
```

### ⚙️ Modify Database Time Zone (PL/SQL)

```sql
EXEC rdsadmin.rdsadmin_util.alter_db_time_zone(p_new_tz => 'Australia/Sydney');
```

### 🔁 Reboot the DB Instance  
After reboot, verify the new time zone:

```sql
SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
Australia/Sydney
```

But… notice this:

```sql
SQL> SELECT systimestamp FROM dual;

SYSTIMESTAMP
-------------------------------
21-FEB-25 01.30.49.252245 AM +00:00

SQL> SELECT sysdate FROM dual;

SYSDATE
-------------------
21-FEB-25 01:31:10
```

🛑 `SYSDATE` and `SYSTIMESTAMP` are still showing UTC!  
That’s because the **host system time** is not affected by `alter_db_time_zone`.

---

## 🧰 Setting Host-Level Time Zone with an Option Group

You can configure this using the `TIMEZONE` option in a custom **Option Group**:

1. Navigate to **Amazon RDS > Option Groups**.
2. Create a new Option Group and add the `TIMEZONE` option with `Australia/Sydney`.
3. Assign this Option Group to your DB instance.

### ⚠️ Warning: Once Set, Cannot Change

<img width="806" alt="image" src="https://github.com/user-attachments/assets/809502e7-9a03-4186-8902-a63180f77426" />


When applying the new time zone option, you might get this error:

> ❌ _"The time zone option that you requested in the new option group (Australia/Sydney) is different from the time zone option of the existing option group (UTC). After you have set the time zone of a DB instance, you cannot change it to a different time zone."_

Even if you later run:

```sql
EXEC rdsadmin.rdsadmin_util.alter_db_time_zone(p_new_tz => 'UTC');
```

The **system time zone** remains unchanged. Only the database time zone will revert, not SYSDATE/SYSTIMESTAMP.

---

## 🔄 Proper Way to Update System Time Zone

If your DB instance uses the **default option group**, follow these steps:

1. 📸 Take a snapshot of your DB instance.
2. ➕ Add the `TIMEZONE` option to your DB instance.

If your DB instance uses a **non-default option group**:

1. 📸 Take a snapshot.
2. 📦 Create a **new Option Group**.
3. ➕ Add the `TIMEZONE` option **alongside all existing options**.
4. 🔄 Attach the new Option Group to your DB instance.

📘 [Full documentation and valid time zone values here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.Timezone.html#Appendix.Oracle.Options.Timezone.Zones)

---

## ✅ Summary

| Task                      | Method                        |
|---------------------------|-------------------------------|
| Set database time zone    | `rdsadmin_util.alter_db_time_zone` |
| Set system time zone      | Option Group with TIMEZONE    |
| Revert to default timezone | Not supported at system level after change |

---

If you're planning to change the time zone for your Oracle RDS, be sure to **understand the distinction** between the two approaches to avoid unexpected behaviors in your applications.

Have any questions or ran into issues with time zone management in RDS? Feel free to reach out—I’m happy to help!

--- 
