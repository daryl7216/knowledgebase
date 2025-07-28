## TiffFileWritten Table: File Written Event

### 1. Table Name
**TiffFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a TIFF (Tagged Image File Format) file. While TIFF is a common image format, it has also been associated with vulnerabilities in image parsing libraries and can be used for steganography. Monitoring the creation of these files can be useful for investigating malware delivery and data hiding techniques.

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
  "TargetFileName": "C:\\Users\\user\\Pictures\\scan_001.tiff",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "scan_001.tiff"
}
```
---
### 5. Example Queries
```xql
// Find all TIFF files written to disk
event_simpleName=TiffFileWritten TargetFileName="*.tiff" OR TargetFileName="*.tif"

// Find TIFF files written to a network drive (Windows only)
event_simpleName=TiffFileWritten IsOnNetwork=true

// Find TIFF files written to a removable disk (Windows only)
event_simpleName=TiffFileWritten IsOnRemovableDisk=true

// Investigate TIFF file writes by a specific file operator SID (Windows only)
event_simpleName=TiffFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find TIFF files based on their context base file name
event_simpleName=TiffFileWritten ContextBaseFileName="secret_scan.tif"
```
---
