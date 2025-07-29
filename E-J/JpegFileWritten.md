## JpegFileWritten Table: File Written Event

### 1. Table Name
**JpegFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a JPEG image file. While JPEG is a very common image format, monitoring its creation can be useful in specific threat hunting scenarios, such as detecting the exfiltration of sensitive information via screenshots or identifying steganography, where attackers hide data within image files.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Pictures\\Screenshots\\screenshot-2023.jpg",
  "ContextBaseFileName": "screenshot-2023.jpg"
}
```
---
### 5. Example Queries
```xql
// Find all JPEG files written to disk
event_simpleName=JpegFileWritten TargetFileName IN ("*.jpg", "*.jpeg")

// Hunt for JPEG files being written to temporary or download directories
event_simpleName=JpegFileWritten TargetFileName IN ("*\\Temp\\*", "*\\Downloads\\*")

// Find JPEG files that might be screenshots
event_simpleName=JpegFileWritten ContextBaseFileName="*screenshot*"
```
---