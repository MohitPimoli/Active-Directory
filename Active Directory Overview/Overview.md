# <span style="color:blue">Active Directory Overview</span>

## What is Active Directory?

Active Directory (AD) is a directory service developed by Microsoft for Windows domain networks. It is included in most Windows Server operating systems as a set of processes and services.

## Key Components

- **Domain**: A logical group of network objects (computers, users, devices) that share the same Active Directory database.
- **Domain Controller (DC)**: A server that responds to security authentication requests (logging in, checking permissions, etc.) within a Windows domain.
- **Organizational Units (OUs)**: Containers within a domain that can hold users, groups, computers, and other OUs. They help organize and manage a large number of objects.
- **Forest**: The top-level container in an Active Directory configuration that contains one or more domains.
- **Tree**: A collection of one or more domains and domain trees in a contiguous namespace, linked in a transitive trust hierarchy.

## Domain Controller (DC)

- **Primary Functions**:
  - **Authentication**: Verifies user credentials and grants access to resources.
  - **Replication**: Ensures that changes made in one DC are replicated to all other DCs in the domain.
  - **FSMO Roles**: Special roles assigned to one or more DCs to handle specific tasks (e.g., Schema Master, Domain Naming Master).

## Setting Up a Domain Controller

1. **Install Active Directory Domain Services (AD DS)**:

```powershell
   Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

2. **Promote the Server to a Domain Controller:**

```powershell
Install-ADDSForest -DomainName "example.com"
```

# Common Commands

## Check Domain Controllers

```powershell
Get-ADDomainController -Filter *
```

## Add a New User:

```powershell
New-ADUser -Name "John Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@example.com" -Path "OU=Users,DC=example,DC=com" -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Enabled $true
```

# Troubleshooting

- **DNS Issues**: Ensure the DNS server is correctly configured and reachable.

- **Replication Problems**: Use the repadmin tool to diagnose and fix replication issues.

```powershell
repadmin /replsummary
```

# Resources

## [Microsoft Learn: Active Directory Domain Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
