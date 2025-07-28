## RansomwareFileAccessPattern Table: Ransomware File Access Pattern Event

### 1. Table Name
**RansomwareFileAccessPattern**

---

### 2. Table Usage
This is a helper event for detecting ransomware file access patterns on Windows hosts. It is generated when a process accesses files in a way that is characteristic of ransomware, such as rapidly reading and then writing to or renaming a large number of files. This event is a strong indicator of an active ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file of the process exhibiting ransomware-like behavior. | Windows Only |
| CommandLine | The command line used to launch the suspicious process. | Windows Only |
| SourceFileName | The original name of a file that was accessed by the process. | Windows Only |
| TargetFileName | The name of the file after being modified or renamed by the process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\Public\\cryptolocker.exe",
  "CommandLine": "C:\\Users\\Public\\cryptolocker.exe -encrypt",
  "SourceFileName": "C:\\Users\\jsmith\\Documents\\report.docx",
  "TargetFileName": "C:\\Users\\jsmith\\Documents\\report.docx.encrypted"
}
```
---

### 5. Example Queries
```xql
// Find all ransomware file access pattern events
event_simpleName=RansomwareFileAccessPattern

// Hunt for a specific process that is exhibiting ransomware behavior
event_simpleName=RansomwareFileAccessPattern ImageFileName="*\\wannacry.exe"

// Find instances where files with a specific original name are being targeted
event_simpleName=RansomwareFileAccessPattern SourceFileName="*financials.xlsx"

// Search for ransomware that renames files with a specific new extension
event_simpleName=RansomwareFileAccessPattern TargetFileName="*.locked"

// Investigate the command line used by processes flagged for ransomware-like file access
event_simpleName=RansomwareFileAccessPattern | table CommandLine, ImageFileName, SourceFileName, TargetFileName
```
---