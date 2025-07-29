## JavaInjectedThread Table: Java Thread Injection Event

### 1. Table Name
**JavaInjectedThread**

---

### 2. Table Usage
This event is generated when a Java application injects a thread into another process on a Windows host. While this can be a legitimate behavior for some applications, it is also a technique used by malware, particularly in exploits targeting Java vulnerabilities, to execute malicious code within the context of another process. Monitoring this event is important for detecting potential Java-based exploits and malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the process into which the Java thread was injected. | Windows Only |
| InjectedThreadFlag | A bitmask of flags providing context about the injected thread, such as if the start address is in a known module or private memory. | Windows Only |
| UserThread | Indicates whether the thread is a user-mode thread. | Windows Only |
| ThreadStartBytes | The initial bytes of code at the start address of the injected thread. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "4560",
  "InjectedThreadFlag": "512",
  "UserThread": "1",
  "ThreadStartBytes": "fc4883e4f0e8..."
}
```
---

### 5. Example Queries
```xql
// Find all instances of Java thread injection
event_simpleName=JavaInjectedThread

// Hunt for Java injecting a thread into a specific process, like a browser or system process
event_simpleName=JavaInjectedThread TargetProcessId="1234"

// Find injections where the thread starts in private memory (InjectedThreadFlag=512), which is highly suspicious
event_simpleName=JavaInjectedThread InjectedThreadFlag=512

// Investigate the initial bytes of an injected thread for known malicious shellcode patterns
event_simpleName=JavaInjectedThread ThreadStartBytes="*fc4883*"
```
---