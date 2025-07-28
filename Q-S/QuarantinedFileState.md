## QuarantinedFileState Table: Quarantined File State Event

### 1. Table Name
**QuarantinedFileState**

---

### 2. Table Usage
This event reports the state of a file that has been quarantined by Falcon Prevent. It is used to track whether a file is currently quarantined, has been released, or has been deleted. This is crucial for confirming that a threat has been successfully neutralized and for auditing any changes to the status of quarantined files, such as an accidental or malicious release.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full original path to the executable or file that was quarantined. | All |
| QuarantinedFileName | The name or path of the file within the quarantine storage. | All |
| QuarantinedFileState | A numeric code indicating the current state of the file, such as `1` (Quarantined), `2` (Released), or `3` (Deleted). | All |
| IsOnRemovableDisk | A boolean value that is true if the quarantined file was originally located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\user\\Downloads\\malicious_payload.exe",
  "IsOnRemovableDisk": false,
  "QuarantinedFileName": "C:\\ProgramData\\Falcon\\Quarantine\\012345ab-cdef-6789-0123-456789abcdef.qf",
  "QuarantinedFileState": "1"
}
```

---

### 5. Example Queries

```xql
// Find all quarantined file state events
event_simpleName=QuarantinedFileState

// Hunt for files that have been released from quarantine, which could be a security risk
event_simpleName=QuarantinedFileState QuarantinedFileState=2

// Find the quarantine status for a specific file
event_simpleName=QuarantinedFileState ImageFileName="*\\mimikatz.exe"

// Find all quarantined files that originated from a removable disk
event_simpleName=QuarantinedFileState IsOnRemovableDisk=true

// Show the original path and the quarantine path for all currently quarantined files
event_simpleName=QuarantinedFileState QuarantinedFileState=1 | table ImageFileName, QuarantinedFileName
```
---