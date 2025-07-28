## RpmFileWritten Table: File Written Event

### 1. Table Name
**RpmFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an RPM (Red Hat Package Manager) file. RPM files are a standard way to install software on many Linux distributions. Attackers may create or write RPM files to disk as a method of deploying malware or other malicious tools. Monitoring these events is useful for tracking software installation and detecting potential malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/tmp/malicious-package-1.0.rpm",
  "ContextBaseFileName": "malicious-package-1.0.rpm"
}
```

---
### 5. Example Queries

```xql
// Find all RPM files written to disk
event_simpleName=RpmFileWritten TargetFileName="*.rpm"

// Hunt for RPM files written to temporary directories, which is suspicious
event_simpleName=RpmFileWritten TargetFileName="/tmp/*.rpm" OR TargetFileName="/var/tmp/*.rpm"

// Find RPM files based on their context base file name
event_simpleName=RpmFileWritten ContextBaseFileName="*payload*.rpm"
```
---