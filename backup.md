BACKUP

AWS Backup centralizes and automates data protection across AWS services. AWS Backup is a fully managed, policy-based service for data protection at scale. The service is ideal for use cases such as regulatory compliance obligations, business policies for data protection, and business continuity goals.

The following diagram shows using an AWS Backup plan and backup vault to take snapshots of the Amazon RDS instance at scheduled intervals

![image](https://github.com/user-attachments/assets/d84aaed2-2673-465a-9f30-a4c3bae0fa90)

Create a key

The AWS KMS key is used to encrypt your backup data, ensuring its security and privacy.
Benefits of using a KMS key:

Encryption of your backup data at rest

Centralized key management and control

Compliance with data security regulations

To create a KMS key:

Go to the AWS Management Console and navigate to the KMS service.
Click on "Create key" and follow the prompts to create a new KMS key.


Choose the appropriate key type and configure the key settings according to your requirements.



Click Next.

Add labels, tags and click Next.



Define Key administrative permissions and click Next.

For testing purpose I am using user with AWSBACKUPFull policy as I don't have access to create a user with policy in place.





Click Next



Create backup vault

The backup vault is a secure storage location for your backup data.
Benefits of using a backup vault:

Centralized management and organization of your backup data

Compliance with data retention policies

Ability to set access controls and permissions

To create a backup vault:

Go to the AWS Backup service in the AWS Management Console.
Click on "Vaults"--> "Create new vault" 

Choose the appropriate Vault name, Vault type, Encryption key (created earlier) add vault tags and click “Create vault”.



Configure a backup plan

A Backup plan specifies the backup schedule, backup retention rules, frequency, lifecycle rules and other settings for your backups processes.



To configure your backup plan, complete the following steps:

On the AWS Backup console, choose Backup plans in the navigation pane.

Choose Create backup plan.















Click “Create plan”











Create  a backup rule









 

Validate Backup

On-demand backup

Connect to AWS account, select region 

Search for AWS Backup and go to dashboard and then click on "Create on-demand backup.



Select the Resource type as 'RDS' and select the required 'Database name'.

Select Backup window, either 'Create backup now' or 'Customize backup window'

For Total retention period, for testing purpose, I’m selecting a retention period as 1 day , you can choose it on your choice. 

For Backup vault, select the respective vault

For IAM role, I select default or the respective role.

Add tags to easy to define this backup

and click on 'Create on-demand backup'.


Validate Job:

Notification setup:
```
aws backup put-backup-vault-notifications --backup-vault-name Oracle-NonProduction-BackupVault --sns-topic-arn arn:aws:sns:ap-southeast-2:977099011956:Database-Backup-nonproduction-Status --backup-vault-events BACKUP_JOB_COMPLETED RESTORE_JOB_COMPLETED

```
```
~ $ aws backup get-backup-vault-notifications --backup-vault-name Oracle-Production-Backup-Vault{
    "BackupVaultName": "Oracle-Production-Backup-Vault",
    "BackupVaultArn": "arn:aws:backup:ap-southeast-2:982081058810:backup-vault:Oracle-Production-Backup-Vault",
    "SNSTopicArn": "arn:aws:sns:ap-southeast-2:982081058810:Database-Backup-production-Status",
    "BackupVaultEvents": [
        "BACKUP_JOB_COMPLETED",
        "RESTORE_JOB_COMPLETED"
    ]
}
(END)
```
