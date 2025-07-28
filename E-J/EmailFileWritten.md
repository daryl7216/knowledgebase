## EmailFileWritten Table: Email File Written Event

### 1. Table Name
**EmailFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an email-related file (e.g., .eml, .msg, .pst). Monitoring these events can be crucial for detecting data exfiltration via email, phishing attempts, or the storage of sensitive communications.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was written. | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\draft_email.eml",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all email files written with a specific target file name
event_simpleName=EmailFileWritten TargetFileName="*sensitive_report.msg*"

// Find email files written to a temporary directory
event_simpleName=EmailFileWritten TargetFileName="*\\Temp\\*.eml*"

// Identify recently written email files by common extensions
event_simpleName=EmailFileWritten TargetFileName="*.eml" OR TargetFileName="*.msg" OR TargetFileName="*.pst"
```
---