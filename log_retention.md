#  How to Extend Audit Log Retention for Amazon RDS for Oracle

If you're managing Oracle databases on Amazon RDS, you've probably come across a common challenge: **audit logs are only retained for 7 days by default**—and this retention period cannot be changed directly on the RDS instance.

Recently, a customer reached out to us to understand whether it was possible to retain audit logs longer than the default configuration. In this blog post, we'll walk through that question, what we discovered, and how to work around the limitation using **Amazon CloudWatch Logs**.

---

##  Understanding the Default Audit Log Retention

By default, Amazon RDS retains **audit files** and **trace files** for **7 days**. After this period, the platform may automatically delete older files. Although this is sufficient for some scenarios, many organizations—especially those with compliance or security requirements—need longer retention windows.

>  **Important:** The audit log retention period **cannot** be extended or configured directly within RDS.

This limitation is not clearly documented, and it has implications for monitoring and forensic activities.

---

##  Workaround: Export Audit Logs to Amazon CloudWatch Logs

If you need to retain audit logs for longer than 7 days, your best option is to **export logs to CloudWatch Logs**. This AWS-native monitoring service provides **durable log storage**, advanced **search and visualization**, and supports **custom retention periods**.

###  How It Works

When configured, Amazon RDS can stream Oracle logs—including audit logs—to individual **log groups** in CloudWatch.

For example:

```
/aws/rds/instance/my_instance/audit
```

This stream will contain your audit trail logs, safely stored and available for extended retention.

---

##  Setting It Up

To enable audit log export, you need to adjust the `audit_trail` parameter. Valid values include:

```
none | os | db [, extended] | xml [, extended]
```

After setting this parameter, follow AWS documentation to publish the logs:

🔗 [Export RDS Oracle Logs to CloudWatch](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.Oracle.html#USER_LogAccess.Oracle.PublishtoCloudWatchLogs)

Once enabled, RDS begins streaming log data automatically to the appropriate CloudWatch log group.

---

##  Configuring Retention in CloudWatch Logs

By default, **CloudWatch logs are retained indefinitely**, which is great for long-term compliance. However, if needed, you can customize the retention period per log group.

Here’s how:

🔗 [Set Log Retention in CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html#SttingLogRetention)

This gives you full control over how long you store each type of log.

---

##  A Note on Archive Logs and PITR

In addition to audit logs, Oracle on RDS generates **archived redo logs**, which are crucial for **Point-in-Time Recovery (PITR)**.

- The **archive log retention** default is `0` hours.
- This means archived logs are purged soon after creation.
- **However**, PITR remains unaffected. Amazon RDS keeps the necessary logs **outside the instance**, based on your **backup retention settings**.

 [More on Archive Log Retention](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RetainRedoLogs.html)

---

##  Final Thoughts

While it's unfortunate that the audit log retention period cannot be extended directly within Amazon RDS for Oracle, the workaround using **CloudWatch Logs** is both powerful and flexible. It gives you greater control over log management, longer retention, and integrates well with the broader AWS ecosystem for monitoring and alerting.

If you're looking to ensure compliance, improve observability, or simply keep your audit trail intact for longer, enabling CloudWatch log exports should be your next step.

---

Have questions or want help implementing this setup? Drop a comment below or reach out!

---
