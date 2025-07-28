## LsassHandleFromUnsignedModule Table: LSASS Handle from Unsigned Module Event

### 1. Table Name
**LsassHandleFromUnsignedModule**

---

### 2. Table Usage
This event is generated when a handle is created to the LSASS (Local Security Authority Subsystem Service) process from an unsigned module. LSASS stores credentials for active user sessions. Attackers, particularly credential theft tools like Mimikatz, often attempt to access LSASS memory to dump these credentials. A handle being opened by an unsigned (and therefore untrusted) module is a very strong indicator of a credential dumping attempt.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the unsigned module (e.g., a DLL) that created the handle to LSASS. | Windows Only |
| TargetProcessId | The unique ID of the process to which the handle was created. For this event, it will be the process ID of LSASS. | Windows Only |
| FileSigningTime | The timestamp of the module's digital signature. For an unsigned module, this will typically be zero or null. | Windows Only |
| SignInfoFlags | A bitmask indicating the signature status of the module. A value of `512` (0x200) indicates the module has no signature. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\mimikatz.dll",
  "TargetProcessId": "712",
  "FileSigningTime": "0",
  "SignInfoFlags": "512"
}
```
---

### 5. Example Queries
```xql
// Find all instances of an unsigned module creating a handle to LSASS
event_simpleName=LsassHandleFromUnsignedModule

// Hunt for specific unsigned modules by name that are accessing LSASS
event_simpleName=LsassHandleFromUnsignedModule ImageFileName="*\\mimikatz.dll"

// Find all modules explicitly flagged as having no signature that are accessing LSASS
event_simpleName=LsassHandleFromUnsignedModule SignInfoFlags=512

// Investigate unsigned modules from temporary directories that are accessing LSASS
event_simpleName=LsassHandleFromUnsignedModule ImageFileName IN ("*\\Temp\\*", "*\\Users\\Public\\*")
```
---