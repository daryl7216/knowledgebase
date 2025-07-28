## PdfFileWritten Table: File Written Event

### 1. Table Name
**PdfFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a PDF (Portable Document Format) file. While PDFs are a common document format, they can also be used as a vector for phishing attacks, containing malicious links or embedded exploits. Monitoring the creation of PDF files can be useful for investigating malware delivery and document-based threats.

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
  "TargetFileName": "C:\\Users\\user\\Downloads\\Malicious_Invoice.pdf",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "Malicious_Invoice.pdf"
}
```
---

### 5. Example Queries

```xql
// Find all PDF files written to disk
event_simpleName=PdfFileWritten TargetFileName="*.pdf"

// Find PDF files written to a network drive (Windows only)
event_simpleName=PdfFileWritten IsOnNetwork=true

// Find PDF files written to a removable disk (Windows only)
event_simpleName=PdfFileWritten IsOnRemovableDisk=true

// Investigate PDF file writes by a specific file operator SID (Windows only)
event_simpleName=PdfFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find PDF files with a potentially suspicious name like "invoice" or "password"
event_simpleName=PdfFileWritten ContextBaseFileName="*invoice*.pdf" OR ContextBaseFileName="*password*.pdf"
```
---