## ProcessExecOnRDPFile Table: RDP File Execution Event

### 1. Table Name
**ProcessExecOnRDPFile**

---

### 2. Table Usage
This event is generated when a PE (Portable Executable) file, which was written to the host via a Remote Desktop Protocol (RDP) session, is executed. Attackers who have gained RDP access often transfer their tools and malware to the victim's machine and then execute them. Monitoring this event is crucial for detecting malicious activity and lateral movement originating from RDP sessions.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the process that was executed from the RDP session. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "6212"
}
```

---

### 5. Example Queries
```xql
// Find all instances of a PE file being executed from an RDP session
event_simpleName=ProcessExecOnRDPFile

// Investigate a specific process ID that was executed via RDP
event_simpleName=ProcessExecOnRDPFile TargetProcessId="6212"
```
---