## ReflectiveDotnetModuleLoad Table: Reflective .NET Module Load Event

### 1. Table Name
**ReflectiveDotnetModuleLoad**

---

### 2. Table Usage
This event is generated when a .NET module is reflectively loaded into a process, for example, from a byte array in memory rather than from a file on disk. This is a common technique used by fileless malware and in-memory execution frameworks (like Cobalt Strike) to evade detection by traditional antivirus solutions. Monitoring this event is critical for identifying advanced in-memory threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process in which the .NET module was reflectively loaded. | Windows Only |
| CommandLine | The command line of the process that loaded the module. | Windows Only |
| AssemblyName | The name of the assembly that was reflectively loaded. | Windows Only |
| ModuleILPath | The path to the Intermediate Language (IL) code of the loaded module, if available. | Windows Only |
| ModuleNativePath | The path to the native (Just-In-Time compiled) code of the loaded module, if available. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "CommandLine": "powershell -c [System.Reflection.Assembly]::Load([System.IO.File]::ReadAllBytes('C:\\Users\\Public\\mimikatz.dll'))",
  "AssemblyName": "mimikatz, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null",
  "ModuleILPath": "",
  "ModuleNativePath": ""
}
```

---

### 5. Example Queries

```xql
// Find all instances of reflective .NET module loading
event_simpleName=ReflectiveDotnetModuleLoad

// Hunt for reflective loading into PowerShell, a common technique for fileless attacks
event_simpleName=ReflectiveDotnetModuleLoad ImageFileName="*\\powershell.exe"

// Search for the loading of a specific suspicious assembly by name
event_simpleName=ReflectiveDotnetModuleLoad AssemblyName="*mimikatz*" OR AssemblyName="*PowerSploit*"

// Find instances where a process other than a known .NET host is reflectively loading modules
event_simpleName=ReflectiveDotnetModuleLoad ImageFileName!="C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\csc.exe"

// Investigate reflective loading based on the command line arguments used
event_simpleName=ReflectiveDotnetModuleLoad CommandLine="*System.Reflection.Assembly*"
```
---