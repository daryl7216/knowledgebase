## FirewallSetRule Table: Firewall Rule Set Event

### 1. Table Name
**FirewallSetRule**

---

### 2. Table Usage
This table records events when a firewall rule is created or modified on a Windows host. This is crucial for monitoring network security configurations, detecting unauthorized changes that could expose systems, or identifying the establishment of persistence by malicious actors.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                                  | Platforms Affected |
|--------------------|----------------------------------------------------------------------------------------------|--------------------|
| FirewallRule       | The full definition of the firewall rule that was created or modified.                       | Windows Only       |
| UserName           | The user account under which the process that set the firewall rule is running.              | Windows Only       |
| RpcClientProcessId | The ProcessId corresponding to the RPC client that initiated the firewall rule modification. | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "FirewallRule": "Name: Allow Inbound RDP, Direction: In, Action: Allow, Protocol: TCP, LocalPort: 3389",
  "UserName": "NT AUTHORITY\\SYSTEM",
  "RpcClientProcessId": "884"
}
```
### 5. Example Queries
```xql
// Find all firewall rule set events
event_simpleName=FirewallSetRule

// Find firewall rule set events that allow inbound RDP connections
event_simpleName=FirewallSetRule FirewallRule="*Allow Inbound RDP*"

// Investigate firewall rule changes made by a specific user
event_simpleName=FirewallSetRule UserName="*Administrator*"

// Find firewall rule set events initiated by a suspicious RPC client process ID
event_simpleName=FirewallSetRule RpcClientProcessId="1234"
```
---