## CriticalEnvironmentVariableChanged Table: Critical Environment Variable Changed Event

### 1. Table Name
**CriticalEnvironmentVariableChanged**

---

### 2. Table Usage
A process set a critical environment variable.

---

### 3. Important Parameter Description

| Parameter                | Description                                       | Platforms Affected |
|--------------------------|---------------------------------------------------|--------------------|
| EnvironmentVariableName  | The name of the environment variable that was set. | Linux, Falcon Container |
| EnvironmentVariableValue | The value to which the environment variable was set. | Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "EnvironmentVariableName": "PATH",
  "EnvironmentVariableValue": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/new/custom/path"
}
```
---
### 5. Example Queries
```xql
// Find all critical environment variable changes for a specific variable name
event_simpleName=CriticalEnvironmentVariableChanged EnvironmentVariableName="PATH"

// Find critical environment variable changes where the value contains a specific string
event_simpleName=CriticalEnvironmentVariableChanged EnvironmentVariableValue="*new/custom/path*"
```
---
