## RtfFileWritten Table: File Written Event

### 1. Table Name
**RtfFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Rich Text Format (RTF) file. Attackers have historically abused the RTF file format to embed malicious objects and exploits, often used in phishing campaigns to gain initial access. Monitoring the creation of RTF files is useful for investigating potential malware delivery and document-based attacks.

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
  "TargetFileName": "C:\\Users\\user\\Downloads\\Invoice.rtf",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "Invoice.rtf"
}
```
---
### 5. Example Queries

```xql
// Find all RTF files written to disk
event_simpleName=RtfFileWritten TargetFileName="*.rtf"

// Find RTF files written to a network drive (Windows only)
event_simpleName=RtfFileWritten IsOnNetwork=true

// Find RTF files written to a removable disk (Windows only)
event_simpleName=RtfFileWritten IsOnRemovableDisk=true

// Investigate RTF file writes by a specific file operator SID (Windows only)
event_simpleName=RtfFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find RTF files with a potentially suspicious name written to a download folder
event_simpleName=RtfFileWritten ContextBaseFileName="*invoice*.rtf" TargetFileName="*\\Downloads\\*"
```
---