## RansomwareRenameFile Table: Ransomware File Rename Event

### 1. Table Name
**RansomwareRenameFile**

---

### 2. Table Usage
This event is generated when a file is renamed in a manner that is indicative of ransomware activity. Typically, ransomware encrypts files and then renames them, often by adding a specific extension. This event is critical for detecting the active encryption phase of a ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process that performed the file rename. | Windows Only |
| CommandLine | The command line of the process that renamed the file. | Windows Only |
| SourceFileName | The original name of the file before it was renamed. | Windows Only |
| TargetFileName | The new name of the file after being renamed by the ransomware. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\Public\\Documents\\evil.exe",
  "CommandLine": "C:\\Users\\Public\\Documents\\evil.exe -encrypt C:\\Users\\user\\Documents",
  "SourceFileName": "C:\\Users\\user\\Documents\\important_document.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\important_document.docx.locked"
}
```

---

### 5. Example Queries

```xql
// Find all ransomware file rename events
event_simpleName=RansomwareRenameFile

// Hunt for specific file extensions commonly used by ransomware
event_simpleName=RansomwareRenameFile TargetFileName="*.locked" OR TargetFileName="*.encrypted"

// Find what the original file name was for a renamed file
event_simpleName=RansomwareRenameFile TargetFileName="*report.docx.crypt*" | table SourceFileName

// Investigate which process is responsible for renaming files
event_simpleName=RansomwareRenameFile | stats count by ImageFileName

// Search for files being renamed by a process with a specific command line
event_simpleName=RansomwareRenameFile CommandLine="* -encrypt *"
```
---