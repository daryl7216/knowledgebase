## CrxFileWritten Table: CRX File Written Event

### 1. Table Name
**CrxFileWritten**

---

### 2. Table Usage
This event indicates when a CRX (Chrome Extension) file has been written to disk. It's useful for tracking potential browser compromise or unauthorized extension installations.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | Windows Only       |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                            | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                                         | All                |
| UserName            | The name of the user associated with the event.                                              | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\extension.crx",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "extension.crx",
  "UserName": "johndoe"
}
```
---
### 5. Example Queries
```xql
// Find all CRX files written with a specific target file name
event_simpleName=CrxFileWritten TargetFileName="*malicious_extension.crx*"

// Find CRX files written to a network drive (Windows only)
event_simpleName=CrxFileWritten IsOnNetwork=true

// Find CRX files written to a removable disk (Windows only)
event_simpleName=CrxFileWritten IsOnRemovableDisk=true

// Investigate CRX file writes by a specific file operator SID (Windows only)
event_simpleName=CrxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find CRX files based on their context base file name
event_simpleName=CrxFileWritten ContextBaseFileName="adblock.crx"

// Find CRX files written by a specific user
event_simpleName=CrxFileWritten UserName="johndoe"
```
---