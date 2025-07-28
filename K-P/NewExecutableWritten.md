## Crowdstrike Table: NewExecutableWritten

### 1. Table Name

**NewExecutableWritten**

---

### 2. Table Usage

This table is generated when an executable file extension is written, whether or not it is truly an executable file type. Any file that ends with a known executable file extension (e.g. .exe, .bat, .scr) will generate this event. The difference between a NewExecutableWritten event and PeFileWritten is that NewExecutableWritten looks for the extension of a file written, while PEFileWritten looks for the PE header in a file that was written. This event is useful for detecting potential malware drops, suspicious file creation, and persistence mechanisms across Linux, macOS, and Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was written. | All |
| ContextImageFileName | The image file name of the process that generated the event. | Windows |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "malware.exe",
  "TargetFileName": "C:\\Users\\user\\Downloads\\malware.exe",
  "ContextImageFileName": "C:\\Windows\\System32\\cmd.exe"
}
```

---

### 5. Example Queries

```xql
// Find all newly written executable files
event_simpleName=NewExecutableWritten

// Find newly written executable files with a specific target file name
event_simpleName=NewExecutableWritten TargetFileName="*temp_script.bat*"

// Find newly written executable files by the base name of the context file
event_simpleName=NewExecutableWritten ContextBaseFileName="installer.exe"

// Find newly written executable files where the context image file name indicates a suspicious origin (Windows only)
event_simpleName=NewExecutableWritten ContextImageFileName="*\\AppData\\Local\\Temp\\*"
```

---