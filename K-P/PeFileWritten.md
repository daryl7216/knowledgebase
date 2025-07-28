## PeFileWritten Table: PE File Written Event

### 1. Table Name
**PeFileWritten**

---

### 2. Table Usage
This event is generated when a Windows Portable Executable (PE) file, such as an .exe or .dll, is written to disk. This is a critical event for threat hunting because it often signifies malware being dropped onto a system, an attacker staging tools for later use, or "living-off-the-land" techniques where legitimate utilities are written to disk for malicious purposes. This event provides rich context about both the file that was written and the process that wrote it.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path of the PE file that was written to disk. | All |
| ContextBaseFileName | The base name of the process that wrote the PE file. | All |
| ContextImageFileName | The full path of the process that wrote the PE file. | Windows Only |
| UserName | The user account that was active when the file was written. | All |
| IsOnNetwork | Set to true if the PE file was written to a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means the PE file was written to a removable disk. | All |
| FileOperatorSid | The Security Identifier (SID) of the user or entity that performed the file write operation. | Windows Only |
| ImageSubsystem | The subsystem required to run the PE file, such as GUI (`2`) or CUI (Console) (`3`). | Windows Only |
| ModuleCharacteristics | A bitmask of flags indicating the characteristics of the PE file, such as whether it's an executable (`2`) or a DLL (`8192`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.dll",
  "ContextBaseFileName": "powershell.exe",
  "ContextImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "UserName": "CORP\\admin",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-1234567890-123456789-1234567890-1001",
  "ImageSubsystem": "2",
  "ModuleCharacteristics": "8192"
}
```

---

### 5. Example Queries
```xql
// Find all PE files written to disk
event_simpleName=PeFileWritten

// Hunt for PE files written by scripting engines like PowerShell, which can indicate fileless attacks dropping a payload
event_simpleName=PeFileWritten ContextImageFileName IN ("*\\powershell.exe", "*\\wscript.exe", "*\\cscript.exe")

// Find PE files being written to suspicious locations like Temp or Public user directories
event_simpleName=PeFileWritten TargetFileName="*\\Temp\\*" OR TargetFileName="*\\Users\\Public\\*"

// Search for DLLs (ModuleCharacteristics=8192) being written, as they can be used for hijacking or sideloading
event_simpleName=PeFileWritten ModuleCharacteristics=8192

// Find PE files written to a removable disk, which might be used for data transfer or offline attacks
event_simpleName=PeFileWritten IsOnRemovableDisk=true
```
---