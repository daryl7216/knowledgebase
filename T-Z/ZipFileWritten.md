## ZipFileWritten Table: File Written Event

### 1. Table Name
**ZipFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a ZIP file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was written.                                          | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                         | Windows Only       |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                                 | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\confidential_archive.zip",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "confidential_archive.zip"
}
```

---

### 5. Example Queries
```xql
// Find all ZIP files written with a specific target file name
event_simpleName=ZipFileWritten TargetFileName="*secret.zip*"

// Find ZIP files written to a network drive (Windows only)
event_simpleName=ZipFileWritten IsOnNetwork=true

// Find ZIP files written to a removable disk (Windows only)
event_simpleName=ZipFileWritten IsOnRemovableDisk=true

// Investigate ZIP file writes by a specific file operator SID (Windows only)
event_simpleName=ZipFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find ZIP files based on their context base file name
event_simpleName=ZipFileWritten ContextBaseFileName="backup.zip"
```
---
