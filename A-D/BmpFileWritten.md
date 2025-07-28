## BmpFileWritten Table: BMP File Written Event

### 1. Table Name
**BmpFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a BMP (Bitmap) image file. Monitoring these events can be useful for tracking image generation, potential data manipulation, or the creation of visual artifacts by applications.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                                | All                |
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                              | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "new_drawing.bmp",
  "TargetFileName": "C:\\Users\\Public\\Pictures\\new_drawing.bmp",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-9876543210-..."
}
```
---
### 5. Example Queries
```xql
// Find all BMP files written to a network drive (Windows only)
event_simpleName=BmpFileWritten IsOnNetwork=true

// Find BMP files written to a removable disk
event_simpleName=BmpFileWritten IsOnRemovableDisk=true

// Investigate BMP file writes by a specific file operator SID (Windows only)
event_simpleName=BmpFileWritten FileOperatorSid="S-1-5-21-9876543210-*"

// Find BMP files with a specific target file name
event_simpleName=BmpFileWritten TargetFileName="*screenshot.bmp*"

// Find BMP files based on their context base file name
event_simpleName=BmpFileWritten ContextBaseFileName="wallpaper.bmp"
```
---
