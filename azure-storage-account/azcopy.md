# AzCopy

## Overview

**AzCopy** is a command-line utility from Microsoft for transferring data to and from **Azure Storage**.

It is optimized for high-performance data transfers and is commonly used for:

- Uploading files to Azure Storage
- Downloading files from Azure Storage
- Copying data between Azure Storage resources
- Migrating large amounts of data
- Synchronizing local directories with Azure Storage
- Automating storage transfers in scripts

AzCopy supports services including:

- Azure Blob Storage
- Azure Files

---

## Installation

AzCopy is available for:

- Windows
- Linux
- macOS

After installation, verify that AzCopy is available:

```bash
azcopy --version
```

Display available commands:

```bash
azcopy --help
```

---

## Authentication

AzCopy supports different authentication methods.

Common options include:

- Microsoft Entra ID
- Shared Access Signature (SAS)
- Managed Identity
- Storage Account access mechanisms depending on the scenario

For interactive authentication with Microsoft Entra ID:

```bash
azcopy login
```

AzCopy opens an authentication flow and uses the signed-in identity for subsequent operations.

The identity must have the required Azure RBAC permissions on the target Storage Account or container.

---

## Basic Command Structure

The general structure of an AzCopy command is:

```bash
azcopy <command> <source> <destination> [flags]
```

Common commands include:

```text
copy
sync
remove
list
login
logout
```

---

## Upload Files

Upload a single file to Azure Blob Storage:

```bash
azcopy copy "C:\Data\example.txt" "https://<storage-account>.blob.core.windows.net/<container>/<SAS-token>"
```

Upload an entire directory:

```bash
azcopy copy "C:\Data" "https://<storage-account>.blob.core.windows.net/<container>/<SAS-token>" --recursive=true
```

The `--recursive=true` option includes all subdirectories and files.

---

## Download Files

Download a single file:

```bash
azcopy copy "https://<storage-account>.blob.core.windows.net/<container>/example.txt?<SAS-token>" "C:\Data\example.txt"
```

Download an entire directory:

```bash
azcopy copy "https://<storage-account>.blob.core.windows.net/<container>?<SAS-token>" "C:\Data" --recursive=true
```

---

## Azure Files

AzCopy can also transfer data to and from Azure File Shares.

Example upload:

```bash
azcopy copy "C:\Data" "https://<storage-account>.file.core.windows.net/<file-share>/<SAS-token>" --recursive=true
```

Example download:

```bash
azcopy copy "https://<storage-account>.file.core.windows.net/<file-share>?<SAS-token>" "C:\Data" --recursive=true
```

This can be useful when migrating existing file server data into Azure Files.

---

## Copy Between Azure Storage Resources

AzCopy can perform server-to-server transfers between supported Azure Storage resources.

Example:

```bash
azcopy copy \
"https://<source-storage-account>.blob.core.windows.net/<container>?<SAS-token>" \
"https://<target-storage-account>.blob.core.windows.net/<container>?<SAS-token>" \
--recursive=true
```

The data does not need to be downloaded manually to the administrator workstation before being uploaded again.

---

## Copy vs. Sync

AzCopy provides both `copy` and `sync`.

### Copy

```bash
azcopy copy <source> <destination> --recursive=true
```

`copy` transfers the selected files from the source to the destination.

It is useful for:

- Initial migrations
- Uploads
- Downloads
- One-time data transfers

### Sync

```bash
azcopy sync <source> <destination> --recursive=true
```

`sync` compares the source and destination and transfers data required to synchronize them.

It is useful for scenarios where the destination should be kept synchronized with the source.

> Be careful when using synchronization options that delete files from the destination. Always validate the source and destination before running destructive operations.

---

## Useful Options

### Recursive Copy

Copy directories and their contents:

```bash
--recursive=true
```

### Overwrite Behavior

Control whether existing files are overwritten:

```bash
--overwrite=true
```

### Include Files

Example:

```bash
--include-pattern="*.txt"
```

This can be used to transfer only specific file types.

### Exclude Files

Example:

```bash
--exclude-pattern="*.tmp"
```

This can be useful for excluding temporary files during a migration.

---

## Jobs and Logging

AzCopy creates jobs for transfer operations.

List previous jobs:

```bash
azcopy jobs list
```

Display information about a specific job:

```bash
azcopy jobs show <job-id>
```

Failed or interrupted transfers can therefore be investigated using the job information and AzCopy logs.

This is especially useful for large migration jobs where individual transfer failures may otherwise be difficult to identify.

---

## Example Migration

A simple Azure Files migration could look like this:

```text
Local File Server
       │
       │ AzCopy
       ▼
Azure Storage Account
       │
       ▼
Azure File Share
```

Example command:

```bash
azcopy copy "D:\CompanyData" "https://<storage-account>.file.core.windows.net/<file-share>?<SAS-token>" --recursive=true
```

After the transfer, verify:

- Number of transferred files
- Failed transfers
- Directory structure
- Required file metadata and permissions
- Application access to the migrated data

---

## AzCopy vs. Robocopy

AzCopy and Robocopy can both be used in storage migration scenarios, but they have different focuses.

| AzCopy | Robocopy |
|---|---|
| Designed for Azure Storage | Designed for Windows file systems |
| Uses Azure Storage endpoints | Uses local or SMB file system paths |
| Optimized for Azure data transfers | Optimized for Windows file copies |
| Supports Blob Storage and Azure Files | Commonly used for SMB file shares |
| Useful for cloud migrations and automation | Useful when Windows file system metadata and NTFS permissions are important |

A typical choice is:

**AzCopy** → Azure-native storage transfers

**Robocopy** → Windows file server and SMB migrations

---

## Troubleshooting

### Authentication Failed

Verify:

- Microsoft Entra authentication
- Azure RBAC permissions
- SAS token validity
- SAS token expiration time
- Storage Account network restrictions

---

### Transfer Failed

Check the AzCopy jobs:

```bash
azcopy jobs list
```

Then inspect the affected job:

```bash
azcopy jobs show <job-id>
```

Review the AzCopy logs for individual transfer failures.

---

### Storage Endpoint Cannot Be Reached

Verify:

- DNS resolution
- Internet or private network connectivity
- Storage Account firewall
- Private Endpoint configuration
- Proxy configuration
- Required Azure Storage endpoints

---

### Access Denied

An authenticated identity still requires the appropriate permissions for the requested operation.

Check:

- Azure RBAC
- SAS permissions
- Azure Files permissions where applicable

---

## Best Practices

- Prefer identity-based authentication where appropriate.
- Avoid exposing Storage Account keys in scripts.
- Use SAS tokens with the minimum required permissions and lifetime when SAS is required.
- Test migration commands with a small dataset first.
- Review source and destination paths carefully before using `sync`.
- Monitor AzCopy jobs during large migrations.
- Validate transferred data after migration.
- Keep AzCopy updated.

---

## Key Takeaways

- **AzCopy** is Microsoft's command-line utility for high-performance Azure Storage transfers.
- It can upload, download, copy, and synchronize data.
- It supports Azure Blob Storage and Azure Files.
- `copy` is typically used for one-time transfers and migrations.
- `sync` can synchronize a source and destination.
- AzCopy supports automation and large-scale data migration scenarios.
- For Windows-centric file migrations where NTFS metadata and permissions are important, **Robocopy** may be the more appropriate tool.
