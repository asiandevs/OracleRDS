
Connection Issue
https://repost.aws/knowledge-center/rds-oracle-connection-errors

Modify SID name
https://repost.aws/knowledge-center/rds-oracle-change-sid-instance

Huge Pages 
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Concepts.HugePages.html


Issue on DMS:
Endpoint identifier
Replication instance
Status
Message
dmsuatuaus
reptst01
failed
Test Endpoint failed: Application-Status: 1020912, Application-Message: Log Miner is not supported in Oracle PDB environment Endpoint initialization failed.

<img width="1551" alt="image" src="https://github.com/user-attachments/assets/0614ff54-a67e-453d-8ff1-cf4d277fac1b" />


Database Features:
Overview of RDS for Oracle CDBs - Amazon Relational Database Service

Database init parameter and options

The following features aren't supported at the PDB level but are supported at the CDB level:

Option groups (options are installed on all PDBs on your CDB instance)

Parameter groups (all parameters are derived from the parameter group associated with your CDB instance)

Reference

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Concepts.CDBs.html#Oracle.Concepts.single-tenant-limitations

In Transit - encryption

You enable SSL encryption for an Oracle RDS database instance by adding the Oracle SSL option to the option group associated with an Oracle DB instance.
o enable SSL encryption for an RDS for Oracle DB instance, add the Oracle SSL option to the option group associated with the DB instance. Amazon RDS uses a second port, as required by Oracle, for SSL connections. This approach allows both clear text and SSL-encrypted communication to occur at the same time between a DB instance and SQL*Plus.

Oracle Secure Sockets Layer - Amazon Relational Database Service

Connecting to an RDS for Oracle DB instance using SSL - Amazon Relational Database Service

SSL connection

Additional configuration for using Oracle 12 and 19 databases with SSL/TLS connection | Jive Documentation

Amazon RDS FAQs | Cloud Relational Database | Amazon Web Services



RAC:
There are different high availability features are available in AWS RDS but not Oracle RAC features offered.
Oracle Database@AWS allows customers to access Oracle Autonomous Database and Oracle Exadata Database Service on AWS, providing a unified experience between Oracle Cloud Infrastructure (OCI) and AWS. Exadata by default RAC setup with different database logic in place to process application request much faster. 

https://aws.amazon.com/blogs/database/migrate-from-oracle-rac-to-aws-alternatives-on-aws/

READ Replica
===============

I observed around 3-7 minutes lag in the latest restore time (LRT) for your RDS Oracle instance. Hence, how you can achieve zero RPO (Recovery Point Objective) without implementing Multi-AZ or a Physical Standby Database.

The Multi-AZ feature of Amazon RDS automatically provisions and maintains a synchronous standby replica in a different Availability Zone. Since, the data is replicated in real time, the Recovery Point Objective (RPO) is 0.

However, in order to implement zero RPO, the only other option would be if you setup RDS for Custom Oracle, and configure the Dataguard with maximum protection mode.

Regarding the reason for the lag in LRT, copy operations take a few moments to complete and are initiated every 5 minutes on the RDS DB instance. Hence, the typical RPO for the RDS instance in its source Region is approximately 5 to 10 minutes. Having said that, all other options available for RDS Oracle Disaster Recovery are near zero RPO but will not be complete zero RPO. 

The other options to consider which have RPO in minutes are as follows :

[1] RDS for Oracle Read Replicas

This solution requires Oracle Enterprise Edition (EE) and offers two distinct configuration options :
   
   a) Read-only mode: This configuration requires an Active Data Guard license and allows for read operations on the replica database. It's ideal for scenarios where you need to offload read operations while maintaining DR capabilities.
   
   b) Mounted mode: This option doesn't require an Active Data Guard license. The replica remains in a mounted state, ready for promotion when needed. This configuration offers a cost-effective DR solution with minimal resource consumption.

Recommended Solution :

For your specific requirement of RPO, I recommend implementing Oracle read replicas in mounted mode. I understand that this approach is almost similar to having physical standby but this has key benefits.

This solution offers several key advantages :

- Faster failover capabilities compared to creating new instances
- Minimal resource consumption during standby periods
- Quick promotion to primary when needed
- Simplified management through RDS managed service

Important Considerations :

- Mounted replicas cannot accept user connections or serve read-only workloads
- Cross-region replicas are only supported with Oracle Enterprise Edition
- For scenarios requiring greater control, Amazon RDS Custom provides administrative access to both database and operating system

[2] Setup a replication from RDS Oracle instance from a region to another via AWS Database Migration Service (DMS) or Oracle GoldenGate.

DMS is a general-purpose tool that enables heterogeneous migration between different database engines however you can use it to perform homogeneous migration as well. Oracle GoldenGate is a database migration tool that can be used with RDS for Oracle instances. Unlike Multi-AZ solution, this solution will have some database loss which depends on the replication lag.

[+] https://aws.amazon.com/dms/ 
[+] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.OracleGoldenGate.html 

While neither option provides true zero RPO, the mounted read replica configuration offers the best balance of functionality, cost, and management overhead within the constraints of RDS Oracle.

Please refer the below documentation for the details of RPO for all available RDS engines :

[+] https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-database-disaster-recovery/choosing-database.html 
 
LIMITATION:

RDS for Oracle supports Data Guard read replicas for Oracle Database 19c and 21c CDBs in the single-tenant configuration only. 

https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-read-replicas.overview.html#oracle-read-replicas.overview.modes

 
 Different Create date
 ====================
It appears that the variation in creation dates is due to the DBCA templates used during database provisioning. When creating an RDS Oracle instance, different templates result in different “created” timestamps. 

In the testing environment, they observed the following:

For a test database created on March 9th:
• Oracle Version: 19.0.0.0.ru-2025-01.rur-2025-01.r1
• SELECT created FROM V$DATABASE; → 2025-01-24 08:33:21
• ALTER SESSION SET nls_date_format='DD-MON-YY HH24:MI:SS'; SELECT sysdate FROM dual; → 13-MAR-25 16:16:45

• Oracle Version: 19.0.0.0.ru-2024-04.rur-2024-04.r1
• SELECT created FROM V$DATABASE; → 2024-04-19 19:44:47
• ALTER SESSION SET nls_date_format='DD-MON-YY HH24:MI:SS'; SELECT sysdate FROM dual; → 13-MAR-25 16:15:39

This difference in timestamps is expected and does not impact your licensing. The variations occur due to the use of different templates when selecting different Oracle versions.

OMS ISSUE
===========

Thank you for the agent status output, the following output means the agent could not communicate back to OMS:
   Last attempted heartbeat to OMS              : 2025-03-03 21:52:15
   Last successful heartbeat to OMS             : (none)

Check RDS can resolve the oms host on DNS example 
SQL> SELECT UTL_INADDR.get_host_address('OMS_Host_Name') FROM dual;
   If cannot resolve, test specifying the OMS tcpip address instead of its name in the option group.
Please note that as RDS instance qat-cdb has been deleted at 2025-03-04 00:42:41 UTC , I cannot review rds agent, rds instance with its option group.

References:
[R1] Troubleshooting oem agent https://repost.aws/knowledge-center/rds-oracle-oem-agent-errors 
[R2] RDS oem agent https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Options.OEMAgent.html 


incompatible_parameter status
==============================
An Amazon RDS DB instance in the incompatible-parameters state means that least one of the parameters in the associated group is set with a value that's not compatible with the current engine version or DB instance class. [1]

This can be caused by:

A DB instance that's scaled to use an instance type that has less memory available than the previous one. At least one of the memory settings in the associated parameter group exceeds the memory size available for the current DB instance.
    
A database engine that's upgraded to a different version. The engine is no longer compatible with one or more parameter settings of the current custom parameter group.

Configurations can fail if you attempt to associate a different parameter group, scale the DB instance type, change the engine version, or modify the DB instance configuration. To accept a new configuration, DB instances must be in the available state. If the DB instance is in an incompatible-parameters state, then you can only reboot or delete it. 

To resolve the issue, you can try to identify what the parameter is that is causing this error. For information about how to determine which values are incompatible, see How do I identify which Amazon RDS DB parameters are in custom parameter groups and which are in default parameter groups? [2] The alternative which is simpler, is to temporarily change back to the default parameter group for that engine version and reset the parameters. This should resolve the error and give you a chance to compare the parameter groups to see what potentially was the issue.

References:
[1] How can I fix an Amazon RDS DB instance that is stuck in the incompatible-parameters status? - https://repost.aws/knowledge-center/rds-incompatible-parameters 
[2] How do I identify which Amazon RDS DB parameters are in custom parameter groups and which are in default parameter groups? - https://repost.aws/knowledge-center/default-custom-groups 


Stop Running Backup
======================
you would like to stop an ongoing backup job. Please be noted, this is not possible to do. However, if you want to disable the automated backups completely from occuring in future, that is possible and I can fetch the steps for that.

To further dive deep, please share the ARN of the DB instance.

[+] Getting an existing ARN: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Tagging.ARN.html#USER_Tagging.ARN.Getting 


RESTORE
=========
You can restore a DB instance to a specific point in time, creating a new DB instance without modifying the source DB instance.

The documentation on how to restore a database to a point in time using our APIs.
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html


Flashback Technology
====================
Amazon RDS for Oracle supports flashback table, flashback query, flashback transaction, and Oracle flashback features, but does not support flashback database to restore the entire database to a specific point-in-time. 

The documentation of the various flashback technologies ( flashback table, flashback query, flashback transaction, flashback database) in Oracle and what is supported and what is not.
https://aws.amazon.com/blogs/database/alternatives-to-the-oracle-flashback-database-feature-in-amazon-rds-for-oracle/

For Flashback Table:
enable table level row movement
enable dba_recyclebin


DMS
=====
DMS is not getting the target endpoint if RDS with Multitenant database.

You can use the following data stores as target endpoints for data migration using AWS DMS.

On-premises and Amazon EC2 instance databases

Oracle versions 10g, 11g, 12c, 18c, and 19c for the Enterprise, Standard, Standard One, and Standard Two editions


You can upgrade to Oracle Database 21c and higher only if your DB engine uses the multitenant architecture.

For a non CDB first we need to convert to a CDB architecture and only then perform an upgrade in a separate operation.

Rather than the AWS-provided name, we want it to be short and simple to remember

If you have your Route53 hosted zone, it will be as simple as:

Create a CNAME record and point it to the DNS name of your RDS instance [How to make custom DNS for RDS instance? ]


FSx
====
integrating FSx mount point and creating an oracle directory on top if it for your ETL purpose

Natively we cant integrate FSx and RDS oracle as there is no option group available.
===============
Replication steps:-
===============
1) Created an RDS oracle instance of same version- ' 19.0.0.0.ru-2025-01.rur-2025-01.r1'
2) Created an FSx file system.
3) Created an EC2 (amazon linux) instance to communicate between RDS and FSx.
4) Created a new directory and mounted the FSX filesystem to that directory.

mkdir -p /bharath

ec2-user@ip-172-xx-1-xxx ~]$ df -h
Filesystem                                               Size  Used Avail Use% Mounted on
devtmpfs                                                 4.0M     0  4.0M   0% /dev
tmpfs                                                    475M     0  475M   0% /dev/shm
tmpfs                                                    190M  448K  190M   1% /run
/dev/xvda1                                               8.0G  1.9G  6.2G  23% /
tmpfs                                                    475M     0  475M   0% /tmp
/dev/xvda128                                              10M  1.3M  8.7M  13% /boot/efi
fs-xxxxxxxxxxxx.amazonaws.com:/fsx/   64G   54M   64G   1% /bharath
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

SQL> create directory test as '/bharath';
create directory test as '/bharath'
*
ERROR at line 1:
ORA-04088: error during execution of trigger 'RDSADMIN.RDS_DDL_TRIGGER2'
ORA-00604: error occurred at recursive SQL level 1
ORA-20900: Invalid path used for directory: /bharath
ORA-06512: at "RDSADMIN.RDSADMIN_TRIGGER_UTIL", line 714
ORA-06512: at line 1
ORA-06512: at line 12
-------------------------------

==========
Conclusion:-
==========
=> As discussed initially, as RDS is an managed service, if anything need to be done/required OS level activity those will be provided as part of option group. All those integration through option group provided necessary OS level permission by providing a predefined oracle stored procedure and functions. Thus whenever you are trying to create directory using EFS integration, the predefined procedure/function have access to underlying OS filesystem.

=> However, in FSx we need to manually create a oracle directory by mentioning the OS level (FSX mount) directory. And while doing so, the predefined RDS trigger is denying us due to  security reasons.

ORA-04088: error during execution of trigger 'RDSADMIN.RDS_DDL_TRIGGER2'
ORA-20900: Invalid path used for directory: /bharath

=> So it is always recommended to use either S3 or EFS for additional export/import/ETL process.

