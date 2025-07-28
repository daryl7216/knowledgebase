## ExecutableDeleted Table: Executable Deleted Event

### 1. Table Name
**ExecutableDeleted**

---

### 2. Table Usage
This table records events when an executable file is deleted from a host. This event is critical for identifying potential malicious activity, such as malware self-deletion to evade detection, or legitimate software uninstallation.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was deleted. | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\Temp\\malicious_payload.exe",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all executable deleted events
event_simpleName=ExecutableDeleted

// Find executable deleted events for a specific file name
event_simpleName=ExecutableDeleted TargetFileName="*temp_tool.exe*"

// Identify recently deleted executables with common suspicious names
event_simpleName=ExecutableDeleted TargetFileName="*powershell.exe*" OR TargetFileName="*mimikatz.exe*"
```
---