## Crowdstrike Table: OleFileWritten

### 1. Table Name

**OleFileWritten**

---

### 2. Table Usage

This event is generated when a Microsoft Office (Pre-Office 2007) file type is written to disk. This table is useful for investigations into file creation, data exfiltration, and malware drops, especially concerning older Office document formats, across Windows, Linux, and macOS platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "legacy_document.doc",
  "TargetFileName": "C:\\Users\\user\\Documents\\legacy_document.doc",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false
}
```

---

### 5. Example Queries

```xql
// Find all OLE files written with a specific target file name
event_simpleName=OleFileWritten TargetFileName="*confidential_spreadsheet.xls*"

// Find OLE files written to a network drive (Windows only)
event_simpleName=OleFileWritten IsOnNetwork=true

// Find OLE files written to a removable disk
event_simpleName=OleFileWritten IsOnRemovableDisk=true

// Find OLE files based on their context base file name
event_simpleName=OleFileWritten ContextBaseFileName="old_report.ppt"
```

---