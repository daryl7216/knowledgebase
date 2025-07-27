## XarFileWritten Table: File Written Event

### 1. Table Name
**XarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a XAR (eXtensible ARchive) file. This format is common for macOS installer files. It provides details about the file and the process involved, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/Users/admin/Downloads/installer.xar",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "installer.xar"
}
```

---

### 5. Example Queries
```xql
// Find all XAR files written with a specific target file name
event_simpleName=XarFileWritten TargetFileName="*installer.xar*"

// Find XAR files written to a network drive (Windows only)
event_simpleName=XarFileWritten IsOnNetwork=true

// Find XAR files written to a removable disk (Windows only)
event_simpleName=XarFileWritten IsOnRemovableDisk=true

// Investigate XAR file writes by a specific file operator SID (Windows only)
event_simpleName=XarFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find XAR files based on their context base file name
event_simpleName=XarFileWritten ContextBaseFileName="important_package.xar"
```