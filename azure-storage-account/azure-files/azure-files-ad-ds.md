# Azure Files - Active Directory Domain Services (AD DS)

## Overview

Azure Files provides fully managed SMB file shares in Microsoft Azure.

In hybrid environments, Azure Files can use an existing **Active Directory Domain Services (AD DS)** environment for identity-based authentication.

Users can then access the Azure File Share using their existing Active Directory identities instead of Storage Account Keys or SAS tokens.

This setup combines:

- Azure Files
- Active Directory Domain Services
- Kerberos authentication
- Azure RBAC
- NTFS permissions

---

## Architecture

```text
Active Directory Domain Services
        │
        │ Kerberos
        ▼
Domain-joined Windows Client
        │
        │ SMB
        ▼
Azure Storage Account
        │
        ▼
Azure File Share
```

The Domain Controller authenticates the user and provides the Kerberos ticket used to access the Azure File Share.

---

## Permission Model

Azure Files uses two authorization layers:

```text
User
 │
 ▼
Authentication (Kerberos)
 │
 ▼
Share-Level Permissions (Azure RBAC)
 │
 ▼
NTFS Permissions
 │
 ▼
Effective Access
```

Both permission layers must allow access.

| Layer | Purpose |
|---|---|
| Azure RBAC | Controls access to the Azure File Share |
| NTFS ACLs | Controls access to individual files and folders |

---

## Prerequisites

### Azure

- Azure Subscription
- Storage Account
- SMB Azure File Share
- Appropriate Azure RBAC permissions
- Network connectivity to Azure Storage

### Active Directory

- Active Directory Domain Services
- Domain Controller
- Domain-joined Windows client
- Appropriate Active Directory permissions
- TCP port **445** reachable from the client

---

## Configuration

### 1. Create the Storage Account and File Share

Create a Storage Account and an SMB Azure File Share.

Example structure:

```text
Storage Account
└── File Share
    └── documents
```

At this point, the File Share exists but AD DS authentication has not yet been configured.

---

### 2. Configure AD DS Authentication

Navigate in the Azure Portal to:

```text
Storage Account
↓
File shares
↓
Identity-based access
↓
Active Directory Domain Services
```

Azure Files cannot independently create the required objects inside the on-premises Active Directory.

Microsoft therefore provides the **AzFilesHybrid** PowerShell module.

The module handles important parts of the integration, including:

- creating the Active Directory object
- configuring Kerberos
- registering the required SPNs
- configuring the Storage Account for AD DS authentication

Import the module:

```powershell
Import-Module AzFilesHybrid
```

Authenticate to Azure:

```powershell
Connect-AzAccount
```

Select the required subscription if necessary:

```powershell
Set-AzContext -Subscription "<Subscription Name>"
```

Join the Storage Account to Active Directory:

```powershell
Join-AzStorageAccount `
    -ResourceGroupName "rg-storage" `
    -StorageAccountName "stfilesdemo001" `
    -SamAccountName "AZUREFILES01" `
    -DomainAccountType ComputerAccount `
    -OrganizationalUnitDistinguishedName "OU=Servers,DC=contoso,DC=local"
```

The Storage Account is represented by an object inside Active Directory and can participate in Kerberos authentication.

---

### 3. Validate AD DS Integration

Verify that the corresponding object exists in **Active Directory Users and Computers**.

Example:

```text
OU=Servers
│
├── DC01
├── FILE01
└── AZUREFILES01
```

The configuration can also be validated using:

```powershell
Debug-AzStorageAccountAuth `
    -StorageAccountName "stfilesdemo001" `
    -ResourceGroupName "rg-storage"
```

This checks important components such as:

- Active Directory object
- Kerberos configuration
- SPNs
- Storage Account configuration

---

### 4. Configure Share-Level Permissions

Authentication alone does not grant access to the File Share.

Azure Files first evaluates the **Share-Level Permission** using Azure RBAC.

Common Azure Files roles include:

| Role | Access |
|---|---|
| Storage File Data SMB Share Reader | Read |
| Storage File Data SMB Share Contributor | Read / Write / Delete |
| Storage File Data SMB Share Elevated Contributor | Read / Write / Delete / Modify ACLs |

Whenever possible, permissions should be assigned to groups rather than individual users.

Example:

```text
Azure File Share
        │
        ▼
Azure RBAC
        │
        ▼
AD Group
```

---

### 5. Configure NTFS Permissions

After Share-Level Permissions have been granted, NTFS permissions control access to individual files and folders.

They can be configured using standard Windows tools:

```text
Folder
↓
Properties
↓
Security
↓
Edit
```

Typical permissions include:

| Permission | Typical Usage |
|---|---|
| Read | Read-only access |
| Modify | Read, create, edit and delete |
| Full Control | Administrative access |

NTFS permissions can also be managed with `icacls`.

Example:

```powershell
icacls "Z:\Projects" /grant "CONTOSO\Developers:(OI)(CI)(M)"
```

A user requires both the appropriate **Azure RBAC permission and NTFS permission**.

---

### 6. Mount the Azure File Share

The Azure File Share is accessed through its UNC path:

```text
\\<storage-account>.file.core.windows.net\<share-name>
```

Example:

```text
\\stfilesdemo001.file.core.windows.net\documents
```

The share can be mapped using Windows Explorer or the command line.

Example:

```cmd
net use Z: \\stfilesdemo001.file.core.windows.net\documents
```

For a persistent mapping:

```cmd
net use Z: \\stfilesdemo001.file.core.windows.net\documents /persistent:yes
```

Because AD DS authentication is configured, Windows can use the current Active Directory identity instead of a Storage Account Key.

---

## Validation

After the configuration, verify the complete authentication and authorization chain.

### Network Connectivity

```powershell
Test-NetConnection stfilesdemo001.file.core.windows.net -Port 445
```

Expected:

```text
TcpTestSucceeded : True
```

### DNS Resolution

```powershell
Resolve-DnsName stfilesdemo001.file.core.windows.net
```

### Kerberos Authentication

Display the current Kerberos tickets:

```cmd
klist
```

A ticket for the Azure File Share service should be present.

### SMB Connection

```powershell
Get-SmbConnection
```

### Functional Test

Verify that the user can:

- open the File Share
- create files
- modify files
- delete files
- access only permitted folders

---

## Troubleshooting

A useful troubleshooting order is:

```text
Network
   ↓
DNS
   ↓
AD DS Integration
   ↓
Kerberos
   ↓
Azure RBAC
   ↓
NTFS Permissions
   ↓
SMB Client
```

### Port 445

Azure Files SMB requires TCP port **445**.

```powershell
Test-NetConnection <storage-account>.file.core.windows.net -Port 445
```

If this fails, check:

- Firewall rules
- VPN connectivity
- Private Endpoint configuration
- Network routing

### DNS

```powershell
Resolve-DnsName <storage-account>.file.core.windows.net
```

When using a Private Endpoint, verify that the Storage Account resolves to the expected private IP address.

### Kerberos

```cmd
klist
```

If the expected ticket is missing, verify:

- Active Directory integration
- Storage Account AD object
- SPNs
- Kerberos configuration

### Existing SMB Sessions

Check current connections:

```powershell
Get-SmbConnection
```

Remove stale connections if necessary:

```cmd
net use * /delete
```

Then reconnect the File Share.

### Permissions

If authentication succeeds but access is denied, check both:

```text
Azure RBAC
+
NTFS ACL
=
Effective Access
```

Missing permission at either layer can result in **Access Denied**.

---

## Useful Commands

```powershell
# Azure authentication
Connect-AzAccount

# Current Azure context
Get-AzContext

# Validate Azure Files AD authentication
Debug-AzStorageAccountAuth `
    -ResourceGroupName "rg-storage" `
    -StorageAccountName "stfilesdemo001"

# Test SMB connectivity
Test-NetConnection `
    <storage-account>.file.core.windows.net `
    -Port 445

# DNS resolution
Resolve-DnsName <storage-account>.file.core.windows.net

# SMB connections
Get-SmbConnection
```

```cmd
:: Kerberos tickets
klist

:: Current user
whoami

:: AD group memberships
whoami /groups

:: Remove existing SMB connections
net use * /delete
```

---

## Key Takeaways

- Azure Files can authenticate users against an existing **Active Directory Domain Services (AD DS)** environment.
- Authentication is based on **Kerberos**.
- The Storage Account is represented by an object in Active Directory.
- **AzFilesHybrid** can be used to configure the AD DS integration.
- Authorization consists of two layers:
  - **Azure RBAC** for Share-Level Permissions
  - **NTFS ACLs** for files and folders
- Both permission layers must allow access.
- SMB access requires connectivity over **TCP port 445**.
- `klist`, `Test-NetConnection`, `Resolve-DnsName` and `Get-SmbConnection` are useful troubleshooting tools.
