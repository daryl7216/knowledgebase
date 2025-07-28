## ScheduledTaskRegistered Table: Scheduled Task Creation Event

### 1. Table Name
**ScheduledTaskRegistered**

---

### 2. Table Usage
This event is generated when a new scheduled task is registered on a Windows host, either locally or remotely. Attackers frequently use scheduled tasks to establish persistence, run malicious code at specific times, or execute commands with elevated privileges. Monitoring this event is critical for detecting these persistence and execution techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TaskName | The name of the scheduled task. | Windows Only |
| TaskAuthor | The author of the scheduled task. | Windows Only |
| TaskExecCommand | The command that the task is configured to execute. | Windows Only |
| TaskXml | The full XML definition of the scheduled task, which contains all configuration details. | Windows Only |
| UserName | The user account that registered the scheduled task. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to register the task (e.g., the PID for `svchost.exe` hosting the Task Scheduler service). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TaskName": "\\Microsoft\\Windows\\EvilTask",
  "TaskAuthor": "CORP\\Admin",
  "TaskExecCommand": "C:\\Windows\\Temp\\payload.exe",
  "TaskXml": "<?xml version=\"1.0\" encoding=\"UTF-16\"?><Task version=\"1.2\" xmlns=\"[http://schemas.microsoft.com/windows/2004/02/mit/task](http://schemas.microsoft.com/windows/2004/02/mit/task)\">...</Task>",
  "UserName": "CORP\\Admin",
  "RpcClientProcessId": "1234"
}
```

---
### 5. Example Queries
```xql
// Find all newly registered scheduled tasks
event_simpleName=ScheduledTaskRegistered

// Hunt for scheduled tasks that execute a command from a suspicious location
event_simpleName=ScheduledTaskRegistered TaskExecCommand="*\\Temp\\*" OR TaskExecCommand="*\\Users\\Public\\*"

// Find tasks created by a specific author
event_simpleName=ScheduledTaskRegistered TaskAuthor="*attacker*"

// Search for tasks that execute PowerShell commands
event_simpleName=ScheduledTaskRegistered TaskExecCommand="*powershell.exe*"

// Investigate the full XML for tasks that run with SYSTEM privileges
event_simpleName=ScheduledTaskRegistered TaskXml="*<UserId>S-1-5-18</UserId>*"
```
---