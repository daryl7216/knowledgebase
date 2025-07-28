## SuspiciousEseFileWritten Table: Credential File Copy Event

### 1. Table Name
**SuspiciousEseFileWritten**

---

### 2. Table Usage
This event indicates that a possible domain credential file (NTDS.dit) was copied, likely from a volume snapshot. The NTDS.dit file contains the Active Directory database, including user password hashes. Attackers copy this file to perform offline credential theft attacks. Monitoring this event is critical for detecting attempts to compromise an entire domain.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| SourceFileName | The original name of the file that was copied, typically `ntds.dit`. | Windows Only |
| TargetFileName | The destination path where the credential file was written. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "SourceFileName": "\\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy1\\Windows\\NTDS\\ntds.dit",
  "TargetFileName": "C:\\Windows\\Temp\\activedirectory.dat"
}
```

---

### 5. Example Queries
```xql
// Find all suspicious ESE file write events
event_simpleName=SuspiciousEseFileWritten

// Find instances where ntds.dit is copied from a volume shadow copy
event_simpleName=SuspiciousEseFileWritten SourceFileName="*\\ntds.dit"

// Hunt for ntds.dit being saved with a different name
event_simpleName=SuspiciousEseFileWritten TargetFileName!="*\\ntds.dit"
```
---
