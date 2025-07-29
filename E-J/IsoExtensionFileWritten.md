## IsoExtensionFileWritten Table: File Written Event

### 1. Table Name
**IsoExtensionFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ISO disk image file (`.iso`). Attackers increasingly use ISO files to deliver malware, often within phishing emails, because these files can bypass certain security controls. When a user mounts an ISO file, its contents appear as a virtual drive, making it a convenient container for malicious executables, scripts, and LNK files.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the full path of the .iso file). | Windows Only |
| ContextBaseFileName | The base name of the context file (the name of the .iso file itself). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\Urgent_Invoice.iso",
  "ContextBaseFileName": "Urgent_Invoice.iso"
}
```
---
### 5. Example Queries
```xql
// Find all ISO files written to disk
event_simpleName=IsoExtensionFileWritten TargetFileName="*.iso"

// Hunt for ISO files written to common download or temp directories
event_simpleName=IsoExtensionFileWritten TargetFileName IN ("*\\Downloads\\*", "*\\Temp\\*")

// Find ISO files with potentially suspicious names
event_simpleName=IsoExtensionFileWritten ContextBaseFileName IN ("*invoice*", "*urgent*", "*payment*")
```
---