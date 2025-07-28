## ClassifiedModuleLoad Table: Classified Module Load Event

### 1. Table Name
**ClassifiedModuleLoad**

---

### 2. Table Usage
The ClassifiedModuleLoad event provides information about a module which has been loaded into a process's memory space. It is only generated for 'classified' module loads, that is module loads that are interesting enough for us to cloud.

---

### 3. Important Parameter Description

| Parameter         | Description                                                          | Platforms Affected |
|-------------------|----------------------------------------------------------------------|--------------------|
| TargetFileName    | The full path to the executable file for the target process.         | All                |
| IsOnNetwork       | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk.         | Windows Only       |
| FileOperatorSid   | Security Identifier (SID) of the file operator.                      | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                   | All                |

---
### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\System32\\example.dll",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "example.dll"
}
```
---
### 5. Example Queries
```xql
// Find all classified module loads for a specific target file name
event_simpleName=ClassifiedModuleLoad TargetFileName="*example.dll*"

// Find classified module loads from a network drive (Windows only)
event_simpleName=ClassifiedModuleLoad IsOnNetwork=true

// Find classified module loads from a removable disk (Windows only)
event_simpleName=ClassifiedModuleLoad IsOnRemovableDisk=true

// Investigate classified module loads by a specific file operator SID (Windows only)
event_simpleName=ClassifiedModuleLoad FileOperatorSid="S-1-5-21-12345-67890-*"

// Find classified module loads based on their context base file name
event_simpleName=ClassifiedModuleLoad ContextBaseFileName="module.dll"
```
---
