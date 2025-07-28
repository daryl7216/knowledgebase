## BlfFileWritten Table: BLF File Written Event

### 1. Table Name
**BlfFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a BLF (Binary Log File) file. These files are often used for logging system events or driver information. Monitoring this event can help identify suspicious logging activities, data manipulation, or system component compromise.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                                | All                |
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                              | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "sys_log.blf",
  "TargetFileName": "C:\\Windows\\System32\\config\\sys_log.blf",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-18"
}
```
---
### 5. Example Queries

```xql
// Find all BLF files written to a network drive (Windows only)
event_simpleName=BlfFileWritten IsOnNetwork=true

// Find BLF files written to a removable disk
event_simpleName=BlfFileWritten IsOnRemovableDisk=true

// Investigate BLF file writes by a specific file operator SID (Windows only)
event_simpleName=BlfFileWritten FileOperatorSid="S-1-5-18"

// Find BLF files with a specific target file name
event_simpleName=BlfFileWritten TargetFileName="*temp_update.blf*"

// Find BLF files based on their context base file name
event_simpleName=BlfFileWritten ContextBaseFileName="diagnostic.blf"
```
---
