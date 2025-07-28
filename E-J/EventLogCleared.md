## EventLogCleared Table: Event Log Cleared Event

### 1. Table Name
**EventLogCleared**

---

### 2. Table Usage
This table records events generated when a Windows event log is cleared. Clearing event logs can be a tactic used by attackers to remove traces of their activity. Monitoring these events is critical for detecting potential malicious activity and maintaining forensic integrity.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                                  | Platforms Affected |
|--------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName     | The resulting file name of the event log that was cleared.                           | Windows Only       |
| UserName           | The user name associated with the process that cleared the event log.                | Windows Only       |
| RpcClientProcessId | The ProcessId corresponding to the RPC client that initiated the event log clear operation. | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\System32\\Winevt\\Logs\\Security.evtx",
  "UserName": "SYSTEM",
  "RpcClientProcessId": "4"
}
```
---
### 5. Example Queries
```xql
// Find all event log cleared events
event_simpleName=EventLogCleared

// Find event log cleared events for a specific log file (e.g., Security log)
event_simpleName=EventLogCleared TargetFileName="*Security.evtx*"

// Investigate event log cleared events initiated by a specific user
event_simpleName=EventLogCleared UserName="*Administrator*"

// Find event log cleared events by a specific RPC client process ID
event_simpleName=EventLogCleared RpcClientProcessId="4"
```
---