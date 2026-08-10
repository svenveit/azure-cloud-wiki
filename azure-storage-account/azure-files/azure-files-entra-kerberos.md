# Azure Files – Microsoft Entra Kerberos Authentication

## Overview

Azure Files supports identity-based authentication for SMB file shares using **Microsoft Entra Kerberos**.

This allows users to access an Azure File Share with their identity instead of using the Storage Account access key.

Microsoft Entra Kerberos is especially useful for environments where Azure Files is accessed from **Microsoft Entra joined or hybrid joined Windows clients**.

The authorization model consists of two layers:

1. **Share-level permissions** using Azure RBAC
2. **File and folder permissions** using NTFS ACLs

---

## Authentication Flow

A simplified authentication flow looks like this:

```text
User
  │
  ▼
Windows Client
  │
  │ Microsoft Entra authentication
  ▼
Microsoft Entra ID
  │
  │ Kerberos ticket
  ▼
Azure Files
  │
  ▼
NTFS permissions
  │
  ▼
Files / Folders
```

The user signs in with their Microsoft Entra identity.

Microsoft Entra Kerberos provides the Kerberos authentication required for SMB access to the Azure File Share.

Access is then controlled through Azure RBAC and NTFS permissions.

---

## Prerequisites

Typical requirements include:

- Azure Storage Account
- Azure File Share using SMB
- Microsoft Entra tenant
- Windows clients with the required identity configuration
- Microsoft Entra Kerberos enabled for the Storage Account
- Azure RBAC permissions for the File Share
- Appropriate NTFS permissions

> Microsoft Entra Kerberos is used for authentication. It does not replace the authorization mechanisms provided by Azure RBAC and NTFS ACLs.

---

## Enable Microsoft Entra Kerberos

Open the Storage Account in the Azure Portal.

Navigate to:

**File shares → Identity-based access**

Select **Microsoft Entra Kerberos** as the identity source and enable the configuration.

The Storage Account can then use Microsoft Entra Kerberos for identity-based SMB authentication.

---

## Share-Level Permissions

Before a user can access the File Share, the identity requires an appropriate Azure RBAC role.

Common Azure Files roles include:

| Role | Purpose |
|---|---|
| Storage File Data SMB Share Reader | Read access |
| Storage File Data SMB Share Contributor | Read and write access |
| Storage File Data SMB Share Elevated Contributor | Extended permissions including modification of NTFS ACLs |

The role can be assigned to users or groups.

Using groups is generally preferable for managing access.

Example:

```text
Microsoft Entra Group
        │
        ▼
Azure RBAC Role
        │
        ▼
Azure File Share
```

---

## NTFS Permissions

Azure RBAC determines whether a user is allowed to access the File Share.

NTFS permissions determine what the user can do with individual folders and files.

Typical permissions include:

- Read
- Write
- Modify
- Full Control

The effective access therefore depends on both permission layers:

```text
Azure RBAC
    +
NTFS ACL
    │
    ▼
Effective File Access
```

---

## Mount the File Share

After authentication and permissions have been configured, the Azure File Share can be mounted using its UNC path.

Example:

```text
\\<storage-account>.file.core.windows.net\<file-share>
```

For example, the share can be mapped from Windows Explorer using:

**This PC → Map network drive**

or with PowerShell:

```powershell
New-PSDrive `
  -Name "Z" `
  -PSProvider FileSystem `
  -Root "\\<storage-account>.file.core.windows.net\<file-share>" `
  -Persist
```

No Storage Account access key should be required when identity-based authentication is working correctly.

---

## Microsoft Entra Kerberos vs. AD DS

Azure Files supports different identity-based authentication options.

| | Microsoft Entra Kerberos | Active Directory Domain Services |
|---|---|---|
| Identity | Microsoft Entra ID | Active Directory Domain Services |
| Authentication | Kerberos | Kerberos |
| Storage Account key required for users | No | No |
| Share-level authorization | Azure RBAC | Azure RBAC |
| File/folder authorization | NTFS ACLs | NTFS ACLs |
| Typical scenario | Modern Azure / Entra environments | Traditional or hybrid AD environments |

The appropriate authentication method depends primarily on the identity architecture and the clients accessing Azure Files.

---

## Validation

After configuration, verify that:

- Microsoft Entra Kerberos is enabled
- the user or group has an Azure Files RBAC role
- NTFS permissions are configured
- the client meets the identity requirements
- the File Share can be mounted without a Storage Account key
- files and folders can be accessed according to the configured permissions

---

## Troubleshooting

### Access denied

Check both permission layers:

1. Azure RBAC assignment
2. NTFS permissions

Having an Azure RBAC role alone does not automatically provide access to every file and folder.

### File Share cannot be mounted

Verify:

- SMB connectivity
- DNS resolution
- client identity configuration
- Microsoft Entra Kerberos configuration
- Storage Account network restrictions

### Authentication works but files cannot be accessed

Authentication and authorization are separate.

Successful Kerberos authentication does not guarantee access to the files.

Check the NTFS ACLs assigned to the user or group.

---

## Key Takeaways

- Microsoft Entra Kerberos provides identity-based SMB authentication for Azure Files.
- Users do not need to authenticate with the Storage Account access key.
- Azure RBAC controls access at the File Share level.
- NTFS ACLs control access to folders and files.
- Microsoft Entra Kerberos is an alternative to traditional AD DS-based Azure Files authentication.
- The authentication method should match the identity and client architecture of the environment.
