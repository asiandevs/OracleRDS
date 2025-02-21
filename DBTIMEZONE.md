Regarding your RDS instance DBTIMEZONE.

<img width="806" alt="image" src="https://github.com/user-attachments/assets/809502e7-9a03-4186-8902-a63180f77426" />

RDS instance's timezone can be modified in 2 ways:- 
-One is by having a timezone option in your option group. 
-Another is by altering the timezone from the database.

Key difference:-
As mentioned in the below documentation, the Timezone option changes the time zone at the host level and affects all date columns and values such as SYSDATE while the alter_db_time_zone procedure changes the time zone for only certain data types, and doesn't change SYSDATE.

Setting the database time zone
[+]https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.TimeZoneSupport.html 
[+]https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.Timezone.html 


Before doing anything, it was default UTC 
```
SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
UTC
```

Then I modify the value and reboot
```
EXEC rdsadmin.rdsadmin_util.alter_db_time_zone(p_new_tz => 'Australia/Sydney');
```
```
SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
Australia/Sydney
```
But still from the database it was showing UTC timestamp

SQL> select systimestamp from dual;

SYSTIMESTAMP
---------------------------------------------------------------------------
21-FEB-25 01.30.49.252245 AM +00:00

SQL> alter session set nls_date_format='DD-MON-YY HH24:MI:SS';

Session altered.

SQL> select sysdate from dual;

SYSDATE
------------------
21-FEB-25 01:31:10

Then I have added Option [ TIME ZONE] with Australia / Sydney
Replacing option group [Option Group with Option Settings [ TIME_ZONE with Australia / Sydney]


--- 
We're sorry, your request to modify DB instance dev-cdb has failed.
The time zone option that you requested in the new option group (Australia/Sydney) is different from the time zone option of the existing option group (UTC). After you have set the time zone of a DB instance, you cannot change it to a different time zone.
---

You can set to different database time zone as below but database system time will not modify even if you create a OPTION GROUP with Separate Time Zone [Like to UTC back]. You will get above message message 

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.Timezone.html#Appendix.Oracle.Options.Timezone.Zones

SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
Australia/Sydney

SQL> EXEC rdsadmin.rdsadmin_util.alter_db_time_zone(p_new_tz => 'UTC');

PL/SQL procedure successfully completed.

SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
Australia/Sydney

=== Reboot ==

SQL> SELECT dbtimezone FROM DUAL;

DBTIMEZONE
----------------
UTC

## To update database system time after setting a timezone value with option group - follow below steps 
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.Timezone.html
If your DB instance uses the default option group, then follow these steps:

Take a snapshot of your DB instance.

Add the time zone option to your DB instance.

If your DB instance currently uses a nondefault option group, then follow these steps:

Take a snapshot of your DB instance.

Create a new option group.

Add the time zone option to it, along with all other options that are currently associated with the existing option group.

This prevents the existing options from being uninstalled while enabling the time zone option.

Add the option group to your DB instance.
