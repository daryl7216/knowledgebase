## GenericFileWritten Table: Generic File Written Event

### 1. Table Name
**GenericFileWritten**

---

### 2. Table Usage
This table records events when a process finishes writing to a file that does not match a more specific `*FileWritten` event. These events are currently generated for removable disks only. This is useful for capturing write activity to miscellaneous file types, especially on removable media, which can be an indicator of data transfer or suspicious file drops.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                                          | All                |
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                              | Windows Only       |
| UserName            | The user account associated with the process that wrote the file.                            | All                |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "report.dat",
  "TargetFileName": "D:\\backups\\report.dat",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": true,
  "FileOperatorSid": "S-1-5-21-12345-67890-...",
  "UserName": "johndoe"
}
```
---

### 5. Example Queries
```xql
// Find all generic file write events to removable disks
event_simpleName=GenericFileWritten IsOnRemovableDisk=true

// Find generic file writes where the target file name indicates a potential sensitive file
event_simpleName=GenericFileWritten TargetFileName="*passwords.txt*"

// Investigate generic file writes by a specific user to removable media
event_simpleName=GenericFileWritten UserName="malicious_user" IsOnRemovableDisk=true

// Find generic file writes on a network drive (Windows only)
event_simpleName=GenericFileWritten IsOnNetwork=true

// Find generic file writes by a specific file operator SID (Windows only)
event_simpleName=GenericFileWritten FileOperatorSid="S-1-5-18"
```
---