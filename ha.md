
## Convert to Multi-AZ DB instance deployment

Select the database 
click on Actions
Then click "Convert to Multi-AZ deployment"

There are two options to Schedule database modification
1) Apply during the next scheduled maintenance window
Current maintenance window:
2) Apply immediately
The modifications in this request and any pending modifications will be asynchronously applied as soon as possible, regardless of the maintenance window setting for this database instance.

select the option and click on Convert to Multi-AZ

# Validate 

Select the database
click on Configuration section and validate Multi-AZ [ yes ] on Instance class section

## Create a standby database
Select the database instance and click on Modify
Availability & durability
Multi-AZ deployment
Two options
i) Create a standby instance (recommended for production usage)
Creates a standby in a different Availability Zone (AZ) to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups.
ii) Do not create a standby instance

If you select "Do not create a standby instance" - it will convert database to single AZ.

### 
```

As you have rightly mentioned, the Multi-AZ feature of Amazon RDS automatically provisions and maintains a synchronous standby replica in a different Availability Zone. Since, the data is replicated in real time, the Recovery Point Objective (RPO) is 0.

However, in order to implement zero RPO, the only other option would be if you setup RDS for Custom Oracle, and configure the Dataguard with maximum protection mode. 

Please refer the links below for more information :

[+] https://aws.amazon.com/blogs/database/build-high-availability-for-amazon-rds-custom-for-oracle-using-read-replicas/
[+] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/working-with-custom-oracle.html

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
```
