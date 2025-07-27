## CriticalFileAccessed Table: Critical File Accessed Event

### 1. Table Name
**CriticalFileAccessed**

---

### 2. Table Usage
A critical file was accessed.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                 | Platforms Affected         |
|--------------------|-----------------------------------------------------------------------------|----------------------------|
| ContextTimeStamp   | System time of event creation.                                    | macOS, Linux, Falcon Container |
| ContextProcessId   | UPID of process originating this event.                           | macOS, Linux, Falcon Container |
| ContextThreadId    | UTID of thread originating this event                             | macOS, Linux, Falcon Container |
| TreeId             | If this event is part of a detection tree, the tree ID it is part of. | macOS, Linux, Falcon Container |
| TargetFileName     | The resulting file name that was downloaded                       | macOS, Linux, Falcon Container |
| UID                | Unix User Identifier.                                             | macOS, Linux, Falcon Container |
| GID                | Unix Group Identifier.                                            | macOS, Linux, Falcon Container |
| UnixMode           | Describes the traditional Unix permissions of a file system entry (eg. 0755). | macOS, Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "TargetFileName": "/etc/passwd",
  "UID": 0,
  "GID": 0,
  "UnixMode": "0644"
}
```
---
### 5. Example Queries
```xql
// Find all critical file access events for a specific target file name
event_simpleName=CriticalFileAccessed TargetFileName="/etc/shadow"

// Find critical file access events by a specific UID
event_simpleName=CriticalFileAccessed UID=0

// Find critical file access events with a specific Unix mode
event_simpleName=CriticalFileAccessed UnixMode="0777"

// Investigate critical file access events part of a specific detection tree
event_simpleName=CriticalFileAccessed TreeId="abc-xyz"
```
---