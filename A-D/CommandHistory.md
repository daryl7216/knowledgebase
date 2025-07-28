## CommandHistory Table: Command History Event

### 1. Table Name
**CommandHistory**

---

### 2. Table Usage
This event includes the full command history associated with one of the consoles in a process that has recently terminated (identified by the TargetProcessId). Note that a given UPID may be associated with multiple consoles (identified by ApplicationName).

---

### 3. Important Parameter Description

| Parameter       | Description                                                                                                                                                                                                                                                                                                              |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus. For example, the TargetProcessId of a process that performed thread injection in an InjectedThread event |
| CommandHistory  | The full command history associated with one of the consoles in a process.                                                                                                                                                                                                                                     |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 12345,
  "ApplicationName": "cmd.exe",
  "CommandCount": 5,
  "CommandCountMax": 50,
  "FirstCommand": "cd C:\\Users\\user\\Documents",
  "LastAdded": "dir",
  "LastDisplayed": "dir",
  "CommandHistory": [
    "cd C:\\Users\\user\\Documents",
    "mkdir NewFolder",
    "echo Hello > NewFolder\\file.txt",
    "type NewFolder\\file.txt",
    "dir"
  ],
  "CommandSequence": 5,
  "UserSid": "S-1-5-21-...",
  "LogonType": 2,
  "LogonTime": "2023-10-27T10:00:00Z",
  "AuthenticationId": 999
}
```
---
### 5. Example Queries
```xql
// Find command history for a specific target process ID
event_simpleName=CommandHistory TargetProcessId=12345

// Find command history containing a specific command
event_simpleName=CommandHistory CommandHistory="*powershell*"
```
---
