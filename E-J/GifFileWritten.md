## GifFileWritten Table: GIF File Written Event

### 1. Table Name
**GifFileWritten**

---

### 2. Table Usage
This table records events when a process is done writing a GIF image file. Monitoring these events can be useful for tracking image generation, potential data manipulation, or the creation of visual artifacts by applications, especially in the context of web content or multimedia.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                          | All                |
| TargetFileName      | The resulting file name that was downloaded.                                 | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "animated_banner.gif",
  "TargetFileName": "C:\\Users\\Public\\Pictures\\animated_banner.gif",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-9876543210-..."
}
```
---

### 5. Example Queries
```xql
// Find all GIF files written to a network drive (Windows only)
event_simpleName=GifFileWritten IsOnNetwork=true

// Find GIF files written to a removable disk
event_simpleName=GifFileWritten IsOnRemovableDisk=true

// Investigate GIF file writes by a specific file operator SID (Windows only)
event_simpleName=GifFileWritten FileOperatorSid="S-1-5-21-9876543210-*"

// Find GIF files with a specific target file name pattern
event_simpleName=GifFileWritten TargetFileName="*thumbnail*.gif"

// Find GIF files based on their context base file name
event_simpleName=GifFileWritten ContextBaseFileName="loading_spinner.gif"
```
---