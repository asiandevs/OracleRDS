Here we will do below tasks in AWS :

1. **Create an Oracle RDS 21c Multitenant Database (CDB)**
2. **Add a Tenant Database (PDB)**
3. **Drop the Second Tenant Database (PDB)**

---

### **Step 1: Create Oracle RDS 21c Multitenant Database**
AWS RDS for Oracle supports multitenant architecture in **Enterprise Edition**. To create an Oracle RDS 21c CDB:

1. **Log in to AWS Console** → Go to **RDS** → Click **Create database**.
2. Select **Standard Create**.
3. Choose **Oracle** as the Engine and then select different **Options** based on the business requirements.![image](https://github.com/user-attachments/assets/7bcfc594-eda7-4a22-9040-2fb2a16831d4)
Note: e.g., Select the patched version**Version 21c**.To see the 21c versions, make sure to select **Oracle multitenant architecture** to Enable **Multitenant Architecture**.
4. Select the Templates either **Production** or **Dev/Test**
5. Under setting Type a name for your DB instance (CDB Name)![image](https://github.com/user-attachments/assets/7c1b14d8-7b4f-403a-a45a-2703688fcf1f)

6. For multitenant database, Pass the **Tenant database name**, **Master Username and password**![image](https://github.com/user-attachments/assets/3712d94c-9f3a-4eaf-9f25-8b3a691f33cd)

7. Configure other settings like instance type, storage, and networking.
9. Click **Create Database**.

10. When done
     ![image](https://github.com/user-attachments/assets/d45ebbfe-ad5a-43a4-b247-70c1c15b37b6)


---

### **Step 2: Create a Pluggable Database (PDB)**
Once the RDS instance is running, connect using **SQL*Plus** or any SQL client (SQLcl, SQL Developer, etc.).

#### **1. Connect to RDS as ADMIN User**
```sql
sqlplus admin@CDB21C
```

#### **2. Create a Pluggable Database (PDB)**
```sql
BEGIN
   DBMS_PDB.CREATE_PDB (
      pdb_name    => 'PDB1',
      pdb_admin   => 'pdbadmin',
      admin_pass  => 'StrongPassword123'
   );
END;
/
```
Wait for the operation to complete, then open the new PDB:

```sql
ALTER PLUGGABLE DATABASE PDB1 OPEN;
```

To verify:

```sql
SELECT PDB_NAME, STATUS FROM DBA_PDBS;
```

---

### **Step 3: Drop the Second Tenant Database**
If you have a second tenant (e.g., `PDB2`) and want to drop it:

#### **1. Close the PDB**
```sql
ALTER PLUGGABLE DATABASE PDB2 CLOSE IMMEDIATE;
```

#### **2. Unplug the PDB**
```sql
ALTER PLUGGABLE DATABASE PDB2 UNPLUG INTO '/opt/oracle/pdb2.xml';
```

#### **3. Drop the PDB**
```sql
DROP PLUGGABLE DATABASE PDB2 INCLUDING DATAFILES;
```

To verify:

```sql
SELECT PDB_NAME FROM DBA_PDBS;
```

---

### **Summary**
- Created an **Oracle RDS 21c Multitenant** Database (CDB).
- Added a **Pluggable Database (PDB1)**.
- Dropped a **Second Pluggable Database (PDB2)**.

