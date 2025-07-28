## PrivilegedProcessHandleFromUnsignedModule Table: Privileged Process Handle from Unsigned Module Event

### 1. Table Name
**PrivilegedProcessHandleFromUnsignedModule**

---

### 2. Table Usage
This event is generated when an unsigned module creates a handle to a privileged process on a Windows host. Attackers often use unsigned DLLs to inject into and access the memory of sensitive processes like LSASS for credential theft. Monitoring this event is critical for detecting attempts by untrusted code to access privileged process memory.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the unsigned module that created the handle. | Windows Only |
| FileSigningTime | The timestamp when the file was signed. For an unsigned module, this may be null or zero. | Windows Only |
| SignInfoFlags | A bitmask indicating the signature status of the module. A value of `512` (0x200) indicates the module has no signature. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.dll",
  "FileSigningTime": "0",
  "SignInfoFlags": "512"
}
```

---

### 5. Example Queries
```xql
// Find all instances of privileged handles being created by unsigned modules
event_simpleName=PrivilegedProcessHandleFromUnsignedModule

// Hunt for a specific unsigned module by name
event_simpleName=PrivilegedProcessHandleFromUnsignedModule ImageFileName="*\\evil.dll"

// Find all modules that are explicitly flagged as having no signature
event_simpleName=PrivilegedProcessHandleFromUnsignedModule SignInfoFlags=512

// Investigate unsigned modules creating handles from a temp directory
event_simpleName=PrivilegedProcessHandleFromUnsignedModule ImageFileName="*\\Temp\\*"
```
---