## UnsignedModuleLoad Table: Unsigned Module Load Event

### 1. Table Name
**UnsignedModuleLoad**

***

### 2. Table Usage
This event contains information about an unsigned module that was loaded into a target process. It combines relevant information from the ImageHash and SignInfoWithCertAndContext events for the unsigned module. Monitoring this event is important because malware often uses unsigned modules to execute malicious code.

***

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the unsigned module (e.g., DLL, EXE) that was loaded. | Windows Only |
| MappedFromUserMode | Indicates whether this module is loaded into user memory space (non-zero) or kernel space (zero). | Windows Only |
| ModuleCharacteristics | A bitmask field indicating optional characteristics of the loaded module, such as whether it's an executable or a DLL. | Windows Only |
| TargetProcessId | The unique ID of the process into which the unsigned module was loaded. | Windows Only |
| SHA256HashData | The SHA256 hash of the unsigned module file. | Windows Only |

***

### 4. Sample Log Example
```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.dll",
  "MappedFromUserMode": 1,
  "ModuleCharacteristics": "8194",
  "TargetProcessId": "4321",
  "SHA256HashData": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2"
}
```

---

### 5. Example Queries
```xql
// Find all unsigned modules loaded by a specific process
event_simpleName=UnsignedModuleLoad TargetProcessId="1234"

// Search for a specific unsigned DLL being loaded
event_simpleName=UnsignedModuleLoad ImageFileName="*evil.dll"

// Hunt for unsigned modules using their SHA256 hash
event_simpleName=UnsignedModuleLoad SHA256HashData="a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2"

// Find all unsigned modules loaded into user memory space
event_simpleName=UnsignedModuleLoad MappedFromUserMode=1
```
---
