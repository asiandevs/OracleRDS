Amazon S3 role with the RDS instance
1.	On the Amazon RDS console, choose Option groups.
2.	Choose the group attached to the RDS instance.
3.	Choose Add option.
4.	For Option, choose S3_INTEGRATION.
5.	For Version, choose 1.0.
6.	For Apply Immediately, select Yes.
7.	Choose Add Option.
After you add S3_Integration to the option group, create an IAM role to integrate with the Oracle RDS instance.
8.	In the navigation pane of the IAM console, choose Roles, then choose Create role.
9.	Under Select trusted entity, choose AWS service and choose RDS  [RDS - Add Role to Database]..
10.	Under Add permissions, choose AmazonS3FullAccess.
11.	Under Role Details, enter RDS_S3_Integration_Role as the role name and choose Create role.
After the IAM role and S3_Integration is created, associate them with your RDS DB instance.
12.	On the Amazon RDS console, choose your DB instance.
13.	On the Connectivity & Security tab, choose Manage IAM roles.
14.	For Add IAM role to this instance, choose RDS_S3_Integration_Role (the role that you created).
15.	For Features, choose S3_INTEGRATION.
16.	Choose Add role.
17.	Validate -  On the Amazon RDS console, choose your DB instance. On the Connectivity & Security tab,
