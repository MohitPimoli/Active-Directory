# <span style="color:blue">Domain Controller in Active Directory (AD)</span>

## Introduction

A **Domain Controller (DC)** is a server in a **Microsoft Active Directory (AD)** environment that responds to **authentication requests** and **verifies users** on the **network**. It is a crucial component in managing network security, providing **centralized authentication**, and **enforcing policies** across devices in the domain.

## Functions of a Domain Controller

1. **Authentication**: DC authenticates and authorizes users and devices in the network. It stores credentials and verifies them during the login process.

2. **Directory Services**: DC manages the directory services, which include storing information about users, computers, and other resources in a domain.

3. **Policy Enforcement**: It applies Group Policies (GPOs) to users and computers, ensuring compliance with organizational policies.

4. **Replication**: DCs replicate directory information across other domain controllers within the domain, ensuring consistency and availability of data.

5. **Access Control**: DCs control access to resources based on user identity and permissions.

6. **LDAP Services**: DCs use the Lightweight Directory Access Protocol (LDAP) for directory queries.

## Commands for Managing Domain Controllers

### 1. **Active Directory Installation and Promotion**

- To install the Active Directory Domain Services (AD DS) role and promote a server to a domain controller:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "YourDomainName.com"
```

### 2. **Managing Domain Controllers**

- Check the Domain Controller status:

```powershell
Get-ADDomainController
```

Find all Domain Controllers in a domain:

```powershell
Get-ADDomainController -Filter *
```

Check replication status between Domain Controllers:

```powershell
repadmin /replsummary
```

Forcing replication between Domain Controllers:

```powershell
repadmin /syncall /AeD
```

- Transfer FSMO roles:

To transfer a specific FSMO role (e.g., PDC):

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "TargetDCName" -OperationMasterRole PDCEmulator
```

Transfer all FSMO roles:

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "TargetDCName" -OperationMasterRole SchemaMaster, DomainNamingMaster, PDCEmulator, RIDMaster, InfrastructureMaster
```

### 3. Managing Users and Groups

- Create a new user:

```powershell
New-ADUser -Name "John Doe" -GivenName "John" -Surname "Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@YourDomainName.com" -Path "CN=Users,DC=YourDomainName,DC=com" -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Enabled $true
```

Add a user to a group:

```powershell
Add-ADGroupMember -Identity "GroupName" -Members "jdoe"
```

Reset a user password:

```powershell
Set-ADAccountPassword -Identity "jdoe" -NewPassword (ConvertTo-SecureString "NewP@ssw0rd" -AsPlainText -Force)
```

### 4. **Group Policy Management**

Update Group Policy on a Domain Controller:

```powershell
gpupdate /force
```

List applied Group Policies:

```powershell
gpresult /r
```

Create a new Group Policy Object (GPO):

```powershell
New-GPO -Name "NewGPO" -Domain "YourDomainName.com"
```

Link a GPO to an Organizational Unit (OU):

```powershell
New-GPLink -Name "NewGPO" -Target "OU=YourOU,DC=YourDomainName,DC=com"
```

### 5. **Backup and Restore Domain Controllers**

Backup the System State (includes AD database):

```powershell
wbadmin start systemstatebackup -backuptarget:D:
```

Restore Active Directory from backup:

- Boot the server into Directory Services Restore Mode (DSRM).

Use the following command:

```powershell
wbadmin start systemstaterecovery -version:<VersionIdentifier> -recoveryTarget:C:
```

## Working of Domain Controller

- **Authentication Process**: When a user logs in, their credentials are sent to the Domain Controller, which checks these credentials against the AD database. If valid, the DC grants access, otherwise, it denies it.

- **Replication Process**: DCs replicate AD changes to each other based on a multi-master model. This ensures that changes made on one DC (e.g., a new user creation) are propagated to other DCs.

- **FSMO Roles**: Flexible Single Master Operation (FSMO) roles are special roles assigned to one or more DCs to prevent conflicts during certain operations. There are five FSMO roles:

1. Schema Master
2. Domain Naming Master
3. PDC Emulator
4. RID Master
5. Infrastructure Master

## Best Practices

1.  **Redundancy**: Always have more than one DC to ensure availability.
2.  **Regular Backups**: Regularly back up your AD database and System State.
3.  **Monitoring**: Monitor replication and DC health regularly.
4.  **Securing DSRM**: Secure the Directory Services Restore Mode (DSRM) password.

## Conclusion

The Domain Controller is a cornerstone of any Active Directory environment, handling authentication, directory services, and policy enforcement. Proper management, monitoring, and redundancy planning are key to maintaining a healthy and secure AD infrastructure.
