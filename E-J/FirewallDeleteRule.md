## FirewallDeleteRule Table: Firewall Rule Deletion Event

### 1. Table Name
**FirewallDeleteRule**

---

### 2. Table Usage
A firewall rule has been deleted, such as removing a rule preventing inbound RDP connections. The data will indicate the initial process (command-line tool, custom utility, or GUI application) or remote address/hostname that resulted in this action. This event is useful for detecting unauthorized network configuration changes.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FirewallRuleId | The unique identifier of the firewall rule that was deleted. | Windows Only |
| UserName | The name of the user who initiated the firewall rule deletion. | Windows Only |
| RpcClientProcessId | The ProcessId corresponding to the RPC client that requested the deletion. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "FirewallRuleId": "{C5DD2D2C-7901-4A5D-B5F5-1D8C8B7E3F0F}",
  "RpcNestingLevel": 0,
  "InterfaceGuid": "0x81bb7a364498f135ad3298f038001003",
  "InterfaceVersion": 1.0,
  "RpcOpNum": 31,
  "RpcClientProcessId": 1234,
  "RpcClientThreadId": 5678,
  "ContextProcessId": 9876,
  "RemoteAddressIP4": "192.168.1.100",
  "RemoteAddressIP6": "::1",
  "AuthenticationId": 999,
  "TokenType": 1,
  "UserName": "Administrator"
}
```
---

### 5. Example Queries
```xql
// Find all firewall rule deletion events for a specific rule ID
event_simpleName=FirewallDeleteRule FirewallRuleId="{C5DD2D2C-7901-4A5D-B5F5-1D8C8B7E3F0F}"

// Find firewall rule deletion events initiated by a specific user
event_simpleName=FirewallDeleteRule UserName="Administrator"

// Find firewall rule deletion events requested by a specific RPC client process ID
event_simpleName=FirewallDeleteRule RpcClientProcessId=1234
```
---