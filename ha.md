
## Convert to Multi-AZ DB instance deployment

Select the database 
click on Actions
Then click "Convert to Multi-AZ deployment"

There are two options to Schedule database modification
1) Apply during the next scheduled maintenance window
Current maintenance window:
2) Apply immediately
The modifications in this request and any pending modifications will be asynchronously applied as soon as possible, regardless of the maintenance window setting for this database instance.

select the option and click on Convert to Multi-AZ

# Validate 

Select the database
click on Configuration section and validate Multi-AZ [ yes ] on Instance class section

## Create a standby database
Select the database instance and click on Modify
Availability & durability
Multi-AZ deployment
Two options
i) Create a standby instance (recommended for production usage)
Creates a standby in a different Availability Zone (AZ) to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups.
ii) Do not create a standby instance

If you select "Do not create a standby instance" - it will convert database to single AZ.
