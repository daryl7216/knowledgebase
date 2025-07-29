## HostedServiceStarted Table: Hosted Service Start Event

### 1. Table Name
**HostedServiceStarted**

---

### 2. Table Usage
This event is generated when a hosted service, which is a service that runs inside a generic `svchost.exe` process, is started by the Service Control Manager (SCM). Attackers often create malicious services that are loaded by the legitimate `svchost.exe` to establish persistence and conceal their activities. Monitoring this event is crucial for detecting such persistence techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ServiceDisplayName | The display name of the hosted service that was started. | Windows Only |
| ImageFileName | The full path to the service's DLL file that is loaded into the `svchost.exe` process. | Windows Only |
| UserName | The user account under which the service is configured to run. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to start the service. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ServiceDisplayName": "Malicious Windows Update Service",
  "ImageFileName": "C:\\Windows\\Temp\\malware.dll",
  "UserName": "LocalSystem",
  "RpcClientProcessId": "784"
}
```
---

### 5. Example Queries
```xql
// Find all hosted service start events
event_simpleName=HostedServiceStarted

// Hunt for hosted services loading DLLs from unusual locations
event_simpleName=HostedServiceStarted ImageFileName IN ("*\\Temp\\*", "*\\Users\\Public\\*")

// Find hosted services with suspicious display names
event_simpleName=HostedServiceStarted ServiceDisplayName IN ("*malware*", "*backdoor*")

// Investigate which process is responsible for starting a specific service
event_simpleName=HostedServiceStarted ServiceDisplayName="SuspiciousSvc" | table RpcClientProcessId, ImageFileName
```
---