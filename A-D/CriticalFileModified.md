## CriticalFileModified Table: Critical File Modified Event

### 1. Table Name
**CriticalFileModified**

---

### 2. Table Usage
A critical file was modified.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected         |
|----------------|-------------------------------------------|----------------------------|
| TargetFileName | The resulting file name that was downloaded | macOS, Linux, Falcon Container |
| UID            | Unix User Identifier.                     | macOS, Linux, Falcon Container |
| GID            | Unix Group Identifier.                    | macOS, Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "TargetFileName": "/etc/hosts",
  "UID": 0,
  "GID": 0,
  "UnixMode": "0644"
}
```
---

### 5. Example Queries
```xql
// Find all critical file modification events for a specific target file name
event_simpleName=CriticalFileModified TargetFileName="/etc/resolv.conf"

// Find critical file modification events by a specific UID
event_simpleName=CriticalFileModified UID=0

// Find critical file modification events by a specific GID
event_simpleName=CriticalFileModified GID=0
```
---
