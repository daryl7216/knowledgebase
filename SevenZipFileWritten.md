## SevenZipFileWritten Table: File Written Event

### 1. Table Name
**SevenZipFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a 7-Zip archive file (.7z). Attackers often use archiving tools like 7-Zip to compress and stage stolen data before exfiltration. Monitoring the creation of these files is useful for investigating data staging, potential data theft, and malware deployment.

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
  "TargetFileName": "C:\\Users\\user\\Documents\\archive.7z",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "archive.7z"
}
```

---

### 5. Example Queries
```xql
// Find all 7-Zip files written to disk
event_simpleName=SevenZipFileWritten TargetFileName="*.7z"

// Find 7-Zip files written to a network drive (Windows only)
event_simpleName=SevenZipFileWritten IsOnNetwork=true

// Find 7-Zip files written to a removable disk (Windows only)
event_simpleName=SevenZipFileWritten IsOnRemovableDisk=true

// Investigate 7-Zip file writes by a specific file operator SID (Windows only)
event_simpleName=SevenZipFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find 7-Zip files based on their context base file name, which might indicate staging
event_simpleName=SevenZipFileWritten ContextBaseFileName="*secret*.7z"
```
---