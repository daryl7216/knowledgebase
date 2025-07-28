## Crowdstrike Table: MbrOverwriteRawDetectInfo

### 1. Table Name
**MbrOverwriteRawDetectInfo**

---

### 2. Table Usage
This table provides raw detection information related to Master Boot Record (MBR) overwrites. It is emitted on Windows platforms when an MBR overwrite is detected, providing details about the process involved and the affected volume. This information is crucial for analyzing rootkit activity and system integrity compromises.

---

### 3. Important Parameter Description

| Parameter   | Description                                                                                                                                                                                                                                  | Platforms Affected |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| ImageFileName | The full path to an executable (PE) file. For ProcessRollup2 events, this is the full path to the main executable for the created process.                                                                                                                                                                                                                                                          | Windows            |
| VolumeName  | The name of the volume affected by the MBR overwrite.                                                                                                                                                                                                                                                                                                                                                | Windows            |
| CommandLine | The command line used to create this process. May be empty in some circumstances.                                                                                                                                                                                                                                                                                                                       | Windows            |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\malicious.exe",
  "VolumeName": "\\\\.\\PhysicalDrive0",
  "CommandLine": "malicious.exe -install"
}
```

---

### 5. Example Queries

```xql
// Find all MBR overwrite detection events
event_simpleName=MbrOverwriteRawDetectInfo

// Find MBR overwrite events by a specific image file name
event_simpleName=MbrOverwriteRawDetectInfo ImageFileName="*\\malicious.exe"

// Find MBR overwrite events on a specific volume
event_simpleName=MbrOverwriteRawDetectInfo VolumeName="\\\\.\\PhysicalDrive0"

// Find MBR overwrite events with a specific command line
event_simpleName=MbrOverwriteRawDetectInfo CommandLine="* -install*"
```

---
