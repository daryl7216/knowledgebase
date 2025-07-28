## WebScriptFileWritten Table: File Written Event

### 1. Table Name
**WebScriptFileWritten**

---

### 2. Table Usage
This table records events when a web script file (e.g., JavaScript, VBScript) is written to disk. This activity is often associated with drive-by downloads or the saving of malicious scripts from web pages. Monitoring these events is useful for investigating malware delivery, data exfiltration, and the initial stages of a compromise.

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
  "TargetFileName": "C:\\Users\\user\\Downloads\\malicious_script.js",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "malicious_script.js"
}
```

---

### 5. Example Queries
```xql
// Find all JavaScript files written to disk
event_simpleName=WebScriptFileWritten TargetFileName="*.js"

// Find web scripts written to a network drive (Windows only)
event_simpleName=WebScriptFileWritten IsOnNetwork=true

// Find web scripts written to a removable disk (Windows only)
event_simpleName=WebScriptFileWritten IsOnRemovableDisk=true

// Investigate web script file writes by a specific file operator SID (Windows only)
event_simpleName=WebScriptFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find web scripts based on their context base file name
event_simpleName=WebScriptFileWritten ContextBaseFileName="payload.vbs"
```
---
