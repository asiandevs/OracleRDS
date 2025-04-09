You reached out to us as you wished to know how to change the retention period for audit files for a longer duration than the default configuration of 7 days as it is not present in the documentation.

Over the chat we discussed on the issue. Further I requested you that I will be taking this offline and perform my investigation on the ask.

  As you may already know that the default retention period for audit files is seven days. Amazon RDS might delete audit files older than seven days. While the documentation states that Audit files and trace files share the same retention configuration.


 However unfortunately the retention period of audit logs cannot be changed on the RDS Instance and will remain at default period of 7 days. But as you mentioned that you wish to retain the audit logs for a longer period of time then in that case you leverage CloudWatch logs where you can configure your RDS for Oracle DB instance to publish log data to a log group in Amazon CloudWatch Logs. With CloudWatch Logs, you can analyze the log data, and use CloudWatch to create alarms and view metrics. You can use CloudWatch Logs to store your log records in highly durable storage.

	> Amazon RDS publishes each Oracle database log as a separate database stream in the log group. For example, if you configure the export function to include the audit log, audit data is stored in an audit log stream in the /aws/rds/instance/my_instance/audit log group.


 >> For exporting audit logs you can set the set the audit_trail parameter to any of the following allowed values and based on the parameter value the audit logs get captured and exported to CloudWatch.

	- { none | os | db [, extended] | xml [, extended] }


 You can refer the following documentation to enable exporting the logs to CloudWatch,
 	[+] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.Oracle.html#USER_LogAccess.Oracle.PublishtoCloudWatchLogs 


Please note that by default CloudWatch logs are stored indefinitely, however you can modify the retention period of this log group by referring the following documentation,
 	[+] Change log data retention in CloudWatch Logs - https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html#SttingLogRetention 


Archivelog log retention specifies the duration in hours before archive/redo log files are automatically deleted. As for the archive log retention hours, I would like to inform you that the default value would be 0 and this indicates that the archive logs are purged after their creation, however this will not have any impact on your Point In time restores as when the archived log retention period expires, RDS for Oracle removes the archived redo logs from your DB instance. To support restoring your DB instance to a point in time, Amazon RDS retains the archived redo logs outside of your DB instance based on the backup retention period.  


You can read more on archive log retention from the following documentation, 
	[+] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RetainRedoLogs.html 
