## KernelModelLoadImage Table: Kernel Module Load Event

### 1. Table Name
**KernelModelLoadImage**

---

### 2. Table Usage
This event is generated when a kernel module (driver) is loaded on a Windows host. Monitoring the loading of kernel modules is critical for security, as attackers often use malicious or vulnerable drivers (rootkits) to escalate privileges, hide their presence, and gain deep control over the operating system.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the kernel module (driver file, typically `.sys`) that was loaded. | Windows Only |
| ImageModuleName | The name of the module. | Windows Only |
| ImageModuleAuthor | The author of the module, as specified in its properties. | Windows Only |
| ImageModuleDescription | A description of the module, as specified in its properties. | Windows Only |
| ModuleInitFlags | A bitmask field indicating the initialization flags of the module. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\drivers\\evil_rootkit.sys",
  "ImageModuleName": "evil_rootkit",
  "ImageModuleAuthor": "Malicious Actor",
  "ImageModuleDescription": "Suspicious Kernel Driver",
  "ModuleInitFlags": "0"
}
```

---

### 5. Example Queries
```xql
// Find all kernel module load events
event_simpleName=KernelModelLoadImage

// Hunt for kernel modules loaded from non-standard directories like Temp or user profiles
event_simpleName=KernelModelLoadImage ImageFileName IN ("*\\Temp\\*", "*\\Users\\*")

// Find kernel modules with a suspicious name
event_simpleName=KernelModelLoadImage ImageModuleName="*rootkit*" OR ImageModuleName="*hide*"

// Search for modules with a suspicious description or author
event_simpleName=KernelModelLoadImage ImageModuleDescription="*bypass*" OR ImageModuleAuthor="*hacker*"

// Investigate a specific driver being loaded
event_simpleName=KernelModelLoadImage ImageFileName="*\\evil_driver.sys"
```
---