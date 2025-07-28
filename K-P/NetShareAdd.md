## Crowdstrike Table: NetShareAdd

### 1. Table Name

**NetShareAdd**

---

### 2. Table Usage

This table is generated when a network share is added or modified on a host. This event is critical for monitoring system configuration changes, detecting potential privilege escalation attempts, and identifying suspicious lateral movement activities within a network environment on Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ShareData | The data associated with the network share. | Windows |
| ShareName | The name of the network share. | Windows |
| SharePath | The local path that is being shared. | Windows |
| UserName | The user name associated with the event. | Windows |
| RpcClientProcessId | The ProcessId corresponding to the RPC client of this request. | Windows |

---

### 4. Sample Log Example

```json
{
  "ShareData": "Some share data",
  "ShareName": "MySharedFolder",
  "SharePath": "C:\\Shared",
  "UserName": "Administrator",
  "RpcClientProcessId": 1234
}
```

---

### 5. Example Queries

```xql
// Find all network share additions or modifications
event_simpleName=NetShareAdd

// Find network share additions with a specific share name
event_simpleName=NetShareAdd ShareName="*SensitiveDataShare*"

// Find network share additions from a specific user
event_simpleName=NetShareAdd UserName="*baduser*"

// Find network share additions for a specific shared path
event_simpleName=NetShareAdd SharePath="*C:\\Windows\\System32*"

// Find network share additions originating from a specific RPC client process ID
event_simpleName=NetShareAdd RpcClientProcessId=5678
```

---
