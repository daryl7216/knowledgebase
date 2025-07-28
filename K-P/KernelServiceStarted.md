## KernelServiceStarted Table: Kernel Service Start Event

### 1. Table Name
**KernelServiceStarted**

---

### 2. Table Usage
This event is generated when a kernel service (driver) is started on a Windows host, either locally or remotely. Attackers may install and start malicious drivers to gain kernel-level privileges, which allows them to hide their presence (rootkit functionality) or tamper with the operating system and security products. Monitoring this event is critical for detecting the loading of potentially malicious or unauthorized drivers.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ServiceDisplayName | The display name of the kernel service that was started. | Windows Only |
| ImageFileName | The full path to the driver file (e.g., `.sys` file) associated with the service. | Windows Only |
| ImageBaseName | The base name of the driver file. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to start the service. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ServiceDisplayName": "Malicious Kernel Driver",
  "ImageFileName": "C:\\Windows\\System32\\drivers\\evil.sys",
  "ImageBaseName": "evil.sys",
  "RpcClientProcessId": "784"
}
```

---

### 5. Example Queries
```xql
// Find all kernel service start events
event_simpleName=KernelServiceStarted

// Hunt for drivers being loaded from unusual locations
event_simpleName=KernelServiceStarted ImageFileName!="C:\\Windows\\System32\\drivers\\*"

// Find kernel services with suspicious display names
event_simpleName=KernelServiceStarted ServiceDisplayName="*rootkit*" OR ServiceDisplayName="*hide*"

// Investigate which process is responsible for starting a specific kernel service
event_simpleName=KernelServiceStarted ServiceDisplayName="MySuspiciousDriver" | table RpcClientProcessId, ImageFileName
```
---