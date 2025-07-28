## ProcessExecOnSMBFile Table: Remote Execution from SMB Share Event

### 1. Table Name
**ProcessExecOnSMBFile**

---

### 2. Table Usage
This event is generated when a process executes a PE (Portable Executable) file that was written via SMB (Server Message Block). This is a very common technique used by attackers for lateral movement, allowing them to run tools and malware on remote systems without first writing them to the local disk. Monitoring this event is critical for detecting remote code execution and lateral movement attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user account that executed the file from the SMB share. | Windows Only |
| UserSid | The Security Identifier (SID) of the user who performed the execution. | Windows Only |
| LogonDomain | The domain of the user account that initiated the execution. | Windows Only |
| RemoteAddressIP4 | The source IPv4 address from which the SMB connection originated, indicating the machine where the execution was initiated from. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "jsmith",
  "UserSid": "S-1-5-21-1234567890-123456789-1234567890-1001",
  "LogonDomain": "CORP",
  "RemoteAddressIP4": "10.0.5.25"
}
```

---

### 5. Example Queries

```xql
// Find all instances of a process executing from an SMB share
event_simpleName=ProcessExecOnSMBFile

// Hunt for executions originating from a specific remote IP address
event_simpleName=ProcessExecOnSMBFile RemoteAddressIP4="192.168.1.100"

// Find all remote executions performed by a specific user
event_simpleName=ProcessExecOnSMBFile UserName="*admin*"

// Investigate executions associated with a specific user SID
event_simpleName=ProcessExecOnSMBFile UserSid="S-1-5-21-1234567890-123456789-1234567890-500"

// Look for remote executions within a specific domain
event_simpleName=ProcessExecOnSMBFile LogonDomain="FINANCE"
```
---