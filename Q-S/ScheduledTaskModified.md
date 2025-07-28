## ScheduledTaskModified Table: Scheduled Task Modification Event

### 1. Table Name
**ScheduledTaskModified**

---

### 2. Table Usage
This event is generated when an existing scheduled task is modified on a Windows host. Attackers often hijack legitimate scheduled tasks to establish persistence or execute malicious code with the privileges of the original task. Monitoring these modifications is crucial for detecting attempts to abuse the Task Scheduler for malicious purposes.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TaskName | The name of the scheduled task that was modified. | Windows Only |
| TaskAuthor | The author of the scheduled task. | Windows Only |
| TaskExecCommand | The new command that the task is configured to execute after modification. | Windows Only |
| TaskXml | The full XML definition of the modified scheduled task, containing all new configuration details. | Windows Only |
| UserName | The user account that performed the modification of the task. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to modify the task. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TaskName": "\\Microsoft\\Windows\\UpdateOrchestrator\\UpdateAssistant",
  "TaskAuthor": "SYSTEM",
  "TaskExecCommand": "C:\\Windows\\Temp\\malicious.exe",
  "TaskXml": "<?xml version=\"1.0\" encoding=\"UTF-16\"?><Task version=\"1.3\" xmlns=\"[http://schemas.microsoft.com/windows/2004/02/mit/task](http://schemas.microsoft.com/windows/2004/02/mit/task)\">...</Task>",
  "UserName": "CORP\\Admin",
  "RpcClientProcessId": "1234"
}
```

### 5. Example Queries
```xql
// Find all scheduled task modification events
event_simpleName=ScheduledTaskModified

// Hunt for legitimate tasks that have been modified to run executables from suspicious locations
event_simpleName=ScheduledTaskModified TaskExecCommand="*\\Temp\\*" OR TaskExecCommand="*\\Users\\Public\\*"

// Find tasks modified to execute PowerShell commands
event_simpleName=ScheduledTaskModified TaskExecCommand="*powershell.exe*"

// Investigate modifications made by a specific user
event_simpleName=ScheduledTaskModified UserName="*svc_account*"

// Search the raw Task XML for suspicious arguments or configurations
event_simpleName=ScheduledTaskModified TaskXml="*Invoke-Expression*" OR TaskXml="*DownloadString*"
```
---