## TarFileWritten Table: File Written Event

### 1. Table Name
**TarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a TAR (Tape Archive) file. Attackers often use TAR files to aggregate stolen data before exfiltration. Monitoring the creation of these files is useful for investigating data staging, potential data theft, and malware deployment.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/tmp/staged_data.tar",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "staged_data.tar"
}
```
---
### 5. Example Queries
```xql
// Find all TAR files written to disk
event_simpleName=TarFileWritten TargetFileName="*.tar"

// Find TAR files written to a network drive (Windows only)
event_simpleName=TarFileWritten IsOnNetwork=true

// Find TAR files written to a removable disk (Windows only)
event_simpleName=TarFileWritten IsOnRemovableDisk=true

// Investigate TAR file writes by a specific file operator SID (Windows only)
event_simpleName=TarFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find TAR files based on their context base file name, which might indicate staging
event_simpleName=TarFileWritten ContextBaseFileName="*backup*.tar"
```
---
