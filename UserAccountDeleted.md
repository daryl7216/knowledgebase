## UserAccountDeleted Table: User Account Deletion Event

### 1. Table Name
**UserAccountDeleted**

---

### 2. Table Usage
This event is generated when a user account is deleted on a Windows host. It is critical for security auditing, as it can indicate malicious activity, such as an attacker trying to cover their tracks. The event can help identify the process or remote host that initiated the deletion.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user account that was deleted. | Windows, Forensics |
| RpcClientProcessId | The Process ID (PID) of the RPC client that initiated the account deletion request. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "TempAdmin",
  "RpcClientProcessId": "784"
}
```
---
### 5. Example Queries
```xql
// Find all user account deletion events
event_simpleName=UserAccountDeleted

// Investigate which process was responsible for deleting a specific user account
event_simpleName=UserAccountDeleted UserName="service-acct"

// Find all user accounts deleted by a specific process ID
event_simpleName=UserAccountDeleted RpcClientProcessId="1337"
```
