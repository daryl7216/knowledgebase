## ArcFileWritten Table: File Written Event

### 1. Table Name
**ArcFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ARC (Archive) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was written.                                          | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                       | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                                 | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\report.zip",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "report.zip"
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
