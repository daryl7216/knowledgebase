## EseFileWritten Table: ESE File Written Event

### 1. Table Name
**EseFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ESE (Extensible Storage Engine) database file. ESE databases are used by many Microsoft applications and services (e.g., Active Directory, Exchange, Windows Update). Monitoring these events can indicate legitimate database operations, but also potential data corruption, unauthorized access, or malware activity targeting system databases.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                          | All                |
| TargetFileName      | The resulting file name that was downloaded.                                 | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "esent.edb",
  "TargetFileName": "C:\\Windows\\System32\\edb.log",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-18"
}
```
---
### 5. Example Queries
```xql
// Find all ESE files written to a network drive (Windows only)
event_simpleName=EseFileWritten IsOnNetwork=true

// Find ESE files written to a removable disk
event_simpleName=EseFileWritten IsOnRemovableDisk=true

// Investigate ESE file writes by a specific file operator SID (Windows only)
event_simpleName=EseFileWritten FileOperatorSid="S-1-5-18"

// Find ESE files with a specific target file name
event_simpleName=EseFileWritten TargetFileName="*temp_database.edb*"

// Find ESE files based on their context base file name
event_simpleName=EseFileWritten ContextBaseFileName="app_data.jrs"
```
---