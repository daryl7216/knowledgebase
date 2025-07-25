## SentinelOne Table: Process Creation

### 1. Table Name
**Process Creation**

---

### 2. Table Usage
This table records process creation events, showing the relationships between source and target processes on an endpoint. It is useful for investigations into process tree relationships, privilege escalation, lateral movement, malware execution, and other endpoint activities.

---

### 3. Important Parameter Description

| Parameter                     | Description                                                                                  |
|-------------------------------|----------------------------------------------------------------------------------------------|
| src.process.image.path        | Full path of the source process (the parent process).                                        |
| src.process.cmdline           | The command line used to start the source process.                                           |
| src.process.user              | User account under which the source process is running.                                      |
| src.process.signedStatus      | Signature status of the source process (e.g., signed, unsigned, verified).                   |
| tgt.process.image.path        | Full path of the target process (the process being created).                                 |
| tgt.process.cmdline           | The command line used to start the target process.                                           |
| tgt.process.signedStatus      | Signature status of the target process (e.g., signed, unsigned, verified).                   |
| tgt.process.uid               | Process Unique ID of the target process.                                                     |
| src.process.uid               | Process Unique ID of the source process.                                                     |
| tgt.process.image.sha1        | SHA1 hash of the target process.                                                             |
| tgt.process.image.sha256      | SHA256 hash of the target process.                                                           |
| tgt.process.image.md5         | MD5 hash of the target process.                                                              |
| src.process.image.sha1        | SHA1 hash of the source process.                                                             |
| src.process.image.sha256      | SHA256 hash of the source process.                                                           |
| src.process.image.md5         | MD5 hash of the source process.                                                              |
| agent.uuid                    | Unique ID of the agent.                                                                      |

---

### 4. Sample Log Example

```json
{
  "timestamp": "18:32:29.470",
  "event.type": "Process Creation",
  "event.category": "process",
  "src.process.image.path": "C:\\\\Program Files\\\\Acme\\\\acme-infra\\\\acme-infra.exe",
  "src.process.cmdline": "C:\\\\Program Files (x86)\\\\Microsoft\\\\important_stuff\\\\stuff.EXE\\\\",
  "src.process.user": "NT AUTHORITY\\\\SYSTEM",
  "src.process.signedStatus": "signed",
  "tgt.process.image.path": "C:\\\\Program Files\\\\Acme\\\\acme-infra\\\\acme-integrations\\\\nr-winpkg.exe",
  "tgt.process.cmdline": "./nr-winpkg",
  "tgt.process.user": "NT AUTHORITY\\\\SYSTEM",
  "tgt.process.signedStatus": "signed",
  ...
}
```

---

### 5. Example Queries

```xql
// Find all processes created by a specific process UID
event.type = "Process Creation" and src.process.uid = "123456789" and agent.uuid = "agent-123"

// Find all child processes of a suspicious executable
event.type = "Process Creation" and src.process.image.path contains "suspicious.exe" and agent.uuid = "agent-123"

// Find the parent process of a suspicious executable
event.type = "Process Creation" and tgt.process.uid = "123456789" and agent.uuid = "agent-123"

// Find unsigned processes created on a specific agent
event.type = "Process Creation" and agent.uuid = "agent-123" and tgt.process.signedStatus = "unsigned"

// Investigate process creation timeline for a specific agent (last 24 hours)
event.type = "Process Creation" and agent.uuid = "agent-123"

// Find processes with suspicious command line patterns
event.type = "Process Creation" and (tgt.process.cmdline contains "powershell" and tgt.process.cmdline contains "encoded") and agent.uuid = "agent-123"

// Track process tree from a specific SHA256 hash
event.type = "Process Creation" and (src.process.image.sha256 = "hash123" or tgt.process.image.sha256 = "hash123") and agent.uuid = "agent-123"
```

---
