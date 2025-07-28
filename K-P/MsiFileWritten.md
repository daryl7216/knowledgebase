## Crowdstrike Table: MsiFileWritten

### 1. Table Name

**MsiFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing a Microsoft Installer (MSI) file. It provides details about the file, the process involved, and its location, which is useful for investigations into software installations, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "example_installer.msi",
  "TargetFileName": "C:\\Users\\user\\Downloads\\example_installer.msi",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MSI files written with a specific target file name
event_simpleName=MsiFileWritten TargetFileName="*malicious_setup.msi*"

// Find MSI files written to a network drive (Windows only)
event_simpleName=MsiFileWritten IsOnNetwork=true

// Find MSI files written to a removable disk
event_simpleName=MsiFileWritten IsOnRemovableDisk=true

// Investigate MSI file writes by a specific file operator SID (Windows only)
event_simpleName=MsiFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MSI files based on their context base file name
event_simpleName=MsiFileWritten ContextBaseFileName="update.msi"
```

---