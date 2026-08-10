# Robocopy

## Übersicht

**Robocopy (Robust File Copy)** ist ein in Windows integriertes Kommandozeilentool zum Kopieren und Synchronisieren von Dateien und Verzeichnissen.

Robocopy ist insbesondere für große Datenmengen und Fileserver-Migrationen geeignet.

Typische Einsatzgebiete:

- Migration von Windows-Fileservern
- Kopieren großer Datenmengen
- Synchronisieren von Verzeichnissen
- Kopieren über SMB-Netzwerkfreigaben
- Übernahme von NTFS-Berechtigungen
- Migration von Daten zu **Azure Files**
- Wiederaufnahme unterbrochener Kopiervorgänge

Robocopy ist standardmäßig in modernen Windows- und Windows-Server-Versionen enthalten.

---

## Grundlegende Syntax

Die allgemeine Syntax lautet:

```powershell
robocopy <Quelle> <Ziel> [Optionen]
```

Beispiel:

```powershell
robocopy "C:\Data" "D:\Backup"
```

Dabei wird der Inhalt von `C:\Data` nach `D:\Backup` kopiert.

---

## Verzeichnisse rekursiv kopieren

Mit `/E` werden alle Unterverzeichnisse einschließlich leerer Verzeichnisse kopiert:

```powershell
robocopy "C:\Data" "D:\Backup" /E
```

Alternativ existiert `/S`:

```powershell
robocopy "C:\Data" "D:\Backup" /S
```

Unterschied:

- `/S` → kopiert Unterverzeichnisse, aber keine leeren Verzeichnisse
- `/E` → kopiert Unterverzeichnisse einschließlich leerer Verzeichnisse

Für Migrationen wird häufig `/E` verwendet.

---

## Kopieren über SMB

Robocopy kann direkt mit UNC-Pfaden arbeiten.

Beispiel:

```powershell
robocopy "C:\Data" "\\fileserver\share" /E
```

Auch Quelle und Ziel können Netzwerkfreigaben sein:

```powershell
robocopy "\\server01\share" "\\server02\share" /E
```

Robocopy eignet sich deshalb besonders für klassische Fileserver-Migrationen.

---

## Azure Files

Ein Azure File Share kann über **SMB** eingebunden werden.

Beispielsweise als Laufwerk:

```text
Z:\
```

Anschließend kann Robocopy wie bei einem normalen Dateisystem verwendet werden:

```powershell
robocopy "D:\Data" "Z:\Data" /E
```

Alternativ kann mit einem entsprechenden UNC-Pfad gearbeitet werden.

Typisches Szenario:

```text
On-Premises Fileserver
        │
        │ SMB
        │
        │ Robocopy
        ▼
   Azure File Share
```

> Für SMB-Zugriff auf Azure Files muss die entsprechende Netzwerkverbindung funktionieren. Bei direktem Zugriff über den öffentlichen Endpoint ist insbesondere TCP-Port 445 relevant.

---

## NTFS-Berechtigungen

Ein wichtiger Vorteil von Robocopy bei Fileserver-Migrationen ist die Möglichkeit, neben den Dateien auch Metadaten und NTFS-Berechtigungen zu kopieren.

### COPY-Option

Mit `/COPY` kann festgelegt werden, welche Dateiinformationen übernommen werden.

Die möglichen Werte sind:

```text
D = Data
A = Attributes
T = Timestamps
S = Security (NTFS ACLs)
O = Owner
U = Auditing
```

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /COPY:DATS
```

Dabei werden übertragen:

- Daten
- Attribute
- Zeitstempel
- NTFS-Berechtigungen

---

## COPYALL

Mit:

```powershell
/COPYALL
```

werden alle Dateiinformationen kopiert.

Das entspricht:

```powershell
/COPY:DATSOU
```

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /COPYALL
```

Dazu gehören:

- Daten
- Attribute
- Zeitstempel
- NTFS ACLs
- Besitzerinformationen
- Auditing-Informationen

> Für das Kopieren bestimmter Sicherheitsinformationen sind entsprechende Windows-Berechtigungen erforderlich.

---

## Verzeichnisberechtigungen

Für Verzeichnisse existiert zusätzlich `/DCOPY`.

Beispiel:

```powershell
/DCOPY:DAT
```

Dabei steht:

```text
D = Data
A = Attributes
T = Timestamps
```

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /COPY:DATS /DCOPY:DAT
```

---

## Mirror-Modus

Mit:

```powershell
/MIR
```

kann das Zielverzeichnis an die Quelle angeglichen werden.

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /MIR
```

`/MIR` entspricht im Wesentlichen:

```text
/E + /PURGE
```

Dadurch werden:

- neue Dateien kopiert
- geänderte Dateien aktualisiert
- Unterverzeichnisse berücksichtigt
- Dateien am Ziel gelöscht, wenn sie an der Quelle nicht mehr vorhanden sind

> **Vorsicht:** `/MIR` kann Dateien und Verzeichnisse am Ziel löschen. Vor der Verwendung sollte unbedingt geprüft werden, ob dieses Verhalten gewünscht ist.

---

## Multi-Threading

Mit `/MT` kann Robocopy mehrere Dateien parallel übertragen.

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /MT:16
```

Hier werden bis zu 16 Threads verwendet.

Weitere Beispiele:

```powershell
/MT:8
/MT:16
/MT:32
```

Multi-Threading kann die Übertragung vieler Dateien erheblich beschleunigen.

Eine höhere Anzahl an Threads ist jedoch nicht automatisch besser und sollte an Netzwerk, Storage und Workload angepasst werden.

---

## Wiederholungsversuche

Standardmäßig kann Robocopy fehlgeschlagene Kopiervorgänge mehrfach versuchen.

Für Migrationen ist es häufig sinnvoll, die Anzahl der Wiederholungen explizit festzulegen.

Beispiel:

```powershell
/R:3
```

Maximal drei Wiederholungsversuche.

Mit `/W` wird die Wartezeit zwischen den Versuchen angegeben:

```powershell
/W:5
```

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /R:3 /W:5
```

Damit versucht Robocopy einen fehlgeschlagenen Vorgang maximal dreimal und wartet jeweils fünf Sekunden.

---

## Restartable Mode

Mit:

```powershell
/Z
```

wird der **Restartable Mode** aktiviert.

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /Z
```

Dadurch können unterbrochene Dateiübertragungen besser fortgesetzt werden.

Dies kann insbesondere bei großen Dateien und instabilen Netzwerkverbindungen hilfreich sein.

---

## Backup Mode

Mit:

```powershell
/B
```

kann Robocopy im Backup-Modus ausgeführt werden.

Dieser Modus verwendet Windows-Backup-Rechte und kann beispielsweise bei Dateien relevant sein, auf die der ausführende Benutzer normalerweise keinen direkten Zugriff besitzt.

Dafür sind entsprechende Berechtigungen erforderlich.

Es existiert außerdem:

```powershell
/ZB
```

Robocopy verwendet zunächst den Restartable Mode und wechselt bei einem Zugriffsproblem in den Backup Mode.

---

## Logging

Bei größeren Migrationen sollten die Ergebnisse protokolliert werden.

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /LOG:"C:\Logs\robocopy.log"
```

Mit:

```powershell
/TEE
```

wird die Ausgabe gleichzeitig:

- im Terminal angezeigt
- in die Logdatei geschrieben

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /E /LOG:"C:\Logs\robocopy.log" /TEE
```

Mit `/LOG+` kann an eine bestehende Logdatei angehängt werden:

```powershell
/LOG+:"C:\Logs\robocopy.log"
```

---

## Dateien vorab prüfen

Vor einer größeren Migration kann mit:

```powershell
/L
```

ein Testlauf durchgeführt werden.

Beispiel:

```powershell
robocopy "D:\Data" "Z:\Data" /MIR /L
```

Robocopy zeigt dann an, welche Aktionen durchgeführt würden, ohne die Dateien tatsächlich zu kopieren oder zu löschen.

Dies ist insbesondere vor der Verwendung von `/MIR` sinnvoll.

---

## Typischer Befehl für eine Migration

Ein möglicher Ausgangspunkt für eine Fileserver-Migration ist:

```powershell
robocopy "D:\Data" "Z:\Data" /E /COPY:DATS /DCOPY:DAT /MT:16 /R:3 /W:5 /LOG:"C:\Logs\migration.log" /TEE
```

Bedeutung:

| Option | Funktion |
|---|---|
| `/E` | Alle Unterverzeichnisse einschließlich leerer Verzeichnisse |
| `/COPY:DATS` | Daten, Attribute, Zeitstempel und ACLs kopieren |
| `/DCOPY:DAT` | Verzeichnisinformationen übernehmen |
| `/MT:16` | 16 parallele Threads |
| `/R:3` | Maximal drei Wiederholungsversuche |
| `/W:5` | Fünf Sekunden zwischen Wiederholungen |
| `/LOG` | Ausgabe protokollieren |
| `/TEE` | Ausgabe zusätzlich im Terminal anzeigen |

Die tatsächlich benötigten Optionen hängen von den Anforderungen der jeweiligen Migration ab.

---

## Typisches Migrationsverfahren

Bei einer größeren Fileserver-Migration kann Robocopy mehrfach ausgeführt werden.

### 1. Initiale Migration

Zunächst wird der komplette Datenbestand kopiert:

```text
Fileserver
    │
    │ große Datenmenge
    ▼
Azure Files
```

### 2. Delta-Kopie

Da während der ersten Übertragung weiterhin Dateien geändert werden können, wird Robocopy später erneut ausgeführt.

Dabei müssen hauptsächlich neue oder geänderte Dateien übertragen werden.

```text
Fileserver
    │
    │ Änderungen seit Initial Copy
    ▼
Azure Files
```

### 3. Finaler Cutover

Zum geplanten Umstellungszeitpunkt werden Änderungen am alten Share verhindert und ein letzter Robocopy-Lauf durchgeführt.

Danach greifen die Benutzer bzw. Anwendungen auf das neue Azure File Share zu.

Dieses Vorgehen reduziert die notwendige Downtime bei größeren Migrationen.

---

## Robocopy vs. AzCopy

| Robocopy | AzCopy |
|---|---|
| Windows-Dateikopiertool | Azure Storage Transfer Tool |
| Dateisystem-/SMB-basiert | Azure Storage APIs |
| Besonders für Fileserver geeignet | Besonders für Azure Storage geeignet |
| Unterstützt NTFS-Metadaten und ACLs | Unterstützt Azure-spezifische Storage-Szenarien |
| Quelle/Ziel erscheinen als Dateisystem | Arbeitet direkt mit Azure Storage URLs |
| Nur Windows | Windows, Linux und macOS |

Für eine klassische **Windows-Fileserver-Migration nach Azure Files** ist Robocopy besonders interessant, wenn bestehende Datei- und Verzeichnisstrukturen einschließlich Berechtigungsinformationen übernommen werden sollen.

---

## Wichtige Optionen auf einen Blick

```text
/E           Unterverzeichnisse inklusive leerer Verzeichnisse
/S           Unterverzeichnisse ohne leere Verzeichnisse
/MIR         Quelle auf Ziel spiegeln
/COPY        Zu kopierende Dateiinformationen bestimmen
/COPYALL     Alle Dateiinformationen kopieren
/DCOPY       Zu kopierende Verzeichnisinformationen bestimmen
/MT:n        Multi-Threading
/Z           Restartable Mode
/B           Backup Mode
/ZB          Restartable Mode mit Fallback auf Backup Mode
/R:n         Anzahl der Wiederholungsversuche
/W:n         Wartezeit zwischen Wiederholungsversuchen
/L           Testlauf ohne tatsächliche Änderungen
/LOG         Ausgabe in Logdatei schreiben
/LOG+        Ausgabe an Logdatei anhängen
/TEE         Ausgabe zusätzlich im Terminal anzeigen
```

---
## Merksatz

> **Robocopy ist ein robustes Windows-Kommandozeilentool zum Kopieren und Synchronisieren großer Datei- und Verzeichnisstrukturen und ist insbesondere für Fileserver- und Azure-Files-Migrationen relevant.**
