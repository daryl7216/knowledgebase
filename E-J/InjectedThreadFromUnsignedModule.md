## InjectedThreadFromUnsignedModule Table: Thread Injection from Unsigned Module Event

### 1. Table Name
**InjectedThreadFromUnsignedModule**

---

### 2. Table Usage
This event is generated when a thread is injected into a process from an unsigned module on a Windows host. This is a highly suspicious activity and a common technique used by malware to execute malicious code within the address space of a legitimate process, thereby evading detection. This event combines details about the injection and the signature status of the source module.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the process into which the thread was injected. | Windows Only |
| TargetThreadModule | The module in the target process where the injected thread resides. | Windows Only |
| InjectedThreadFlag | A bitmask of flags providing context about the injected thread, such as `512` (START_ADDRESS_PRIVATE_MEM) indicating the thread starts in private memory. | Windows Only |
| UserThread | A flag indicating if the thread is a user thread. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "3456",
  "TargetThreadModule": "C:\\Windows\\System32\\ntdll.dll",
  "InjectedThreadFlag": "512",
  "UserThread": "1"
}
```
---
### 5. Example Queries
```xql
// Find all thread injections from unsigned modules
event_simpleName=InjectedThreadFromUnsignedModule

// Hunt for injections into a specific target process ID
event_simpleName=InjectedThreadFromUnsignedModule TargetProcessId="1234"

// Find injections where the thread starts in private memory (InjectedThreadFlag=512), which is highly suspicious
event_simpleName=InjectedThreadFromUnsignedModule InjectedThreadFlag=512

// Investigate which modules in the target process are associated with the injected thread
event_simpleName=InjectedThreadFromUnsignedModule | table TargetProcessId, TargetThreadModule
```
---