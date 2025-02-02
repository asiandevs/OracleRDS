# Creating a Custom Parameter Group and Option Group for AWS Oracle RDS

## Prerequisites
- AWS account with necessary permissions to manage RDS
- An existing RDS Oracle instance or the ability to create one
- AWS CLI installed (optional but recommended for automation)

---

## Step 1: Create a Custom Parameter Group
A parameter group allows you to configure database settings for your Oracle RDS instance.

### Using AWS Console
1. Navigate to the **Amazon RDS** console.
2. Click **Parameter groups** in the left-hand menu.
3. Click **Create parameter group**.
   ![image](https://github.com/user-attachments/assets/86126221-1c4f-4eca-846a-47ed81856063)
4. Enter a **Name** and **Description**.
5. Select **oracle-ee, oracle-se2, oracle-se1, or oracle-se** as the Engine type.
6. Select **oracle-ee-cdb-19, oracle-ee-cdb-21** as the parameter group family.
7. Click **Create**.
9. Select the newly created parameter group and click **Edit parameters**.
10. Modify the necessary parameters and click **Save changes**.

### Using AWS CLI
```sh
aws rds create-db-parameter-group \
    --db-parameter-group-name my-custom-parameter-group \
    --db-parameter-group-family oracle-ee-19 \
    --description "Custom parameter group for Oracle RDS"
```
To modify a parameter:
```sh
aws rds modify-db-parameter-group \
    --db-parameter-group-name my-custom-parameter-group \
    --parameters "ParameterName=max_connections,ParameterValue=200,ApplyMethod=immediate"
```


## Step 2: Create a Custom Option Group
An option group allows you to enable additional features like Oracle TDE or OEM.

### Using AWS Console
1. Navigate to the **Amazon RDS** console.
2. Click **Option groups** in the left-hand menu.
3. Click **Create group**.
4. Enter a **Name** and **Description**.
5. Select **Oracle** as the engine and choose the correct version.
6. Click **Create**.
7. Select the newly created option group and click **Add option**.
8. Select an option (e.g., Oracle TDE) and configure its settings.
9. Click **Add option**.

### Using AWS CLI
```sh
aws rds create-option-group \
    --option-group-name my-custom-option-group \
    --engine-name oracle-ee \
    --major-engine-version 19 \
    --option-group-description "Custom option group for Oracle RDS"
```
To add an option:
```sh
aws rds add-option-to-option-group \
    --option-group-name my-custom-option-group \
    --options "OptionName=TDE" \
    --apply-immediately
```

## Step 3: Associate Groups with RDS Instance
### Using AWS Console
1. Navigate to the **Amazon RDS** console.
2. Select the Oracle RDS instance.
3. Click **Modify**.
4. Under **Database options**, select the custom **Parameter Group**.
5. Under **Option group**, select the custom **Option Group**.
6. Click **Continue** and **Apply changes**.

### Using AWS CLI
```sh
aws rds modify-db-instance \
    --db-instance-identifier my-oracle-instance \
    --db-parameter-group-name my-custom-parameter-group \
    --option-group-name my-custom-option-group \
    --apply-immediately
```

## Step 4: Reboot RDS Instance (If Required)
Some parameter changes require a reboot to take effect.
```sh
aws rds reboot-db-instance --db-instance-identifier my-oracle-instance
```

## Conclusion
You have now successfully created and associated a custom parameter group and option group with your Oracle RDS instance. This allows fine-tuned configuration and additional features as per your requirements.

