#  Automating Oracle RDS Refreshes Across AWS Accounts

Refreshing an Amazon RDS Oracle database from a production account to a non-production account can be a time-consuming and error-prone manual task. Fortunately, AWS offers a variety of native services and third-party solutions to **automate cross-account database refreshes** with ease and security.

In this blog post, we’ll explore **two main options** to achieve this—using **AWS native services** with Lambda and Step Functions, or leveraging a **fully packaged marketplace solution**. We'll also share helpful resources and templates to get you started quickly.

---

##  Option 1: Native AWS Services (Lambda, Step Functions, EventBridge)

Automating the refresh of an Oracle RDS database between AWS accounts involves orchestrating snapshot creation, cross-account sharing, and restoration in a target account.

###  High-Level Steps

#### 1. Set Up AWS Backup  
- Enable **automated backups** for your Oracle RDS instance in the **source account**.  
- Configure **cross-account backup policies** to allow snapshot copies to the **destination account**.

#### 2. Create AWS Lambda Functions  
- Develop Lambda functions to automate snapshot creation, copying, and restoring.  
- Use **AWS SDKs (e.g., Boto3)** to interact with RDS and Backup APIs.

#### 3. Set Up AWS Step Functions  
- Design a **state machine** to orchestrate:
  - Snapshot creation
  - Cross-account copy
  - Restoration
- Each step should handle retry logic and failure notifications.

#### 4. Configure Amazon EventBridge  
- Set up **scheduled triggers** or conditional events to run the workflow periodically.  
- Manage lifecycle events and cleanup tasks efficiently.

#### 5. Implement Security Measures  
- Use **AWS KMS** for snapshot encryption.  
- Share encryption keys across accounts to permit secure snapshot access.

###  Reference Solutions

Check out the following detailed guides from AWS:

- [Automate cross-account RDS Oracle backups including DB parameter groups, option groups, and security groups](https://aws.amazon.com/blogs/database/automate-cross-account-backup-of-amazon-rds-for-oracle-including-database-parameter-groups-option-groups-and-security-groups/)
- [Automate cross-account backups of RDS and Aurora databases using AWS Backup](https://aws.amazon.com/blogs/database/automate-cross-account-backups-of-amazon-rds-and-amazon-aurora-databases-with-aws-backup/)

---

##  Option 2: Use a Marketplace Solution – CirrusHQ

If you're looking for a **ready-to-use, plug-and-play solution**, the **Aurora and RDS Automated Database Refresh** tool by **CirrusHQ** on AWS Marketplace offers exactly that.

###  Key Features:
- Automates snapshot copy, re-encryption, restore, and cleanup
- Zero impact on production systems
- Built on **Infrastructure as Code (IaC)** using CloudFormation and CodePipeline
- Includes a **Step Functions workflow** for orchestration
- Schedule-based refreshes for continuous sync of environments

🔗 [Aurora and RDS Automated Database Refresh – CirrusHQ on AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-kpjltdtykfsj2)

---

##  Additional Resources

###  Guides:
- **Steps of a database refresh:**  
  [Orchestrating database refreshes for Amazon RDS and Aurora](https://aws.amazon.com/blogs/database/orchestrating-database-refreshes-for-amazon-rds-and-amazon-aurora/)

- **Code templates for automation:**  
  [Configuring your database refresh](https://aws.amazon.com/blogs/database/orchestrating-database-refreshes-for-amazon-rds-and-amazon-aurora/)

###  GitHub Repository:
AWS provides a reference implementation including:
- CloudFormation templates
- Lambda function code
- Sample SQL scripts

🔗 [Database Refresh Orchestrator for RDS & Aurora (GitHub)](https://github.com/aws-samples/database-refresh-orchestrator-for-amazon-rds-and-amazon-aurora)

---
