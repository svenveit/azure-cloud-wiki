# AzCopy

## Übersicht

**AzCopy** ist ein von Microsoft bereitgestelltes Kommandozeilentool zum Übertragen von Daten zwischen lokalen Systemen und **Azure Storage**.

Typische Einsatzgebiete sind:

- Upload von lokalen Dateien nach Azure Storage
- Download von Daten aus Azure Storage
- Kopieren von Daten zwischen Azure Storage Accounts
- Synchronisieren von Verzeichnissen
- Migration größerer Datenmengen
- Automatisierte Datenübertragungen über Skripte

AzCopy ist insbesondere für **Azure Blob Storage** und **Azure Files** relevant.

---

## Installation

AzCopy ist für Windows, Linux und macOS verfügbar.

Nach der Installation kann die Version überprüft werden:

```bash
azcopy --version
```

Die allgemeine Syntax lautet:

```bash
azcopy <command> <source> <destination> [flags]
```

Beispiel:

```bash
azcopy copy "C:\Data" "https://storageaccount.blob.core.windows.net/container" --recursive
```

---

## Authentifizierung

AzCopy unterstützt verschiedene Authentifizierungsmöglichkeiten.

### Microsoft Entra ID

Interaktive Anmeldung:

```bash
azcopy login
```

Anschließend können Daten übertragen werden, sofern der angemeldete Benutzer die notwendigen **Azure RBAC-Berechtigungen** besitzt.

Für Blob Storage ist beispielsweise häufig folgende Rolle relevant:

```text
Storage Blob Data Contributor
```

Abmelden:

```bash
azcopy logout
```

> Die notwendigen Berechtigungen unterscheiden sich je nach verwendetem Azure-Storage-Dienst und Operation.

---

### SAS-Token

Alternativ kann ein **Shared Access Signature (SAS)** verwendet werden.

Beispiel:

```bash
azcopy copy "C:\Data\file.txt" "https://storageaccount.blob.core.windows.net/container/file.txt?<SAS-TOKEN>"
```

Das SAS-Token wird direkt an die URL angehängt.

> SAS-Tokens sollten wie Zugangsdaten behandelt und nicht in Git-Repositories oder öffentlich zugänglichen Skripten gespeichert werden.

---

## Wichtige Befehle

### `copy`

Kopiert Dateien oder Verzeichnisse von einer Quelle zu einem Ziel.

```bash
azcopy copy "<source>" "<destination>"
```

---

### `sync`

Synchronisiert Quelle und Ziel.

```bash
azcopy sync "<source>" "<destination>"
```

AzCopy vergleicht dabei Quelle und Ziel und überträgt die erforderlichen Daten.

> `sync` sollte mit Bedacht verwendet werden. Insbesondere Optionen zum Löschen von Dateien am Ziel sollten vor produktiver Verwendung geprüft werden.

---

### `list`

Listet Dateien bzw. Objekte an einem Azure-Storage-Ziel auf.

```bash
azcopy list "https://storageaccount.blob.core.windows.net/container?<SAS-TOKEN>"
```

---

## Upload zu Azure Blob Storage

### Einzelne Datei

```bash
azcopy copy "C:\Data\report.pdf" "https://storageaccount.blob.core.windows.net/container/report.pdf?<SAS-TOKEN>"
```

---

### Komplettes Verzeichnis

```bash
azcopy copy "C:\Data" "https://storageaccount.blob.core.windows.net/container?<SAS-TOKEN>" --recursive
```

Mit:

```text
--recursive
```

werden auch Unterverzeichnisse berücksichtigt.

---

## Download von Azure Blob Storage

Ein kompletter Container kann beispielsweise lokal heruntergeladen werden:

```bash
azcopy copy "https://storageaccount.blob.core.windows.net/container?<SAS-TOKEN>" "C:\Backup" --recursive
```

---

## Azure Files

AzCopy kann ebenfalls für **Azure Files** verwendet werden.

Die URL eines File Shares hat grundsätzlich folgende Struktur:

```text
https://<storage-account>.file.core.windows.net/<file-share>
```

### Upload zu Azure Files

```bash
azcopy copy "C:\Data" "https://storageaccount.file.core.windows.net/fileshare?<SAS-TOKEN>" --recursive
```

### Download von Azure Files

```bash
azcopy copy "https://storageaccount.file.core.windows.net/fileshare?<SAS-TOKEN>" "C:\Data" --recursive
```

AzCopy überträgt Azure Files dabei über die Azure Storage REST API. Der File Share muss dafür nicht wie bei einer klassischen SMB-Verbindung als Netzlaufwerk eingebunden sein.

---

## Daten zwischen Storage Accounts kopieren

AzCopy kann Daten auch direkt zwischen Azure-Storage-Zielen übertragen.

Beispiel:

```bash
azcopy copy \
"https://storageaccount1.blob.core.windows.net/container?<SAS-TOKEN>" \
"https://storageaccount2.blob.core.windows.net/container?<SAS-TOKEN>" \
--recursive
```

Dies ist beispielsweise bei Migrationen zwischen Storage Accounts nützlich.

---

## Copy vs. Sync

### Copy

```bash
azcopy copy
```

wird verwendet, um Daten von einer Quelle zu einem Ziel zu kopieren.

Typische Verwendung:

```text
Lokales Verzeichnis
        │
        ▼
   Azure Storage
```

### Sync

```bash
azcopy sync
```

gleicht Quelle und Ziel miteinander ab.

Typische Verwendung:

```text
Quelle
   │
   │ Vergleich
   ▼
Ziel
```

Für einmalige Migrationen ist häufig **`copy`** ausreichend.

---

## Nützliche Optionen

### Rekursives Kopieren

```bash
--recursive
```

Berücksichtigt Unterverzeichnisse.

Beispiel:

```bash
azcopy copy "C:\Data" "<destination>" --recursive
```

---

### Vorhandene Dateien überschreiben

Das Verhalten beim Überschreiben kann über `--overwrite` gesteuert werden.

Beispiel:

```bash
azcopy copy "C:\Data" "<destination>" --recursive --overwrite=ifSourceNewer
```

Damit werden vorhandene Dateien nur überschrieben, wenn die Quelldatei neuer ist.

---

### Simulation mit Dry Run

Bei `sync` kann vor der tatsächlichen Ausführung geprüft werden, welche Änderungen durchgeführt würden:

```bash
azcopy sync "<source>" "<destination>" --dry-run
```

Dies ist besonders bei größeren oder produktiven Datenbeständen sinnvoll.

---

## Jobs

AzCopy verwaltet Übertragungen als **Jobs**.

Vorhandene Jobs anzeigen:

```bash
azcopy jobs list
```

Informationen zu einem bestimmten Job:

```bash
azcopy jobs show <job-id>
```

Ein unterbrochener Job kann fortgesetzt werden:

```bash
azcopy jobs resume <job-id>
```

Das ist insbesondere bei großen Datenübertragungen und instabilen Netzwerkverbindungen hilfreich.

---

## AzCopy und Robocopy

AzCopy und Robocopy können beide bei Datenmigrationen eingesetzt werden, verfolgen jedoch unterschiedliche Ansätze.

| AzCopy | Robocopy |
|---|---|
| Microsoft-Tool für Azure Storage | Windows-Dateikopiertool |
| Azure Storage REST APIs | Dateisystem-/SMB-basierter Zugriff |
| Blob Storage und Azure Files | Windows-Dateisysteme und SMB Shares |
| Unterstützt Azure-spezifische Authentifizierung | Arbeitet mit Dateisystemberechtigungen |
| Gut für Cloud-Datenübertragungen | Gut für klassische Fileserver-Migrationen |

Bei einer Migration eines klassischen Windows-Fileservers nach **Azure Files** kann deshalb je nach Anforderungen entweder AzCopy oder Robocopy sinnvoll sein.

---

## Typisches Einsatzszenario

Beispiel einer Datenmigration:

```text
On-Premises Server
        │
        │ AzCopy
        ▼
    Internet / VPN
        │
        ▼
 Azure Storage Account
        │
        └── Azure Files / Blob Storage
```

AzCopy eignet sich besonders für schnelle und automatisierbare Datenübertragungen zwischen lokalen Systemen und Azure Storage.

---
## Merksatz

> **AzCopy ist ein Kommandozeilentool für performante Datenübertragungen zu, von und zwischen Azure Storage-Diensten.**
