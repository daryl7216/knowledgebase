## ImgExtensionFileWritten Table: File Written Event

### 1. Table Name
**ImgExtensionFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an IMG disk image file (`.img`). Attackers use IMG files, similar to ISO files, to deliver malware. Because these files are mounted as a drive when opened, they can bypass certain security controls, making them a popular container for malicious executables, scripts, and LNK files.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the full path of the .img file). | Windows Only |
| ContextBaseFileName | The base name of the context file (the name of the .img file itself). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\Urgent_Invoice.img",
  "ContextBaseFileName": "Urgent_Invoice.img"
}
```
---

### 5. Example Queries
```xql
// Find all IMG files written to disk
event_simpleName=ImgExtensionFileWritten TargetFileName="*.img"

// Hunt for IMG files written to common download or temp directories
event_simpleName=ImgExtensionFileWritten TargetFileName IN ("*\\Downloads\\*", "*\\Temp\\*")

// Find IMG files with potentially suspicious names
event_simpleName=ImgExtensionFileWritten ContextBaseFileName IN ("*invoice*", "*urgent*", "*payment*")
```