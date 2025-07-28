## Crowdstrike Table: MSPptxFileWritten

### 1. Table Name

**MSPptxFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing an MS Office Open XML Presentation (PPTX) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FsMagic | File system magic number. | Linux |
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| MSOfficeSubType | Values: UNKNOWN (0), OLECFB (1), OOXML (2). | All |
| FileCategory | Values: OTHER (0), ARCHIVES (1), OFFICE_DOCUMENTS (2), MULTIMEDIA_FILES (3), DESIGN_FILES (4), SOURCE_CODE (5), EXECUTABLE_FILES (6), VIRTUAL_MACHINE_FILES (7), EMAIL_FILES (8), DATA_AND_LOGS (9), ENCRYPTED (10). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "FsMagic": 61267,
  "ContextBaseFileName": "presentation.pptx",
  "TargetFileName": "C:\\Users\\user\\Documents\\presentation.pptx",
  "MSOfficeSubType": 2,
  "FileCategory": 2,
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MS Pptx files written with a specific target file name
event_simpleName=MSPptxFileWritten TargetFileName="*confidential_presentation.pptx*"

// Find MS Pptx files written to a network drive (Windows only)
event_simpleName=MSPptxFileWritten IsOnNetwork=true

// Find MS Pptx files written to a removable disk
event_simpleName=MSPptxFileWritten IsOnRemovableDisk=true

// Investigate MS Pptx file writes by a specific file operator SID (Windows only)
event_simpleName=MSPptxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MS Pptx files based on their context base file name
event_simpleName=MSPptxFileWritten ContextBaseFileName="quarterly_report.pptx"

// Find MS Pptx files that are of OOXML subtype
event_simpleName=MSPptxFileWritten MSOfficeSubType=2

// Find MS Pptx files categorized as Office Documents
event_simpleName=MSPptxFileWritten FileCategory=2

// Find MS Pptx files on a specific file system magic (Linux only)
event_simpleName=MSPptxFileWritten FsMagic=61267
```

---