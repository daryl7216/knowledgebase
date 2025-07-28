## Crowdstrike Table: NewExecutableRenamed

### 1. Table Name

**NewExecutableRenamed**

---

### 2. Table Usage

This table is generated when an executable is renamed. This event is important for detecting potential attempts at masquerading malicious executables, evading detection, or altering system binaries on macOS and Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| SourceFileName | The file's original name. | All |
| TargetFileName | The resulting file name after the rename operation. | All |

---

### 4. Sample Log Example

```json
{
  "SourceFileName": "C:\\Windows\\System32\\svchost.exe",
  "TargetFileName": "C:\\Windows\\Temp\\temp_svchost.exe"
}
```

---

### 5. Example Queries

```xql
// Find all executable rename events
event_simpleName=NewExecutableRenamed

// Find executable rename events for a specific source file name
event_simpleName=NewExecutableRenamed SourceFileName="*powershell.exe"

// Find executable rename events where the target file name indicates a suspicious location
event_simpleName=NewExecutableRenamed TargetFileName="*\\Users\\Public\\*"
```

---