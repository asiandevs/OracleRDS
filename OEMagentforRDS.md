There are below Prerequisites before enabling OEM Agent in an Amazon RDS for Oracle instance after your setup the OMS host


RDS OPTION GROUP

![image](https://github.com/user-attachments/assets/87a4287d-04c4-49f6-bda6-ea3b0eacec18)

![image](https://github.com/user-attachments/assets/f0bdd048-37f7-4b73-9a78-81a2b1c8752c)

1. From a networking and firewall perspective of OMS, make sure the DB listener port and OEM Agent port are allowed. If OMS is set up on Amazon Elastic Compute Cloud (Amazon EC2), make sure to modify the security group and network access control list (ACL). If OMS is set up on premises, make sure you engage your network team to provide the required access to the DB listener port and OEM Agent port.
2. [root@ip-10-16-144-126 ~]# nc -zv <OMS HOST IP> 4903
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to <OMS HOST IP> :4903.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.

4. Create an inbound rule for the security group of your RDS for Oracle instance for the OMS port and OMS host IP as the source.
5. RDS :
Type 		Protocol 	Port range  	Source
Custom TCP 	 TCP		3872		 Custom		                    10.11.34.110/32
Custom TCP   TCP        1529         Custom          10.11.34.110/32

Where 10.11.34.110 is OMS server IP

![image](https://github.com/user-attachments/assets/1e919162-8ba2-45e4-a52a-cf09109e6a52)


6.  Make sure the OEM Agent version you pick is compatible with the OMS version you have installed. For more information, refer to Accessing the Enterprise Manager Certification Matrix.
7. OEM Agent is supported for Standard Edition 2 and Enterprise Edition of Amazon RDS for Oracle. Refer to Using the Management Agent for further details related to supported editions and versions.

We have added the security group inbound rule from OMS IP address "sg-db01" on port 4903. However, OEM_AGENT installation fails with the below error:

 Issue you may have: 
[-]Error while installing OEM_AGENT. Message: Unable to install the Oracle OEM_AGENT because the DB instance cannot reach the OMS host. Update the option settings, verify the security group configuration and try again. 

Please work with your network admin to configure the connectivity between OMS host and RDS Oracle instance for port 1529 (RDS Oracle) and port 3872 for OEM_AGENT and try the installation again. 
AWS Document for your reference: 
  ++ https://aws.amazon.com/blogs/database/monitor-amazon-rds-for-oracle-instances-using-oracle-enterprise-manager/



https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Oracle.Options.OEMAgent.html#Oracle.Options.OEMAgent.Using


select username, account_status from dba_users where username like '%DBSNMP%'

alter user rdsdbsnmp account unlock

alter user rdsdbsnmp identified by CHUMKI#Golam123;

select username, account_status, profile from dba_users where username='RDSDBSNMP'

         
exec rdsadmin.rdsadmin_util.reset_oem_agent_password('CHUMKI#Golam123');

SELECT rdsadmin.rdsadmin_oem_agent_tasks.restart_oem_agent as TASK_ID from DUAL;
SELECT rdsadmin.rdsadmin_oem_agent_tasks.get_status_oem_agent() as TASK_ID from DUAL; 

SELECT rdsadmin.rdsadmin_oem_agent_tasks.list_targets_oem_agent as TASK_ID from DUAL;

SELECT text FROM table(rdsadmin.rds_file_util.read_text_file('BDUMP','dbtask-1739412924563-1270.log'));


![image](https://github.com/user-attachments/assets/f4d74d90-fe23-4003-a4cd-620f330afba2)



![image](https://github.com/user-attachments/assets/bb006165-1545-4f4d-bb77-61f2ed1fa51e)



![image](https://github.com/user-attachments/assets/bc8c29b6-4ff4-4ddb-8ede-19c040ff709c)

![image](https://github.com/user-attachments/assets/3903045d-7285-49f6-a413-c0e200faf209)



![image](https://github.com/user-attachments/assets/9bed1030-7348-4576-aa4c-f230f2b819b2)

![image](https://github.com/user-attachments/assets/9b13122c-f7a9-48f5-90d1-dc038828e5f1)

![image](https://github.com/user-attachments/assets/1cd3f37b-5051-4cd2-a667-5b0a6630b3ae)



