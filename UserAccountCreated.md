## UserAccountCreated Table: User Account Creation Event

### 1. Table Name
**UserAccountCreated**

---

### 2. Table Usage
This event is generated when a new user account is created on a Windows host. Monitoring this event is crucial for security, as unexpected account creation can be an indicator of malicious activity, such as an attacker establishing persistence on a system. The event data can help identify the initial process (e.g., command-line tool, custom utility) or remote host that initiated the action.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the new user account that was created. | Windows Only |
| RpcClientProcessId | The Process ID (PID) corresponding to the RPC client of this request. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "TempAdmin01",
  "RpcClientProcessId": "784"
}
```

---

### 5. Example Queries

```xql// Find all user account creation events
event_simpleName=UserAccountCreated

// Investigate the creation of a specific user account
event_simpleName=UserAccountCreated UserName="BackupSvc"

// Find all user accounts created by a specific parent process ID
event_simpleName=UserAccountCreated RpcClientProcessId="1337"
```
---
