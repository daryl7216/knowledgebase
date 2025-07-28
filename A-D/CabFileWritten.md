## CabFileWritten Table: CAB File Written Event

### 1. Table Name
**CabFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Microsoft Cabinet (CAB) file. CAB files are commonly used for software distribution and compression. Monitoring their creation can help identify legitimate software installations, as well as potential malware packaging or data exfiltration attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file that was written. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "installer.cab",
  "TargetFileName": "C:\\Windows\\Temp\\installer.cab",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-18"
}
```
---
### 5. Example Queries
```xql
// Find all ARC files written with a specific target file name
event_simpleName=ArcFileWritten TargetFileName="*sensitive_data.zip*"

// Find ARC files written to a network drive (Windows only)
event_simpleName=ArcFileWritten IsOnNetwork=true

// Find ARC files written to a removable disk
event_simpleName=ArcFileWritten IsOnRemovableDisk=true

// Investigate ARC file writes by a specific file operator SID (Windows only)
event_simpleName=ArcFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find ARC files based on their context base file name
event_simpleName=ArcFileWritten ContextBaseFileName="archive.rar"
```
---
