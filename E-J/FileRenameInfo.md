## FileRenameInfo Table: File Rename Event

### 1. Table Name
**FileRenameInfo**

---

### 2. Table Usage
This table records events generated when a file is renamed on the endpoint. This event is typically sent as part of a detection. Monitoring file renames can help identify malicious activities such as file obfuscation, evasion techniques, or data manipulation.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| SourceFileName | The file's original name.                 | All                |
| TargetFileName | The file's new name after the rename operation. | All                |

---

### 4. Sample Log Example

```json
{
  "SourceFileName": "C:\\Users\\user\\Documents\\old_report.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\final_report.docx"
}
```
---

### 5. Example Queries
```xql
// Find all file rename events
event_simpleName=FileRenameInfo

// Find file rename events for a specific source file name
event_simpleName=FileRenameInfo SourceFileName="*temp_file.txt*"

// Find file rename events where a file was renamed to a suspicious extension
event_simpleName=FileRenameInfo TargetFileName="*.exe" SourceFileName!="*.exe"

// Investigate file renames where the original file was in a sensitive directory
event_simpleName=FileRenameInfo SourceFileName="*C:\\Windows\\System32\\*"
```
---