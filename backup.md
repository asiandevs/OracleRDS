# 🔐 Automating Oracle RDS Backups with AWS Backup and KMS

Data protection and recovery are crucial for maintaining business continuity, meeting regulatory requirements, and ensuring operational resilience. **AWS Backup** offers a centralized, fully managed solution to automate and scale your data backup operations across AWS services, including Amazon RDS.

In this blog post, we'll walk through how to configure AWS Backup for Oracle RDS with encryption using **AWS Key Management Service (KMS)**. We’ll cover creating a KMS key, setting up a backup vault, configuring a backup plan, and triggering an on-demand backup—all while ensuring your backups are secure, organized, and compliant.

---

## 🛡️ Why Use AWS Backup?

**AWS Backup** is a policy-driven service that simplifies and automates backup management at scale. It’s particularly valuable for:

- Meeting **regulatory compliance** obligations  
- Enforcing **business continuity** and disaster recovery policies  
- Simplifying **centralized backup management** across AWS services  

---

## 📌 Architecture Overview

The setup involves:

1. A **KMS key** to encrypt backup data  
2. A **Backup Vault** to store backups securely  
3. A **Backup Plan** to define schedule, retention, and rules  
4. Optional **notifications** for job status

**AWS Backup for Oracle RDS - High Level Architecture** ![image](https://github.com/user-attachments/assets/d84aaed2-2673-465a-9f30-a4c3bae0fa90)

---

## 🔑 Step 1: Create a KMS Key

A **KMS key** is used to encrypt your backup data at rest, ensuring its privacy and integrity.

### ✅ Benefits:
- Secure **encryption at rest**
- Centralized key control and auditability
- Support for **compliance standards**

### 🛠️ How to Create:
1. Go to **AWS Management Console → Key Management Service (KMS)**
2. Click **Create key**
3. Select the key type and configure settings
4. Add labels/tags, set key permissions
5. Finalize and create the key

> _Note: For demo purposes, we used a user with the `AWSBackupFullAccess` policy due to restricted IAM permissions._

---

## 💾 Step 2: Create a Backup Vault

A **Backup Vault** is a logical container that securely stores backups and allows tagging, access control, and encryption settings.

### ✅ Benefits:
- Centralized storage for backup data
- Fine-grained access control
- Compliance with retention policies

### 🛠️ How to Create:
1. Go to **AWS Backup → Vaults → Create Vault**
2. Specify:
   - **Vault name**
   - **Encryption key** (use the KMS key created earlier)
   - **Tags** (optional)
3. Click **Create vault**

---

## 📆 Step 3: Configure a Backup Plan

A **Backup Plan** defines:
- Backup frequency (daily, weekly, etc.)
- Retention period
- Lifecycle settings (cold storage transition)
- Target resources (RDS, EC2, etc.)

### 🛠️ To create a backup plan:
1. Go to **AWS Backup → Backup Plans**
2. Click **Create Backup Plan**
3. Use **Build a new plan** option
4. Define backup rules:
   - **Schedule**
   - **Retention**
   - **Vault**
5. Save and assign resources (your RDS database)

---

## ⚡ Step 4: Create an On-Demand Backup

You can create ad-hoc backups using the **on-demand** feature.

### 🛠️ How to Do It:
1. Open **AWS Backup Console**
2. Go to **Dashboard → Create on-demand backup**
3. Select:
   - **Resource type**: `RDS`
   - **Database name**
   - **Backup window**: Now or custom
   - **Retention**: e.g., 1 day for testing
   - **Backup vault**: Choose from step 2
   - **IAM Role**: Default or custom
4. Add tags
5. Click **Create on-demand backup**

---

## ✅ Step 5: Validate Backup Job

Once initiated, monitor the backup status under the **Jobs** section in the AWS Backup console.

---

## 📣 Optional: Set Up SNS Notifications

Receive notifications when backup and restore jobs complete using **Amazon SNS**.

### 🛠️ Configure Notifications via CLI:

```bash
aws backup put-backup-vault-notifications \
  --backup-vault-name Oracle-NonProduction-BackupVault \
  --sns-topic-arn arn:aws:sns:ap-southeast-2:977099011956:Database-Backup-nonproduction-Status \
  --backup-vault-events BACKUP_JOB_COMPLETED RESTORE_JOB_COMPLETED
```

### 🔍 Verify Configuration:

```bash
aws backup get-backup-vault-notifications \
  --backup-vault-name Oracle-Production-Backup-Vault
```

**Sample Output:**

```json
{
  "BackupVaultName": "Oracle-Production-Backup-Vault",
  "BackupVaultArn": "arn:aws:backup:ap-southeast-2:376449999999:backup-vault:Oracle-Production-Backup-Vault",
  "SNSTopicArn": "arn:aws:sns:ap-southeast-2:376449999999:Database-Backup-production-Status",
  "BackupVaultEvents": [
    "BACKUP_JOB_COMPLETED",
    "RESTORE_JOB_COMPLETED"
  ]
}
```

---

## 🎯 Final Thoughts

**AWS Backup** offers a powerful, secure, and automated way to protect your **Amazon RDS for Oracle** databases. By combining backup policies, vaults, and KMS encryption, you can meet compliance needs while minimizing manual effort.

Whether you’re looking for regulatory compliance, centralized backup control, or enhanced disaster recovery readiness—**this solution has you covered**.

---

### 📚 Useful Resources:

- 🔗 [AWS Backup Documentation](https://docs.aws.amazon.com/backup/)
- 🔗 [AWS KMS Documentation](https://docs.aws.amazon.com/kms/latest/developerguide/)
- 🔗 [AWS Backup Vault Notifications](https://docs.aws.amazon.com/cli/latest/reference/backup/put-backup-vault-notifications.html)

---
