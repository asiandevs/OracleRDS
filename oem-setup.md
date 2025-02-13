


There are below Prerequisites before enabling OEM Agent in an Amazon RDS for Oracle instance after your setup the OMS host

1. From a networking and firewall perspective of OMS, make sure the DB listener port and OEM Agent port are allowed. If OMS is set up on Amazon Elastic Compute Cloud (Amazon EC2), make sure to modify the security group and network access control list (ACL). If OMS is set up on premises, make sure you engage your network team to provide the required access to the DB listener port and OEM Agent port.
2. Create an inbound rule for the security group of your RDS for Oracle instance for the OMS port and OMS host IP as the source.
3.  Make sure the OEM Agent version you pick is compatible with the OMS version you have installed. For more information, refer to Accessing the Enterprise Manager Certification Matrix.
4. OEM Agent is supported for Standard Edition 2 and Enterprise Edition of Amazon RDS for Oracle. Refer to Using the Management Agent for further details related to supported editions and versions.

We have added the security group inbound rule from OMS IP address "sg-XXX" on port 4903. However, OEM_AGENT installation fails with the below error:
 
[-]Error while installing OEM_AGENT. Message: Unable to install the Oracle OEM_AGENT because the DB instance cannot reach the OMS host. Update the option settings, verify the security group configuration and try again. 

Please work with your network admin to configure the connectivity between OMS host and RDS Oracle instance for port 1521 (RDS Oracle) and port 3872 for OEM_AGENT and try the installation again. 
AWS Document for your reference: 
  ++ https://aws.amazon.com/blogs/database/monitor-amazon-rds-for-oracle-instances-using-oracle-enterprise-manager/ 


<img width="787" alt="image" src="https://github.com/user-attachments/assets/ecdf6b3f-a94f-487e-a6e3-642d278c7be0" />
