## ProcessSelfDeleted Table: Process Self-Deletion Event

### 1. Table Name
**ProcessSelfDeleted**

---

### 2. Table Usage
This event is generated when a process deletes its own executable file from disk. This is a common defense evasion technique used by malware to remove its tracks after it has successfully executed its payload, making forensic analysis more difficult. This event helps detect such clean-up activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file that was deleted by the process. | Windows |
| SourceProcessId | The process ID of the parent or creating process. In some self-deletion scenarios, this may be the same as the `TargetProcessId`. | Windows |
| TargetProcessId | The unique ID of the process that is deleting its own primary module. | Windows |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious_installer.exe",
  "SourceProcessId": "1234",
  "TargetProcessId": "5678"
}
```

---

### 5. Example Queries

```xql
// Find all instances of a process self-deleting
event_simpleName=ProcessSelfDeleted

// Hunt for a specific executable that is known to self-delete
event_simpleName=ProcessSelfDeleted ImageFileName="*\\payload.exe"

// Find processes running from a temp directory that self-delete
event_simpleName=ProcessSelfDeleted ImageFileName="*\\Temp\\*"

// Investigate the relationship between the process that initiated the deletion and the process that was deleted
event_simpleName=ProcessSelfDeleted | table SourceProcessId, TargetProcessId, ImageFileName
```
---
