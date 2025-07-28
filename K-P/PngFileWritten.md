## PngFileWritten Table: File Written Event

### 1. Table Name
**PngFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a PNG (Portable Network Graphics) image file. While PNG is a common and legitimate image format, monitoring its creation can be useful in specific threat hunting scenarios, such as detecting the exfiltration of sensitive information via screenshots or steganography, where data is hidden within an image file.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Pictures\\Screenshots\\screenshot_2023-10-27.png",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "screenshot_2023-10-27.png"
}
```
---

### 5. Example Queries
```xql
// Find all PNG files written to disk
event_simpleName=PngFileWritten TargetFileName="*.png"

// Find PNG files written to a network drive (Windows only)
event_simpleName=PngFileWritten IsOnNetwork=true

// Find PNG files written to a removable disk (Windows only)
event_simpleName=PngFileWritten IsOnRemovableDisk=true

// Investigate PNG file writes by a specific file operator SID (Windows only)
event_simpleName=PngFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Hunt for screenshot-related PNG files being created
event_simpleName=PngFileWritten ContextBaseFileName="*screenshot*"
```
---