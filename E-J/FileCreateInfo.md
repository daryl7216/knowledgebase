## FileCreateInfo Table: File Creation Event

### 1. Table Name
**FileCreateInfo**

---

### 2. Table Usage
This table records events generated when a new file is created by a process on the endpoint. This event is fundamental for tracking file system activity, identifying new file drops (e.g., by malware), or monitoring legitimate application behavior.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was created. | All                |
| UID            | Unix User Identifier.                     | macOS, Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/home/user/new_document.txt",
  "UID": "1001"
}
```
---
### 5. Example Queries
```xql
// Find all file creation events for a specific target file name
event_simpleName=FileCreateInfo TargetFileName="*important_config.ini*"

// Find file creation events by a specific Unix User ID (UID)
event_simpleName=FileCreateInfo UID="1001"

// Identify recently created files in a suspicious directory by a specific UID
event_simpleName=FileCreateInfo TargetFileName="/tmp/*" UID="0"
```
---