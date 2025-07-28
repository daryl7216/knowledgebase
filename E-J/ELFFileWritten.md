## ELFFileWritten Table: ELF File Written Event

### 1. Table Name
**ELFFileWritten**

---

### 2. Table Usage
This table records events when a file with a valid ELF (Executable and Linkable Format) image header is written to the system. ELF files are common executable and shared library formats on Unix-like operating systems. Monitoring these events is crucial for detecting the delivery of new executables, shared objects, or core dumps, which could indicate malware deployment, software installation, or system compromise.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was written. | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/usr/local/bin/new_utility",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all ELF files written with a specific target file name
event_simpleName=ELFFileWritten TargetFileName="/tmp/malicious_elf"

// Find ELF files written to a specific directory
event_simpleName=ELFFileWritten TargetFileName="/var/opt/new_app/*"

// Identify recently written ELF executables by file name patterns
event_simpleName=ELFFileWritten TargetFileName="*.elf" OR TargetFileName="*.so" OR TargetFileName="*.bin"
```
---