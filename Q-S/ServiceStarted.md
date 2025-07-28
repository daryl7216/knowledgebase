## ServiceStarted Table: Service Start Event

### 1. Table Name
**ServiceStarted**

---

### 2. Table Usage
This event is generated when a standalone service is started by the Windows Service Control Manager (SCM). Attackers often use services to establish persistence or run malicious code with elevated privileges. This event provides details about the new service, including its executable path and command line, as well as information about the process that initiated the start request.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the main executable for the new service. | Windows Only |
| ServiceDisplayName | The display name of the service being started. | Windows Only |
| UserName | The user account under which the service is configured to run. | Windows Only |
| CommandLine | The command line used to start the service's process. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the RPC client that requested the service to start (e.g., the PID for `services.exe`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\evil.exe",
  "ServiceDisplayName": "Malicious Updater Service",
  "UserName": "LocalSystem",
  "CommandLine": "C:\\Windows\\System32\\evil.exe -run",
  "RpcClientProcessId": "784"
}
```
---

### 5. Example Queries

```xql
// Find all service start events
event_simpleName=ServiceStarted

// Hunt for services running from unusual locations, like Temp directories
event_simpleName=ServiceStarted ImageFileName="*\\Temp\\*"

// Find services with suspicious display names
event_simpleName=ServiceStarted ServiceDisplayName="*malware*" OR ServiceDisplayName="*backdoor*"

// Find services started with PowerShell in the command line
event_simpleName=ServiceStarted CommandLine="*powershell.exe*"

// Investigate which process requested a specific service to start
event_simpleName=ServiceStarted ServiceDisplayName="SuspiciousSvc" | table RpcClientProcessId
```
---
