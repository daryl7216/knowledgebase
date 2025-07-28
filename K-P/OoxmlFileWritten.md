## Crowdstrike Table: OoxmlFileWritten

### 1. Table Name

**OoxmlFileWritten**

---

### 2. Table Usage

This table is generated when a Microsoft Office (Post-Office 2007) file type is written to disk. This event is crucial for investigations into file creation, data exfiltration, and malware drops, especially concerning newer Office Open XML formats, across Windows, Linux, and macOS platforms.

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
  "ContextBaseFileName": "modern_report.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\modern_report.docx",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false
}
```

---

### 5. Example Queries

```xql
// Find all OOXML files written with a specific target file name
event_simpleName=OoxmlFileWritten TargetFileName="*quarterly_results.xlsx*"

// Find OOXML files written to a network drive (Windows only)
event_simpleName=OoxmlFileWritten IsOnNetwork=true

// Find OOXML files written to a removable disk
event_simpleName=OoxmlFileWritten IsOnRemovableDisk=true

// Find OOXML files based on their context base file name
event_simpleName=OoxmlFileWritten ContextBaseFileName="presentation.pptx"
```

---