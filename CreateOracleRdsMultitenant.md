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

#### **1. Click to RDS (CDB)**
#### **2. Click "Actions --> Add tenant database**

![image](https://github.com/user-attachments/assets/b84565ed-a7e7-458a-857b-ba80ab962d86)
#### **3. Pass values for the Tenant database settings and Click "Add tenant**
![image](https://github.com/user-attachments/assets/edeb9bdf-7a97-44a0-b2e0-621a8916de73)
#### **4. It will in Creating stage and then Available stage
Wait for the operation to complete, then open the new PDB:
![image](https://github.com/user-attachments/assets/19178f31-2e9c-43f3-9f3e-717ed841df1d)

![image](https://github.com/user-attachments/assets/9ae58744-7089-43c3-ab27-1a87baa229e8)

---

### **Step 3: Drop Tenant Database**
For a multitenant architecture, one tenant database must be present.

If you have a second tenant (e.g., `MALOTI`) and want to drop it:
#### **1. Click to RDS (PDB)**
#### **2. Click "Actions --> Delete**
![image](https://github.com/user-attachments/assets/b3d710c3-d584-4926-8a8f-c84ed62875c4)

Take a backup if required [Tick --> Create final snapshot?], and then type delete me into the field and click delete
![image](https://github.com/user-attachments/assets/f664293d-ac6e-4362-a575-8118c6bb0b08)

For CDB delete, you need to select CDB and proceed on that
![image](https://github.com/user-attachments/assets/0369a17d-33e3-4239-a663-1f87d9514f16)


---

### **Summary**
- Created an **Oracle RDS 21c Multitenant** Database (CDB).
- Added a **Pluggable Database (MALOTI)**.
- Dropped a **Second Pluggable Database (MALOTI)**.

