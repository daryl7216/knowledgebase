## BITSJobCreated Table: BITS Job Created Event

### 1. Table Name
**BITSJobCreated**

---

### 2. Table Usage
This table records events generated when a Background Intelligent Transfer Service (BITS) download job is created, typically indicating a file transfer operation initiated by an application or system component. This is useful for tracking file downloads, potential malware delivery, or data exfiltration attempts.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                                  | Platforms Affected |
|--------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName     | The resulting file name that was downloaded.                                         | Windows Only       |
| UserName           | The user name associated with the process creating the BITS job.                     | Windows Only       |
| RpcClientProcessId | The ProcessId corresponding to the RPC client of this request.                     | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\Public\\Downloads\\malware.exe",
  "UserName": "JOHN_DOE\\User",
  "RpcClientProcessId": "7890"
}
```

---
### 5. Example Queries
```xql
// Find all BITS job creation events that downloaded an executable file
event_simpleName=BITSJobCreated TargetFileName="*.exe*"

// Investigate BITS jobs created by a specific user
event_simpleName=BITSJobCreated UserName="JOHN_DOE\\User"

// Find BITS jobs initiated by a specific RPC client process ID
event_simpleName=BITSJobCreated RpcClientProcessId="7890"

// Combine queries to find suspicious BITS downloads by a specific user
event_simpleName=BITSJobCreated UserName="SUSPICIOUS_USER" TargetFileName="*temp\\*.dll*"
```
---
