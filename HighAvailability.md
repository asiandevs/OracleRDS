
## Disaster Recovery and RPO Strategies for Amazon RDS Oracle

The **Multi-AZ** feature of Amazon RDS automatically provisions and maintains a **synchronous standby replica** in a different Availability Zone. Since data is replicated in real-time, the **Recovery Point Objective (RPO)** is effectively **0**.

![image](https://github.com/user-attachments/assets/443b5db8-aa1e-449d-991c-1a5a0a472573)

## Convert to Multi-AZ DB instance deployment

Select the database 
click on Actions
Then click "Convert to Multi-AZ deployment"

![image](https://github.com/user-attachments/assets/0b5cdfb3-7c06-411a-9e2f-4f679d15637a)

There are two options to Schedule database modification
1) Apply during the next scheduled maintenance window
Current maintenance window:
2) Apply immediately
The modifications in this request and any pending modifications will be asynchronously applied as soon as possible, regardless of the maintenance window setting for this database instance.

select the option and click on Convert to Multi-AZ
![image](https://github.com/user-attachments/assets/01fa2515-53f6-4481-a2bc-b8fd0671194f)

# Validate 

Select the database
click on Configuration section and validate Multi-AZ [ yes ] on Instance class section

![image](https://github.com/user-attachments/assets/e172a51b-ea07-4b9d-af97-278fcff5c73f)

## Create a standby database
- Select the database instance and click on Modify
- Availability & durability
- Multi-AZ deployment
    - Create a standby instance (recommended for production usage)
Creates a standby in a different Availability Zone (AZ) to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups.
  - Do not create a standby instance

**If you select "Do not create a standby instance" - it will convert database to single AZ.**

---

### Custom Oracle with Data Guard

To implement **true zero RPO**, another option is to use **Amazon RDS Custom for Oracle** and configure **Data Guard in Maximum Protection Mode**.

🔗 [Build high availability for Amazon RDS Custom for Oracle using read replicas](https://aws.amazon.com/blogs/database/build-high-availability-for-amazon-rds-custom-for-oracle-using-read-replicas/)  
🔗 [Amazon RDS Custom for Oracle documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/working-with-custom-oracle.html)

---

## Understanding RPO Lag with LRT

Lag in **Log Recovery Time (LRT)** is typically caused by the nature of **copy operations**, which occur every **5 minutes** on the RDS DB instance. As a result, the typical RPO for an RDS instance in its source region is approximately **5 to 10 minutes**.

Other **Disaster Recovery** options available for Amazon RDS Oracle offer **near-zero RPO**, but **not zero**.

---

## RPO in Minutes: Recommended Alternatives

### 1. Amazon RDS for Oracle Read Replicas

Requires **Oracle Enterprise Edition (EE)** and provides two configuration modes:

#### a) Read-Only Mode  
- **Requires Active Data Guard license**  
- Allows read operations on the replica  
- Ideal for **read offloading + DR**

#### b) Mounted Mode (Recommended)  
- **No Active Data Guard license required**  
- Replica is in **mounted state**, ready for promotion  
- Ideal for **cost-effective DR**

##### Benefits of Mounted Mode:
- Fast failover compared to creating new instances  
- Minimal resource consumption during standby  
- Quick promotion to primary  
- RDS-managed and simplified

##### Important Notes:
- Mounted replicas **do not** accept user connections  
- **Cross-region replicas** are supported **only** with Oracle EE  
- For greater administrative control, consider **Amazon RDS Custom**

---

### 2. Cross-Region Replication via DMS or Oracle GoldenGate

You can also replicate data between regions using:

- **AWS Database Migration Service (DMS)**:  
  General-purpose migration, supports heterogeneous and homogeneous setups

- **Oracle GoldenGate**:  
  Enterprise-grade replication tool compatible with RDS for Oracle

💡 Note: Unlike Multi-AZ, this approach **may result in some data loss** depending on replication lag.

🔗 [AWS DMS](https://aws.amazon.com/dms/)  
🔗 [Oracle GoldenGate with Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.OracleGoldenGate.html)

---

## Summary: Best Option for Balanced RPO

While neither read replicas nor replication via DMS/GoldenGate provides **true zero RPO**, using **mounted read replicas** strikes the best balance of:

- Functionality  
- Cost  
- Management simplicity

---

## Additional Reference

For a comprehensive view of **RPO across all RDS database engines**, refer to:

🔗 [Choosing the right disaster recovery strategy – AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-database-disaster-recovery/choosing-database.html)

---
