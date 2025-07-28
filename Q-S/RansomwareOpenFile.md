## RansomwareOpenFile Table: Ransomware File Open Event

### 1. Table Name
**RansomwareOpenFile**

---

### 2. Table Usage
This event is generated when a file is opened in a manner that is indicative of ransomware activity. Ransomware needs to open files in order to read their content and encrypt them. This event is a critical indicator for detecting the active encryption phase of a ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path to the file that was opened by the suspected ransomware process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\important_report.docx"
}
```

---
### 5. Example Queries
```xql
// Find all files opened by suspected ransomware processes
event_simpleName=RansomwareOpenFile

// Hunt for specific file types, like documents or spreadsheets, being opened by ransomware
event_simpleName=RansomwareOpenFile TargetFileName="*.docx" OR TargetFileName="*.xlsx"

// Find ransomware activity within a specific user's profile
event_simpleName=RansomwareOpenFile TargetFileName="C:\\Users\\jsmith\\*"
```
---