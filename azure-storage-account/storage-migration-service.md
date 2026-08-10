# Storage Migration Service

## Overview

**Storage Migration Service (SMS)** is a Windows Server feature that simplifies the migration of file servers to newer Windows Servers or Azure virtual machines.

It can inventory existing file servers, transfer files and file shares, preserve security configuration, and optionally move the **server identity** to the destination server.

This makes Storage Migration Service particularly useful when modernizing legacy file servers without requiring users or applications to change existing UNC paths or server names. :contentReference[oaicite:0]{index=0}

---

## Typical Use Cases

Storage Migration Service is commonly used for:

- Replacing legacy Windows File Servers
- Migrating file servers to newer Windows Server versions
- Migrating file servers to Azure VMs
- Migrating SMB file shares
- Preserving existing share configuration and permissions
- Moving server identities during a cutover
- Consolidating file servers

Supported sources can include Windows Servers, Samba-based Linux servers, and supported NetApp CIFS environments. :contentReference[oaicite:1]{index=1}

---

## Architecture

A typical migration consists of three systems:

```text
Source File Server
        │
        │ Inventory + Data Transfer
        ▼
Storage Migration Service
     Orchestrator
        │
        ▼
Destination Server
```

The migration is normally managed through **Windows Admin Center**.

The orchestrator coordinates the inventory, transfer, and optional cutover process.

---

## Core Components

| Component | Purpose |
|---|---|
| **Source Server** | Existing server containing files and shares |
| **Destination Server** | New server receiving the migrated data |
| **Orchestrator** | Coordinates the migration |
| **Windows Admin Center** | Management interface for Storage Migration Service |
| **Storage Migration Service** | Performs inventory, transfer, and cutover |

For a simple single-server migration, the destination server can also act as the orchestrator. For multiple migrations, a dedicated orchestrator can be useful. :contentReference[oaicite:2]{index=2}

---

## Migration Workflow

Storage Migration Service uses three main phases:

```text
Inventory
    │
    ▼
Transfer
    │
    ▼
Cutover
```

The **Cutover** phase is optional.

---

## 1. Inventory

The first phase scans the source file server.

Storage Migration Service collects information about:

- Files
- Folders
- File shares
- Storage volumes
- Network configuration
- Security configuration
- Server identity

The inventory phase helps identify exactly what should be migrated before any data is transferred. :contentReference[oaicite:3]{index=3}

A simplified workflow:

```text
Source Server
      │
      ▼
Inventory Scan
      │
      ▼
Files
Shares
Volumes
Configuration
```

---

## 2. Transfer

After the inventory has completed, the selected data is transferred to the destination server.

Typical transferred components include:

- Files
- Folders
- SMB shares
- File attributes
- Security information

The source server remains available during the transfer phase.

This makes it possible to perform the bulk of the migration before the final cutover window. :contentReference[oaicite:4]{index=4}

---

## 3. Cutover

The optional **Cutover** phase transfers the network identity of the source server to the destination server.

This can include:

- Computer name
- IP configuration
- Server identity

After the cutover, users and applications can continue accessing the file server using the existing name and paths.

Example:

```text
Before Cutover

\\FILESERVER01\Data
        │
        ▼
Old Server


After Cutover

\\FILESERVER01\Data
        │
        ▼
New Server
```

This reduces the need to update:

- Drive mappings
- Applications
- Scripts
- UNC paths
- User configuration

The original source server keeps its data but is moved into a state where it is no longer available to users and applications. :contentReference[oaicite:5]{index=5}

---

## Requirements

A typical Storage Migration Service deployment requires:

- Source file server
- Destination Windows Server
- Storage Migration Service orchestrator
- Windows Admin Center
- Administrative permissions
- Network connectivity between the systems
- Sufficient storage capacity on the destination

Microsoft recommends using a modern Windows Server version for the destination and orchestrator. :contentReference[oaicite:6]{index=6}

---

## Windows Admin Center

Storage Migration Service is normally managed through **Windows Admin Center**.

The migration workflow is presented as a guided process:

```text
Windows Admin Center
        │
        ▼
Storage Migration Service
        │
        ├── Inventory
        ├── Transfer
        └── Cutover
```

This provides a more structured migration workflow than manually copying files and recreating server configuration.

---

## Example Migration Scenario

A typical scenario could be:

```text
Windows Server 2012 R2
      FILE01
        │
        │ Inventory
        ▼
Storage Migration Service
        │
        │ Transfer
        ▼
Windows Server 2025
      FILE02
        │
        │ Cutover
        ▼
Becomes FILE01
```

After the migration:

- Files are located on the new server.
- Shares are available.
- Existing paths can remain unchanged.
- The old server can be decommissioned.

---

## Storage Migration Service vs. Robocopy

Robocopy and Storage Migration Service can both be used for file server migrations, but they solve different levels of the migration.

| Storage Migration Service | Robocopy |
|---|---|
| Complete migration workflow | File-copy utility |
| Inventories the source server | No inventory phase |
| Transfers files and shares | Primarily transfers files |
| Can migrate server identity | Does not migrate server identity |
| Managed through Windows Admin Center | Command-line based |
| Includes optional cutover | Cutover must be managed separately |

Robocopy is useful when only the data needs to be copied.

Storage Migration Service is useful when the **entire file server should be modernized**, including shares, configuration, and potentially the server identity.

---

## Storage Migration Service vs. Azure Storage Mover

The two services have different focuses.

| Storage Migration Service | Azure Storage Mover |
|---|---|
| Windows Server migration technology | Azure-managed storage migration service |
| Destination typically Windows Server or Azure VM | Destination is Azure Storage |
| Can migrate server identity | Does not migrate Windows server identity |
| Focus on file server modernization | Focus on moving datasets to Azure Storage |
| Managed using Windows Admin Center | Managed through Azure |

A simplified distinction:

```text
Storage Migration Service
→ Replace / modernize a file server

Azure Storage Mover
→ Move data into Azure Storage
```

---

## Storage Migration Service vs. Azure File Sync

Storage Migration Service performs a **migration**.

Azure File Sync provides **ongoing synchronization**.

```text
Storage Migration Service
Source Server
      │
      │ Migration
      ▼
New Server


Azure File Sync
Windows Server
      │
      │ Continuous Sync
      ↕
Azure File Share
```

Storage Migration Service can also be used in scenarios where the destination server later runs the Azure File Sync agent. Microsoft notes that Azure Files itself is not a direct Storage Migration Service destination, while servers using Azure File Sync are supported. :contentReference[oaicite:7]{index=7}

---

## Validation

After the migration, verify:

- Expected files were transferred
- File shares exist
- Permissions are correct
- Applications can access the shares
- Users can access the expected UNC paths
- Destination storage capacity is correct
- Network configuration is correct
- Server identity is correct after cutover

If a cutover was performed, also verify:

```text
\\<original-server-name>\<share>
```

Users should reach the new destination server without requiring changes to their existing paths.

---

## Troubleshooting

### Inventory Fails

Verify:

- Administrative permissions
- Firewall configuration
- DNS resolution
- Connectivity to the source server
- Windows Admin Center connectivity

---

### Transfer Fails

Check:

- Available destination storage
- File permissions
- Network connectivity
- Unsupported files or paths
- Destination file system

Microsoft recommends checking destination capacity, OS compatibility, and the underlying file systems as part of troubleshooting migration failures. :contentReference[oaicite:8]{index=8}

---

### Cutover Fails

Verify:

- Domain connectivity
- DNS
- Network configuration
- Administrative permissions
- Computer account permissions
- Source and destination server configuration

Because cutover changes the server identity, this phase should normally be performed during a controlled maintenance window.

---

## Best Practices

- Perform an inventory before planning the final migration.
- Validate destination storage capacity.
- Test the migration in a lab where possible.
- Perform the bulk data transfer before the final maintenance window.
- Schedule the cutover separately.
- Validate file shares and permissions after migration.
- Verify application dependencies before decommissioning the old server.
- Keep the original server available until the migration has been fully validated.
- Back up critical data before performing production migrations.

---

## Key Takeaways

- **Storage Migration Service** is designed for complete file server migration and modernization.
- The migration consists of **Inventory, Transfer, and optional Cutover**.
- Files, shares, and security configuration can be migrated.
- Cutover can transfer the source server identity to the new server.
- Existing UNC paths can therefore remain unchanged.
- Storage Migration Service is managed primarily through **Windows Admin Center**.
- It is broader than Robocopy because it handles more than file transfer.
- It differs from Azure Storage Mover, which focuses on moving data into Azure Storage.
