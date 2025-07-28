## RansomwareCreateFile Table: Ransomware File Creation Event

### 1. Table Name
**RansomwareCreateFile**

---

### 2. Table Usage
This event is generated when a process creates a file in a manner that is indicative of ransomware activity. This often occurs when ransomware drops a ransom note onto the system or creates the new, encrypted version of a victim's file. Monitoring this event is critical for detecting the active encryption phase of a ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file of the process that created the file. | Windows Only |
| CommandLine | The command line used to launch the process that created the file. | Windows Only |
| TargetFileName | The full path to the file that was created by the suspected ransomware process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\Public\\crypt.exe",
  "CommandLine": "C:\\Users\\Public\\crypt.exe --goham",
  "TargetFileName": "C:\\Users\\user\\Documents\\DECRYPT_INSTRUCTIONS.txt"
}
```

---

### 5. Example Queries
```xql
// Find all ransomware file creation events
event_simpleName=RansomwareCreateFile

// Hunt for the creation of common ransom note file names
event_simpleName=RansomwareCreateFile TargetFileName="*DECRYPT*" OR TargetFileName="*RECOVER*" OR TargetFileName="*README*.txt"

// Find files created by a specific suspected ransomware process
event_simpleName=RansomwareCreateFile ImageFileName="*\\cryptolocker.exe"

// Investigate the command line used by processes that are creating ransomware-related files
event_simpleName=RansomwareCreateFile | table CommandLine, ImageFileName, TargetFileName
```
---
