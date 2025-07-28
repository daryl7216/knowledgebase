## SuspiciousRegAsepUpdate Table: Registry Persistence Event

### 1. Table Name
**SuspiciousRegAsepUpdate**

---

### 2. Table Usage
This event describes registry activity that triggered a detection for a suspicious update to an Autostart Extension Point (ASEP). ASEPs are registry keys that cause programs to run automatically (e.g., at startup or user logon). Attackers frequently modify these keys to establish persistence on a compromised system.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path of the process that performed the suspicious registry modification. | Windows Only |
| CommandLine | The full command line of the process that performed the registry modification. | Windows Only |
| RegOperationType | The type of registry operation, such as creating a key or setting a value. | Windows Only |
| RegKeyName | The full path of the registry key that was modified. | Windows Only |
| RegValueName | The name of the specific registry value that was created or modified. | Windows Only |
| RegStringValue | The string data written to the registry value. This often contains the path to the malicious executable. | Windows Only |
| TargetFileName | The file path specified within the registry value that is intended for auto-execution. | Windows Only |
| TargetCommandLineParameters | The command-line parameters associated with the `TargetFileName` in the registry value. | Windows Only |
| RegObjectName | The name of the registry object being manipulated. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\reg.exe",
  "CommandLine": "reg.exe add \"HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\" /v \"MalwareUpdater\" /t REG_SZ /d \"C:\\Users\\admin\\AppData\\Local\\Temp\\payload.exe\"",
  "RegOperationType": "1",
  "RegKeyName": "HKEY_CURRENT_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run",
  "RegValueName": "MalwareUpdater",
  "RegStringValue": "C:\\Users\\admin\\AppData\\Local\\Temp\\payload.exe",
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\payload.exe",
  "TargetCommandLineParameters": "",
  "RegObjectName": "\\REGISTRY\\USER\\S-1-5-21-12345\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"
}
```

---

### 5. Example Queries

```xql
// Find all suspicious ASEP updates
event_simpleName=SuspiciousRegAsepUpdate

// Hunt for modifications to common Run keys
event_simpleName=SuspiciousRegAsepUpdate RegKeyName="*\\CurrentVersion\\Run"

// Find ASEP updates performed by PowerShell
event_simpleName=SuspiciousRegAsepUpdate ImageFileName="*\\powershell.exe"

// Search for a specific executable being set for persistence
event_simpleName=SuspiciousRegAsepUpdate TargetFileName="*\\evil.exe"

// Look for ASEP updates that set a value containing a script file
event_simpleName=SuspiciousRegAsepUpdate RegStringValue="*.js" OR RegStringValue="*.vbs" OR RegStringValue="*.ps1"
```
---
