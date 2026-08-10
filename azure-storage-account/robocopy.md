# Robocopy

## Overview

**Robocopy (Robust File Copy)** is a command-line utility included with Windows for copying files and directories.

It is commonly used by administrators for:

- File server migrations
- Copying large directory structures
- Migrating data to SMB file shares
- Copying data to Azure Files through SMB
- Preserving NTFS permissions and file metadata
- Resuming interrupted file transfers
- Automating recurring copy operations

Robocopy is particularly useful for **Windows-based file migrations** because it provides extensive control over permissions, metadata, retries, logging, and multithreaded transfers.

---

## Basic Command Structure

The general syntax is:

```cmd
robocopy <source> <destination> [options]
```

Example:

```cmd
robocopy C:\Data D:\Backup
```

For network shares, UNC paths can be used:

```cmd
robocopy C:\Data \\fileserver\share
```

---

## Basic Directory Copy

Copy a directory including all subdirectories:

```cmd
robocopy C:\Data D:\Backup /E
```

`/E` copies all subdirectories, including empty directories.

Example:

```text
C:\Data
   │
   │ Robocopy
   ▼
D:\Backup
```

---

## Copy to Azure Files

Because Azure Files provides SMB file shares, a mounted Azure File Share can be used as a Robocopy destination.

Example UNC path:

```text
\\<storage-account>.file.core.windows.net\<file-share>
```

Example:

```cmd
robocopy D:\CompanyData \\stfilesdemo001.file.core.windows.net\documents /E
```

The Azure File Share must already be accessible from the Windows system running Robocopy.

Authentication and permissions therefore need to be configured before starting the migration.

---

## Copy Using a Mapped Drive

If the Azure File Share has already been mapped to a drive letter:

```text
Z:
```

the migration can also be started using:

```cmd
robocopy D:\CompanyData Z:\ /E
```

Using the UNC path directly is generally preferable for scripts because it does not depend on an existing drive mapping.

---

## Preserve File Metadata

One of the most important Robocopy options is `/COPY`.

The available copy attributes are:

```text
D = Data
A = Attributes
T = Timestamps
S = Security (NTFS ACLs)
O = Owner information
U = Auditing information
```

The default is:

```cmd
/COPY:DAT
```

This copies:

- Data
- Attributes
- Timestamps

---

## Preserve NTFS Permissions

For migrations where NTFS permissions must be retained, security information must also be copied.

Example:

```cmd
robocopy D:\CompanyData Z:\ /E /COPY:DATS
```

This includes:

- Data
- Attributes
- Timestamps
- Security / NTFS ACLs

To copy all available file information:

```cmd
robocopy D:\CompanyData Z:\ /E /COPYALL
```

`/COPYALL` is equivalent to:

```text
/COPY:DATSOU
```

This includes:

- Data
- Attributes
- Timestamps
- NTFS permissions
- Owner
- Auditing information

The account running Robocopy must have sufficient permissions to read and write the requested metadata.

---

## Directory Metadata

File metadata and directory metadata are controlled separately.

The `/DCOPY` option defines which directory information should be copied.

Example:

```cmd
/DCOPY:DAT
```

This preserves:

- Directory data
- Attributes
- Timestamps

Example migration:

```cmd
robocopy D:\CompanyData Z:\ /E /COPY:DATS /DCOPY:DAT
```

---

## Multithreaded Copy

Robocopy supports multithreaded transfers using `/MT`.

Example:

```cmd
robocopy D:\CompanyData Z:\ /E /MT:16
```

This uses 16 threads.

Another example:

```cmd
/MT:32
```

Multithreading can significantly improve performance when migrating many files.

The optimal number of threads depends on:

- File sizes
- Number of files
- Network bandwidth
- Storage performance
- Source and destination performance

Higher values do not automatically result in better performance.

---

## Retry Behavior

By default, Robocopy can retry failed operations many times.

For migrations, it is often useful to explicitly control retry behavior.

Example:

```cmd
/R:3 /W:5
```

This means:

```text
/R:3 = Retry failed copies 3 times
/W:5 = Wait 5 seconds between retries
```

Example:

```cmd
robocopy D:\CompanyData Z:\ /E /R:3 /W:5
```

This prevents a migration from becoming stuck for a long time because of a single inaccessible file.

---

## Restartable Mode

Robocopy supports restartable copies:

```cmd
/Z
```

Example:

```cmd
robocopy D:\CompanyData Z:\ /E /Z
```

If a transfer is interrupted, restartable mode can help Robocopy continue the transfer rather than starting the affected file again from the beginning.

Another option is:

```cmd
/ZB
```

This uses restartable mode and falls back to backup mode if access is denied and the executing account has the required backup privileges.

---

## Logging

For migrations, Robocopy output should normally be written to a log file.

Example:

```cmd
robocopy D:\CompanyData Z:\ /E /LOG:C:\Logs\migration.log
```

To append to an existing log:

```cmd
/LOG+:C:\Logs\migration.log
```

Useful additional output options include:

```text
/TEE
```

Displays output in the console while also writing it to the log.

Example:

```cmd
robocopy D:\CompanyData Z:\ /E /LOG:C:\Logs\migration.log /TEE
```

---

## Example Migration Command

A practical file migration command could look like:

```cmd
robocopy D:\CompanyData Z:\ /E /COPY:DATS /DCOPY:DAT /MT:16 /R:3 /W:5 /LOG:C:\Logs\migration.log /TEE
```

This performs the following:

| Option | Purpose |
|---|---|
| `/E` | Copy all subdirectories including empty ones |
| `/COPY:DATS` | Preserve data, attributes, timestamps and NTFS ACLs |
| `/DCOPY:DAT` | Preserve directory metadata |
| `/MT:16` | Use 16 copy threads |
| `/R:3` | Retry failed copies three times |
| `/W:5` | Wait five seconds between retries |
| `/LOG` | Write migration results to a log |
| `/TEE` | Display output and write it to the log |

The exact options should always be selected according to the migration requirements.

---

## Mirror Mode

Robocopy supports mirroring a source directory to a destination:

```cmd
robocopy D:\CompanyData Z:\ /MIR
```

`/MIR` makes the destination reflect the source directory structure.

This is powerful but potentially destructive.

> **Warning:** `/MIR` can delete files and directories from the destination if they no longer exist in the source.

For this reason, `/MIR` should only be used after the source and destination have been carefully verified.

For initial migration testing, `/E` is generally safer.

---

## Initial Copy and Delta Copy

Robocopy can be useful for migrations where the source file server remains in use during the initial transfer.

A typical migration approach is:

```text
Initial Copy
     │
     ▼
Large dataset transferred
     │
     ▼
Users continue working
     │
     ▼
Delta Copy
     │
     ▼
Final Cutover
```

### Initial Copy

Perform the first large data transfer:

```cmd
robocopy D:\CompanyData Z:\ /E /COPY:DATS /DCOPY:DAT /MT:16 /R:3 /W:5
```

### Delta Copy

Before the final cutover, run Robocopy again.

Files that have not changed generally do not need to be transferred again, allowing the final synchronization to complete much faster than the initial migration.

After validation, user access can be switched to the new file share.

---

## Validation

After the migration, verify:

- Expected files exist
- Expected folders exist
- File sizes are correct
- Timestamps are preserved where required
- NTFS permissions are correct
- Failed files are investigated
- Users can access the destination
- Applications can access required files

Robocopy provides a summary at the end of each operation.

Example categories include:

```text
Total
Copied
Skipped
Mismatch
FAILED
Extras
```

Pay particular attention to:

```text
FAILED
```

A migration should not be considered successful until failed transfers have been investigated.

---

## Robocopy Exit Codes

Robocopy exit codes differ from many traditional command-line applications.

A non-zero exit code does **not automatically mean that the copy failed**.

Robocopy uses different exit codes to describe the result of the operation, including whether files were copied, additional files were detected, or failures occurred.

This is particularly important when Robocopy is used inside scripts or automation.

Do not simply interpret every non-zero Robocopy exit code as an error.

---

## Robocopy vs. AzCopy

Robocopy and AzCopy can both be used in Azure storage migration scenarios, but they have different strengths.

| Robocopy | AzCopy |
|---|---|
| Windows file-copy utility | Azure Storage transfer utility |
| Works with file system and SMB paths | Works directly with Azure Storage endpoints |
| Strong NTFS and Windows metadata support | Optimized for Azure Storage transfers |
| Common for file server migrations | Common for cloud storage migrations |
| Useful for Azure Files over SMB | Supports Azure Files and Blob Storage |
| Windows-centric | Available for Windows, Linux and macOS |

A simplified distinction is:

```text
Robocopy
→ Windows file server / SMB migration

AzCopy
→ Azure-native storage transfer
```

For migrations where preserving Windows file system metadata and NTFS ACLs is important, Robocopy is often particularly useful.

---

## Troubleshooting

### Access Denied

Verify:

- Source permissions
- Destination permissions
- NTFS permissions
- Share-level permissions
- Account used to run Robocopy

When copying security information, additional permissions may be required.

---

### Destination Cannot Be Reached

For an Azure File Share, verify SMB connectivity:

```powershell
Test-NetConnection <storage-account>.file.core.windows.net -Port 445
```

Also verify:

- DNS resolution
- Storage Account firewall
- Private Endpoint configuration
- Network routing
- SMB authentication

---

### Files Are Skipped

Robocopy compares source and destination information and may skip files that it determines do not need to be copied.

Review the Robocopy output and log to understand why files were skipped.

---

### Permission Changes Are Not Preserved

Verify that the selected `/COPY` options include security information.

For example:

```cmd
/COPY:DATS
```

or:

```cmd
/COPYALL
```

Also verify that the executing account has sufficient permissions.

---

### Migration Takes Too Long

Consider:

- Using `/MT`
- Checking available network bandwidth
- Reviewing storage performance
- Reducing excessive retry values
- Investigating individual problematic files

---

## Useful Options

| Option | Purpose |
|---|---|
| `/E` | Copy subdirectories including empty directories |
| `/MIR` | Mirror source and destination |
| `/COPY:DAT` | Copy data, attributes and timestamps |
| `/COPY:DATS` | Also copy NTFS security |
| `/COPYALL` | Copy all file information |
| `/DCOPY:DAT` | Preserve directory metadata |
| `/MT:n` | Enable multithreaded copying |
| `/R:n` | Number of retries |
| `/W:n` | Wait time between retries |
| `/Z` | Restartable mode |
| `/ZB` | Restartable mode with backup mode fallback |
| `/LOG:file` | Write output to a log |
| `/LOG+:file` | Append output to a log |
| `/TEE` | Output to console and log |

---

## Best Practices

- Test the Robocopy command with a small dataset first.
- Use `/E` instead of `/MIR` until mirror behavior is explicitly required.
- Be especially careful with options that can delete destination data.
- Define retry and wait values explicitly.
- Use logging for migrations.
- Preserve NTFS permissions only when required and verify them afterwards.
- Use groups rather than individual accounts for file permissions.
- Perform an initial copy before the final migration window when possible.
- Run a final delta copy before cutover.
- Review the Robocopy summary and failed files after every migration.
- Validate user access after the migration.

---

## Key Takeaways

- **Robocopy** is a powerful Windows utility for copying and migrating files.
- It is particularly useful for Windows file server and SMB migrations.
- Azure File Shares can be used as Robocopy destinations through SMB.
- Robocopy can preserve NTFS permissions, timestamps and other Windows file metadata.
- `/MT` enables multithreaded transfers.
- `/R` and `/W` control retry behavior.
- `/MIR` should be used carefully because it can delete destination data.
- Logging and validation are important for production migrations.
- Robocopy is particularly useful when Windows file system behavior and NTFS permissions are important.
