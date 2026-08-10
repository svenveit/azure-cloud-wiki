# Azure File Sync

## Overview

**Azure File Sync** enables files stored on a Windows Server to be synchronized with an **Azure File Share**.

This allows an existing Windows file server to remain available locally while Azure Files provides a centralized cloud location for the synchronized data.

A typical Azure File Sync setup consists of:

- Azure File Share
- Storage Sync Service
- Sync Group
- Cloud Endpoint
- Registered Windows Server
- Server Endpoint
- Azure File Sync Agent

---

## Architecture

A simplified architecture looks like this:

```text
Windows Server
└── C:\FileSync
        │
        │ Azure File Sync Agent
        ▼
   Server Endpoint
        │
        ▼
     Sync Group
        │
        ▼
   Cloud Endpoint
        │
        ▼
 Azure File Share
```

The **Storage Sync Service** is the central Azure resource used to manage the synchronization configuration, registered servers, and Sync Groups.

---

## 1. Create the Storage Sync Service

Create a **Storage Sync Service** in the Azure portal.

The Storage Sync Service acts as the management layer for Azure File Sync.

After deployment, the service provides access to components such as:

- Sync Groups
- Registered Servers
- Monitoring

![Storage Sync Service](images/azure-file-sync-storage-sync-service.png)

---

## 2. Create a Sync Group

Inside the Storage Sync Service, create a **Sync Group**.

A Sync Group defines which locations participate in the synchronization.

The Azure File Share is configured as the **Cloud Endpoint**.

Later, the local Windows Server directory is added as a **Server Endpoint**.

A Sync Group therefore connects:

```text
Azure File Share
       ↕
   Sync Group
       ↕
Windows Server
```

---

## 3. Install the Azure File Sync Agent

The Windows Server requires the **Azure File Sync Agent**.

Download the agent version that matches the Windows Server version.

![Azure File Sync Agent Download](images/azure-file-sync-agent-download.png)

Install the agent on the Windows Server.

After installation, verify that the Azure File Sync Agent is installed and up to date.

![Azure File Sync Agent Installed](images/azure-file-sync-agent-installed.png)

---

## 4. Register the Windows Server

After installing the agent, register the Windows Server with the **Storage Sync Service**.

During registration, select the corresponding:

- Azure Subscription
- Resource Group
- Storage Sync Service

After successful registration, the server appears under:

**Storage Sync Service → Registered servers**

The registered server can now participate in a Sync Group.

---

## 5. Create the Server Endpoint

Add a **Server Endpoint** to the Sync Group.

The Server Endpoint defines the local Windows Server path that should be synchronized.

Example:

```text
Server: FILE01
Path:   C:\FileSync
```

The existing Azure File Share acts as the **Cloud Endpoint**.

After configuration, both endpoints can be viewed in the Sync Group.

![Azure File Sync Sync Group](images/azure-file-sync-sync-group.png)

The synchronization status should eventually report **Healthy**.

---

## 6. Test the Synchronization

Files can now be created or copied into the configured directory on the Windows Server.

Example:

```text
C:\FileSync
```

The following screenshot shows the local folder used as the Server Endpoint:

![Local File Sync Folder](images/azure-file-sync-local-folder.png)

Azure File Sync synchronizes the files with the configured Azure File Share.

The synchronized files can then be verified in the Azure portal:

![Azure File Share](images/azure-file-sync-file-share.png)

This confirms the basic synchronization path:

```text
Windows Server
C:\FileSync
      │
      ▼
Azure File Sync
      │
      ▼
Azure File Share
```

Changes can also be synchronized in the opposite direction, allowing the Windows Server and Azure File Share to maintain synchronized data.

---

## Cloud Tiering

Azure File Sync optionally supports **Cloud Tiering**.

With Cloud Tiering enabled, frequently accessed files can remain cached locally while less frequently used files are tiered to Azure Files.

From the user's perspective, the files remain visible on the Windows Server. When a tiered file is accessed, its contents can be recalled from Azure Files.

Cloud Tiering can therefore reduce the amount of local storage required on the file server.

```text
Frequently used data
        │
        ▼
Local Windows Server

Less frequently used data
        │
        ▼
Azure File Share
```

Cloud Tiering is optional and can be configured on the Server Endpoint.

---

## Important Concepts

| Component | Purpose |
|---|---|
| **Storage Sync Service** | Central management resource for Azure File Sync |
| **Sync Group** | Defines the synchronization relationship |
| **Cloud Endpoint** | Azure File Share participating in the Sync Group |
| **Server Endpoint** | Local Windows Server path participating in the Sync Group |
| **Registered Server** | Windows Server registered with the Storage Sync Service |
| **Azure File Sync Agent** | Software installed on the Windows Server to enable synchronization |
| **Cloud Tiering** | Optional feature for keeping selected file content primarily in Azure |

---

## Important Notes

- Azure File Sync is designed for synchronization between Windows Servers and Azure Files.
- The Windows Server must have the Azure File Sync Agent installed.
- A server must be registered with the Storage Sync Service before a Server Endpoint can be created.
- One Sync Group contains one Cloud Endpoint and can contain one or more Server Endpoints.
- Cloud Tiering is optional.
- Synchronization may not always occur immediately; changes can take some time to appear.
- Azure File Sync should not be considered a replacement for a dedicated backup strategy.

---

## Lab Result

The lab successfully demonstrated the basic Azure File Sync workflow:

```text
Windows Server
C:\FileSync
      │
      ▼
Azure File Sync Agent
      │
      ▼
Server Endpoint
      │
      ▼
Sync Group
      │
      ▼
Cloud Endpoint
      │
      ▼
Azure File Share
```

Files created on the Windows Server were successfully synchronized with the Azure File Share, demonstrating the basic functionality of Azure File Sync in a hybrid file server scenario.
