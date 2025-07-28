## LnkFileWritten Table: File Written Event

### 1. Table Name
**LnkFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Windows Shortcut File (.lnk). Attackers frequently use LNK files for persistence and initial execution, often by placing them in startup folders or sending them as phishing attachments. These malicious shortcuts can be crafted to execute arbitrary commands, PowerShell scripts, or download further malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the path of the .lnk file). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\Update.lnk",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "Update.lnk"
}
```

---

### 5. Example Queries
```xql
// Find all LNK files written to disk
event_simpleName=LnkFileWritten TargetFileName="*.lnk"

// Hunt for LNK files being written to user Startup folders for persistence
event_simpleName=LnkFileWritten TargetFileName="*\\Start Menu\\Programs\\Startup\\*.lnk"

// Find LNK files written to a network drive (Windows only)
event_simpleName=LnkFileWritten IsOnNetwork=true

// Find LNK files written to removable media, which could be used for physical attacks
event_simpleName=LnkFileWritten IsOnRemovableDisk=true

// Investigate LNK file writes by a specific file operator SID (Windows only)
event_simpleName=LnkFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"
```
---