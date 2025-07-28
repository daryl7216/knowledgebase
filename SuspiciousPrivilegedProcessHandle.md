## SuspiciousPrivilegedProcessHandle Table: Privileged Process Handle Event

### 1. Table Name
**SuspiciousPrivilegedProcessHandle**

---

### 2. Table Usage
This event is generated when a suspicious handle is opened to a privileged process. This technique is often used by credential theft tools (like Mimikatz) to access the memory of sensitive processes like LSASS. Monitoring this event is critical for detecting attempts to steal credentials.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the privileged process to which the handle was opened. | Windows Only |
| HandleCreated | The handle that was created to the target process. | Windows Only |
| SuspiciousHandleOpenReason | The reason the handle open was flagged as suspicious. Values include `1` (Opened by LSASS) and `2` (Opened by a low-integrity process). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "684",
  "HandleCreated": "2540",
  "SuspiciousHandleOpenReason": "1"
}
```
---
### 5. Example Queries
```xql
// Find all suspicious handle open events to privileged processes
event_simpleName=SuspiciousPrivilegedProcessHandle

// Find handles opened to a specific privileged process, such as LSASS
event_simpleName=SuspiciousPrivilegedProcessHandle TargetProcessId="684"

// Find handles opened by a low-integrity process
event_simpleName=SuspiciousPrivilegedProcessHandle SuspiciousHandleOpenReason=2
```
---