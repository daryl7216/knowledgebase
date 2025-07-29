## GzipFileWritten Table: Gzip File Written Event

### 1. Table Name
**GzipFileWritten**

---

### 2. Table Usage
This table records events when a file with a Gzip image header is written to the system. Gzip is a common compression format, and monitoring these events can help detect data compression activities, exfiltration attempts, or malware packaging across various operating systems.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/home/user/downloads/archive.gz",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all Gzip files written with a specific target file name
event_simpleName=GzipFileWritten TargetFileName="*backup_data.gz*"

// Find Gzip files written to a temporary directory
event_simpleName=GzipFileWritten TargetFileName="*\\temp\\*.gz"

// Identify recently written Gzip archives
event_simpleName=GzipFileWritten TargetFileName="*.gz"
```
---