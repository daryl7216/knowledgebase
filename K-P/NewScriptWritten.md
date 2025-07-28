## Crowdstrike Table: NewScriptWritten

### 1. Table Name

**NewScriptWritten**

---

### 2. Table Usage

This table is emitted when a process is done writing a script file, as determined by `#!` at the start of the file on Linux and macOS, or when a new script with extensions like .js, .aspx, .bat, .ps1, .vbs, .lua is written on Windows. This event is crucial for detecting the creation of suspicious scripts, which can indicate malware drops, persistence mechanisms, or unauthorized automation.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was written. | All |
| ContextImageFileName | The image file name of the process that generated the event. | Windows |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "malicious_script.ps1",
  "TargetFileName": "C:\\Temp\\malicious_script.ps1",
  "ContextImageFileName": "C:\\Windows\\System32\\cmd.exe"
}
```

---

### 5. Example Queries

```xql
// Find all newly written script files
event_simpleName=NewScriptWritten

// Find newly written script files with a specific target file name
event_simpleName=NewScriptWritten TargetFileName="*temp_script.sh*"

// Find newly written script files by the base name of the context file
event_simpleName=NewScriptWritten ContextBaseFileName="payload.js"

// Find newly written script files where the context image file name indicates a suspicious origin (Windows only)
event_simpleName=NewScriptWritten ContextImageFileName="*\\Users\\Public\\*"
```

---