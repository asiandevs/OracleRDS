Here is  the high level information on how to automate the Oracle RDS refreshes from one account to another .

Automating the refresh of an Oracle RDS database from one AWS account to another involves several steps and the use of AWS services like AWS Backup, AWS Lambda, and AWS Step Functions. 

Option1:

High level steps: 

1. Set Up AWS Backup:
    * Enable automated backups for your Oracle RDS instance in the source account.
    * Configure cross-account backup policies to copy snapshots to the destination account.
2. Create AWS Lambda Functions:
    * Write Lambda functions to automate the snapshot creation, copying, and restoration processes.
    * Use AWS SDKs (e.g., Boto3 for Python) to interact with RDS and AWS Backup APIs.
3. Set Up AWS Step Functions:
    * Create a state machine in AWS Step Functions to orchestrate the workflow.
    * Define states for creating snapshots, copying snapshots, and restoring them in the destination account.
4. Configure EventBridge:
    * Use Amazon EventBridge to trigger the Lambda functions based on scheduled events or specific conditions.
    * Set up rules to handle the lifecycle of snapshots and database refreshes.
5. Implement Security Measures:
    * Use AWS Key Management Service (KMS) to encrypt snapshots and ensure secure data transfer.
    * Share KMS keys between accounts to enable cross-account access to encrypted snapshots.

Please go through the solution outlined [1] and you can automate this using the solution outlined in [2] using Amazon EventBridge and AWS Lambda. 

[1] https://aws.amazon.com/blogs/database/automate-cross-account-backup-of-amazon-rds-for-oracle-including-database-parameter-groups-option-groups-and-security-groups/ 
[2] https://aws.amazon.com/blogs/database/automate-cross-account-backups-of-amazon-rds-and-amazon-aurora-databases-with-aws-backup/ 

Option2:

AWS Marketplace tool: Aurora and RDS Automated Database Refresh from CirrusHQ
[+]https://aws.amazon.com/marketplace/pp/prodview-kpjltdtykfsj2 
Description: CircuHq  Automated Database Refresh for Aurora and RDS automatically migrates Aurora and RDS database snapshots from a production AWS Account to a non-production AWS Account to securely refresh the non-production environment with production-level data. It provides all infrastructure components by utilizing Infrastructure as Code (IaC) and a CI/CD CodePipeline to deploy an AWS Step Function State Machine to orchestrate database snapshot copy, re-encryption, restore and cleanup of the non-production database with zero impact to the production database. 
Optionally executed on a schedule to periodically refresh non-production databases.


Useful documents to refer:

See "Steps of a database refresh" section of this documentation:
[+]https://aws.amazon.com/blogs/database/orchestrating-database-refreshes-for-amazon-rds-and-amazon-aurora/ 

"Configuring your database refresh" section of documentation below, provides you with code templates that you can use to automate the process.
[+]https://aws.amazon.com/blogs/database/orchestrating-database-refreshes-for-amazon-rds-and-amazon-aurora/ 

This GitHub repo has package awssoldb-orchestrator-pkg-cloudformation.zip that represents the solution (CloudFormation templates, Lambda function's code and sample sql-scripts). I would highly suggest reading through this.
[+]https://github.com/aws-samples/database-refresh-orchestrator-for-amazon-rds-and-amazon-aurora 

I hope the above information was helpful and helps you troubleshoot your queries. In case you have any further queries or seek clarifications please feel free to update the case or we can have a call for further discussion if you would like.
Please note that I work in Australian Eastern Timezone, my shift hours are 08:00 AM to 04:00 PM AEDT and I can setup a chime meeting based on your availability anytime within this window.

We value your feedback. Please share your experience by rating this and other correspondences in the AWS Support Center. You can rate a correspondence by selecting the stars in the top right corner of the correspondence.
