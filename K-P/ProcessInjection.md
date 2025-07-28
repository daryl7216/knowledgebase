## ProcessInjection Table: Process Injection Event

### 1. Table Name
**ProcessInjection**

---

### 2. Table Usage
This event is generated when a process writes and executes code within the address space of another process. Process injection is a powerful defense evasion and privilege escalation technique used extensively by malware to hide its malicious code within legitimate processes. Monitoring for these events is critical for detecting a wide range of advanced threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| InjectorImageFileName | The full path to the executable of the process that is performing the injection. | Windows Only |
| InjecteeImageFileName | The full path to the executable of the process that is being injected into. | Windows Only |
| TargetProcessId | The unique ID of the process being injected into (the injectee). | Windows Only |
| ModuleName | If the injection involves a specific module, its name is captured here. This is common in techniques like DLL injection. | Windows Only |
| WellKnownTargetFunction | Identifies if the injection targets a well-known, specific function within the injectee process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "InjectorImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious_loader.exe",
  "InjecteeImageFileName": "C:\\Windows\\System32\\explorer.exe",
  "TargetProcessId": "3456",
  "ModuleName": "C:\\Users\\admin\\AppData\\Local\\Temp\\payload.dll",
  "WellKnownTargetFunction": "0"
}
```

---

### 5. Example Queries

```xql
// Find all process injection events
event_simpleName=ProcessInjection

// Hunt for a specific legitimate process being injected into, like explorer.exe or svchost.exe
event_simpleName=ProcessInjection InjecteeImageFileName IN ("*\\explorer.exe", "*\\svchost.exe")

// Find injection attempts originating from processes running in suspicious locations
event_simpleName=ProcessInjection InjectorImageFileName="*\\Temp\\*" OR InjectorImageFileName="*\\Users\\Public\\*"

// Search for a specific malicious DLL being injected
event_simpleName=ProcessInjection ModuleName="*mimikatz*.dll"

// Investigate injection attempts targeting a specific process ID
event_simpleName=ProcessInjection TargetProcessId="1234"
```
---