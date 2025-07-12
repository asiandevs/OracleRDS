# Monitoring Amazon RDS for Oracle with Oracle Enterprise Manager (OEM)

Amazon Relational Database Service (Amazon RDS) for Oracle is a fully managed database service that simplifies the process of setting up, operating, and scaling Oracle database deployments in the AWS Cloud. By automating routine administrative tasks such as provisioning, backups, patching, monitoring, and scaling, Amazon RDS allows you to focus on more critical application-specific activities.

In this post, we will explore how to install and configure the Oracle Enterprise Manager (OEM) Agent on an Amazon RDS for Oracle instance. We will also walk through the prerequisites, configuration steps, and validation procedures for integrating an RDS instance with Oracle Enterprise Manager.

---

## Prerequisites for Enabling the OEM Agent on Amazon RDS for Oracle

Before enabling the OEM Agent on your Amazon RDS for Oracle instance, you must ensure the following prerequisites are met:

### 1. Oracle Management Service (OMS) Setup  
You are responsible for setting up OMS to monitor the Oracle database. Ensure network connectivity between OMS, the OEM Agent, and the RDS instance.

### 2. Network and Firewall Configuration  
- Open the database listener port and OEM Agent port on OMS.  
- If OMS is running on Amazon EC2, modify the security group and network ACLs.  
- If OMS is on-premises, work with your network team to allow access to the database listener and OEM Agent ports.

#### Validate Connectivity:
```sh
[root@ip- ~]# nc -zv <OMS_HOST_IP> 4903
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to <OMS_HOST_IP> :4903.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.
```

### 3. Security Group Rules  
Create an inbound rule in the security group of your RDS instance to allow traffic from the OMS port and OMS host IP.

![Security Group Configuration](https://github.com/user-attachments/assets/1e919162-8ba2-45e4-a52a-cf09109e6a52)

> **Note:** `10.11.34.110` represents the OMS server IP.

### 4. Compatibility Check  
Ensure the OEM Agent version is compatible with your OMS version. Refer to the [Enterprise Manager Certification Matrix](https://support.oracle.com) for details.

### 5. Supported Editions  
OEM Agent is supported on both **Standard Edition 2** and **Enterprise Edition** of Amazon RDS for Oracle. Refer to the documentation on [Using the Management Agent](https://docs.aws.amazon.com) for more details.

---

## Installing the OEM Agent on an Amazon RDS Instance  

The OEM Agent can be installed by adding the `OEM_AGENT` option to an **option group**. Follow these steps:

1. Navigate to the **Amazon RDS Console** and open the **RDS instance details** page.  
2. Choose the **Option Group** associated with your instance.  
   - If your instance uses the default option group, create a new one or associate an existing option group with OEM enabled.  
3. Click **Add Option** and choose `OEM_AGENT` as the option name.  
4. Provide the necessary OMS configuration details and security group information.  
5. For **Apply Immediately**, select **Yes** and choose **Add Option**.

![Adding OEM Agent Option](https://github.com/user-attachments/assets/87a4287d-04c4-49f6-bda6-ea3b0eacec18)

![OEM Agent Configuration](https://github.com/user-attachments/assets/f0bdd048-37f7-4b73-9a78-81a2b1c8752c)

> **Note:** Adding the `OEM_AGENT` option doesn’t cause any downtime. However, the instance will be in a modifying state until the agent installation is complete.

---

## Adding an RDS Instance as a Target in Oracle Enterprise Manager

Once the OEM Agent is installed, manually add the RDS instance as a target in OEM:

## Step 1: Connect to the RDS Instance  

1. Connect to the RDS instance as a user with the necessary privileges.  
2. Unlock and set a new password for the `DBSNMP` user:  

   ```sql
   SELECT username, account_status 
   FROM dba_users 
   WHERE username LIKE '%DBSNMP%';
   ```

## Step 2: Reset the `DBSNMP` Password  

```sql
EXEC rdsadmin.rdsadmin_util.reset_oem_agent_password('HUYTFV$%gytre');
```

```sql
SELECT rdsadmin.rdsadmin_oem_agent_tasks.restart_oem_agent AS TASK_ID 
FROM DUAL;
```
```sql
SELECT rdsadmin.rdsadmin_oem_agent_tasks.get_status_oem_agent() as TASK_ID from DUAL; 
```
## Collect Task ID and Check Status  

```sql
SELECT text 
FROM table(rdsadmin.rds_file_util.read_text_file('BDUMP', 'dbtask-<taskid>.log'));
```

## Step 3: Add the RDS Instance in OEM  

Open OEM, navigate to:  
   **Setup > Add Target > Add Targets Manually**  

   ![OEM Setup](https://github.com/user-attachments/assets/f4d74d90-fe23-4003-a4cd-620f330afba2)

   ![Add Targets](https://github.com/user-attachments/assets/bb006165-1545-4f4d-bb77-61f2ed1fa51e)

   ![Select Database](https://github.com/user-attachments/assets/bc8c29b6-4ff4-4ddb-8ede-19c040ff709c)

   ![Agent Configuration](https://github.com/user-attachments/assets/3903045d-7285-49f6-a413-c0e200faf209)

## Step 4: Enter Target Details  

```sql
-- Agent Host: The database identifier (e.g., testenv)
-- Target Type: Database Instance
-- Monitoring Password: DBSNMP user password
-- Oracle Home Path: /oracle
```

   ![Enter Details](https://github.com/user-attachments/assets/9bed1030-7348-4576-aa4c-f230f2b819b2)

```
Target Name	<SID>
Database System	<DBSYSTEM>
 
Name
Value
 	
Monitoring Username	dbsnmp
Monitoring Password	******
Role	NORMAL
Oracle Home Path	/rdsdbbin/oracle
Listener Machine Name	<<DB endpoint>>
Port	<<DB Listener Port>>
Connection Protocol	TCP
Database SID	<DB Name>
```
## Step 5: Final Configuration  

```sql
-- Configure other database properties such as port, protocol, and SID.
-- Click Test Connection to verify the entered values.
-- Submit to save the target.
```

   ![Validate Target](https://github.com/user-attachments/assets/9b13122c-f7a9-48f5-90d1-dc038828e5f1)

   ![Final Configuration](https://github.com/user-attachments/assets/1cd3f37b-5051-4cd2-a667-5b0a6630b3ae)

---

## Troubleshooting Common Issues

### Agent Installation Failures  
If the OEM Agent installation fails, check the **Logs & Events** tab on the Amazon RDS console for error messages in the **Recent Events** section. Common causes include:  
- **Network Communication Failure**: Ensure OMS can communicate with the RDS instance.  
- **Configuration Errors**: Export agent logs (e.g., `emctl.log`, `gcagent.log`) to Amazon CloudWatch Logs for further analysis.  

Refer to the [troubleshooting guide](https://docs.aws.amazon.com) for more details.

```
SELECT rdsadmin.rdsadmin_oem_agent_tasks.get_status_oem_agent() as TASK_ID from DUAL;   
```
```
SELECT text FROM table(rdsadmin.rds_file_util.read_text_file('BDUMP','dbtask-1740965836354-2.log'));
```
---

## Modifying or Deleting the OEM Agent Configuration

When modifying the OEM Agent configuration, such as changing the OMS port or agent version, follow these precautions:  
- Backup your database and decommission the targets from OMS.  
- Clear target-specific information in OMS if you delete and restore an RDS instance with the same identifier.  

The `Targets.xml` file contains static entries for all monitored targets. Ensure it is properly updated during reconfiguration.

---

## Cleanup

To revert the changes and clean up resources:  
1. Reassign the RDS instance to its default option group.  
2. Remove the OEM Agent option from any non-default option groups.  
3. Delete unnecessary AWS resources, such as security groups or EC2 instances created for OMS configuration.

---

## Conclusion

In this post, we covered how to enable and configure the OEM Agent for Amazon RDS for Oracle and register the instance in Oracle Enterprise Manager. Proper setup and configuration allow you to effectively monitor and manage your RDS instances using OEM.  

---

**References**  
- [Oracle Documentation](https://docs.oracle.com)  
- [AWS RDS for Oracle Documentation](https://docs.aws.amazon.com)
- [Agent Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Options.OEMAgent.html)

