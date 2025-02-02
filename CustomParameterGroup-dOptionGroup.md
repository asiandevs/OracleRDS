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
5. Select **oracle-ee, oracle-ee-cdb, oracle-se2, or oracle-se2-cdb** as the Engine type.
6. Select **oracle-ee-cdb-19, oracle-ee-cdb-21** as the parameter group family.
7. Click **Create**.
![image](https://github.com/user-attachments/assets/e394bef1-f34e-4cb4-a812-edd3b25d4f1b)

8. Select the newly created parameter group and click **Edit parameters**.
![image](https://github.com/user-attachments/assets/17e92eaa-7a51-446a-932d-2d6d02a25954)

9. Modify the necessary parameters and click **Save changes**.
![image](https://github.com/user-attachments/assets/f2a1ed31-bd0c-47ee-ad9e-2e02d0d71d7f)

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
![image](https://github.com/user-attachments/assets/d45631cb-b047-46e3-a6d7-39672dc8117d)

4. Enter a **Name** and **Description**.
5. Select ****oracle-ee, oracle-ee-cdb, oracle-se2, or oracle-se2-cdb** as the Engine type.** as the engine
6. Select Major Engine Version and choose the correct version **19 or 21**.
7. Click **Create**.![image](https://github.com/user-attachments/assets/54b33263-b2d6-4b64-afc5-05f6cc880a39)
8. Select the newly created option group and click **Add option**.![image](https://github.com/user-attachments/assets/0eb59d25-9660-4973-a522-d7c2cb5a17b7)
9. Select an option (e.g., UTL_MAIL) and configure its settings.
10. Click **Add option**.![image](https://github.com/user-attachments/assets/e3c1081d-7178-4219-94ef-85f3213a2bd7)


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
Either you can add them during database creation
![image](https://github.com/user-attachments/assets/50d2ea52-1452-447e-b704-c08731028c78)

or later you can associated after database creation
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

## Remove or Edit (If Required)
1. Navigate to the custom **Parameter Group** or the custom **Option Group**.
2. For **Parameter Group**

![image](https://github.com/user-attachments/assets/372e647c-e488-4156-96a2-01da536fc364

Or For **Option Group**.
![image](https://github.com/user-attachments/assets/c1dd4a24-a24e-46f7-9bde-2930060e8012)


## Conclusion
You have now successfully created and associated a custom parameter group and option group with your Oracle RDS instance. This allows fine-tuned configuration and additional features as per your requirements.

