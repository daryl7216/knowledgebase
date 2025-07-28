````markdown
## Crowdstrike Table: MachOFileWritten

### 1. Table Name
**MachOFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a MachO file. It is useful for investigations into file creation, data exfiltration, and malware drops across Linux, Windows, and macOS platforms.

---

### 3. Important Parameter Description

| Parameter           | Description                                       | Platforms Affected |
|---------------------|---------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was downloaded.      | All                |
| ContextBaseFileName | The base name of the context file.                | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/usr/local/bin/example_macho",
  "ContextBaseFileName": "example_macho"
}
````

---

### 5. Example Queries

```xql
// Find all MachO files written with a specific target file name
event_simpleName=MachOFileWritten TargetFileName="*example_macho*"

// Find MachO files based on their context base file name
event_simpleName=MachOFileWritten ContextBaseFileName="launcher"
```

---

```