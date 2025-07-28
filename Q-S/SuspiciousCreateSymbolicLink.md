## SuspiciousCreateSymbolicLink Table: Symbolic Link Creation Event

### 1. Table Name
**SuspiciousCreateSymbolicLink**

---

### 2. Table Usage
This event is generated when a suspicious symbolic link is created on a Windows host. Attackers can abuse symbolic links to redirect file operations, which can lead to privilege escalation (e.g., UAC bypass) or be used as a persistence mechanism. Monitoring the creation of these links is important for detecting such malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process that created the symbolic link (e.g., `cmd.exe`). | Windows Only |
| CommandLine | The command line used by the process to create the symbolic link. | Windows Only |
| SymbolicLinkTarget | The target path to which the symbolic link points. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "mklink C:\\Users\\Public\\exploit.dll C:\\Windows\\System32\\uxtheme.dll",
  "SymbolicLinkTarget": "C:\\Windows\\System32\\uxtheme.dll"
}
```
---
### 5. Example Queries
```xql
// Find all suspicious symbolic link creation events
event_simpleName=SuspiciousCreateSymbolicLink

// Find symbolic links created by cmd.exe
event_simpleName=SuspiciousCreateSymbolicLink ImageFileName="*\\cmd.exe"

// Hunt for symbolic links that point to files in the System32 directory
event_simpleName=SuspiciousCreateSymbolicLink SymbolicLinkTarget="*\\System32\\*"

// Find symbolic links created with a specific command line
event_simpleName=SuspiciousCreateSymbolicLink CommandLine="*mklink*"
```
---
