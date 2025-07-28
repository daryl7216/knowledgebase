## PythonFileWritten Table: File Written Event

### 1. Table Name
**PythonFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Python script file (.py). Attackers often use Python scripts for a wide range of malicious activities, including reconnaissance, exploitation, and establishing backdoors. Monitoring the creation of these files is useful for investigating malware drops and the staging of attack tools.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\AppData\\Local\\Temp\\malicious_script.py",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "malicious_script.py"
}
```

---

### 5. Example Queries
```xql
// Find all Python files written to disk
event_simpleName=PythonFileWritten TargetFileName="*.py"

// Find Python files written to a network drive (Windows only)
event_simpleName=PythonFileWritten IsOnNetwork=true

// Find Python files written to a removable disk (Windows only)
event_simpleName=PythonFileWritten IsOnRemovableDisk=true

// Investigate Python file writes by a specific file operator SID (Windows only)
event_simpleName=PythonFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find Python scripts written to a temporary directory
event_simpleName=PythonFileWritten TargetFileName="*\\Temp\\*.py"
```
---