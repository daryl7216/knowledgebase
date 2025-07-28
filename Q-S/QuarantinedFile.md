## QuarantinedFile Table: File Quarantine Event

### 1. Table Name
**QuarantinedFile**

---

### 2. Table Usage
This event is generated when a file is quarantined by Falcon Prevent. This is a critical remediation event, indicating that a malicious or suspicious file has been detected and neutralized by moving it to a secure location where it can no longer execute or be accessed. Monitoring these events is essential for confirming threat containment and understanding what malicious files have been found on hosts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full original path to the executable or file that was quarantined. | All |
| IsOnRemovableDisk | A boolean value that is true if the quarantined file was originally located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\user\\Downloads\\malicious_payload.exe",
  "IsOnRemovableDisk": false
}
```

---

### 5. Example Queries
```xql
// Find all file quarantine events
event_simpleName=QuarantinedFile

// Hunt for a specific file that has been quarantined
event_simpleName=QuarantinedFile ImageFileName="*\\mimikatz.exe"

// Find all files that were quarantined from removable media
event_simpleName=QuarantinedFile IsOnRemovableDisk=true

// Find quarantined files in a specific user's download folder
event_simpleName=QuarantinedFile ImageFileName="*\\Downloads\\*"
```
---