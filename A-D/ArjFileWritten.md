|## ArjFileWritten Table: File Written Event

### 1. Table Name
**ArjFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ARJ (Archive) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\archive.arj",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "archive.arj"
}
```
---

### 5. Example Queries
```xql
// Find all ARJ files written with a specific target file name
event_simpleName=ArjFileWritten TargetFileName="*invoice.arj*"

// Find ARJ files written to a network drive (Windows only)
event_simpleName=ArjFileWritten IsOnNetwork=true

// Find ARJ files written to a removable disk
event_simpleName=ArjFileWritten IsOnRemovableDisk=true

// Investigate ARJ file writes by a specific file operator SID (Windows only)
event_simpleName=ArjFileWritten FileOperatorSid="S-1-5-21-98765-43210-*"

// Find ARJ files based on their context base file name
event_simpleName=ArjFileWritten ContextBaseFileName="backup.arj"
```

---
