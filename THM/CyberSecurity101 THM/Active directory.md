2025-05-26 18:09

Status:

Tags:  [[tryhackme]]
###### Prerequisites:



# Active Directory Domain Services (AD DS)

AD DS is the core of a Windows Domain. It stores and manages information about objects such as users, machines, groups, and more.

---

## Objects in AD

### Users

- Security principals (can be authenticated and granted permissions)
    
- Represent:
    
    - People (e.g. employees)
        
    - Services (e.g. IIS, MSSQL accounts)
        

### Machines

- Created when a computer joins the domain
    
- Security principals with limited rights
    
- Named as `ComputerName$`
    
- Passwords are auto-rotated and complex
    

### Security Groups

- Used to assign access rights to multiple users/machines
    
- Can contain users, machines, and other groups
    

**Default Security Groups**:

|Group|Description|
|---|---|
|Domain Admins|Full domain admin rights|
|Server Operators|Administer DCs, limited scope|
|Backup Operators|Access all files for backup|
|Account Operators|Create/modify domain accounts|
|Domain Users|All domain user accounts|
|Domain Computers|All domain computer accounts|
|Domain Controllers|All DCs in the domain|

---

## Active Directory Users and Computers (ADUC)

- Tool to manage AD objects (users, groups, machines)
    
- Access via Start Menu → "Active Directory Users and Computers"
    
- Objects organized into Organizational Units (OUs)
    

### Organizational Units (OUs)

- Used to apply policies
    
- One OU per user/machine
    
- Often mirrors org structure (e.g. IT, Sales)
    
- Custom OUs can be created
    

### Default Containers

|Container|Purpose|
|---|---|
|Builtin|Default groups|
|Computers|Default location for new machines|
|Domain Controllers|Contains DCs|
|Users|Default domain-wide users/groups|
|Managed Service Accounts|Service-related accounts|

---

## Security Groups vs OUs

| Feature    | Security Groups     | Organizational Units |
| ---------- | ------------------- | -------------------- |
| Purpose    | Assign permissions  | Apply policies       |
| Membership | Multiple per user   | Only one per user    |
| Use Case   | File/printer access | Policy enforcement   |


----
# AD Task: Aligning OUs and Users with Org Chart

## Objective

Update Active Directory to match the new THM organizational chart:

- Remove outdated OUs
    
- Adjust user accounts
    
- Delegate OU control
    

---

## 1. Remove Extra OUs

### Issue

- An obsolete OU exists in AD but is not part of the org chart.
    
- Attempting to delete it throws a protection error.
    

### Solution

1. In ADUC, go to `View` → Enable **Advanced Features**
    
2. Right-click the OU → `Properties` → `Object` tab
    
3. Uncheck **"Protect object from accidental deletion"**
    
4. Delete the OU (including any sub-objects)
    

---

## 2. Sync Users with Org Chart

- Compare each OU's user list with the organizational chart.
    
- **Create or delete users** as necessary to match the chart.
    

---

## 3. Delegation of OU Control

### Goal

Allow Phillip (IT support) to reset passwords for:

- Sales
    
- Marketing
    
- Management OUs
    

### Example: Delegate Sales OU

1. Right-click `Sales` OU → `Delegate Control`
    
2. Add user: `phillip` → Click **Check Names**
    
3. Choose **"Reset user passwords and force password change at next logon"**
    
4. Finish wizard
    

> Repeat for Marketing and Management if desired.

---

## 4. Test Delegation (Using Phillip)

### Credentials

- Username: `THM\phillip`
    
- Password: `Claire2008`
    

### Reset Sophie's Password via PowerShell

powershell

CopyEdit

`Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose`

### Force Password Change at Next Logon

powershell

CopyEdit

`Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose`

---

## 5. Retrieve Flag (as Sophie)

- Log in via RDP:
    
    - Username: `THM\sophie`
        
    - Password: _(new one set by Phillip)_
        
- Check Sophie's desktop for the flag file.
    

---