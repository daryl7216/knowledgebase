## CreateService Table: Service Creation Event

### 1. Table Name
**CreateService**

---

### 2. Table Usage
Generated when a Windows service is created.

---

### 3. Important Parameter Description

| Parameter         | Description                                                                                                                                                                                                          |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ServiceDisplayName | The display name of the service.                                                                                                                                                                                     |
| ServiceImagePath  | The path to the service binary.                                                                                                                                                                                      |
| ServiceType       | The type of service. Values: SERVICE_KERNEL_DRIVER (1) SERVICE_FILE_SYSTEM_DRIVER (2) SERVICE_ADAPTER (4) SERVICE_RECOGNIZER_DRIVER (8) SERVICE_WIN32_OWN_PROCESS (16) SERVICE_WIN32_SHARE_PROCESS (32) SERVICE_INTERACTIVE_PROCESS (256) |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 1234,
  "ContextThreadId": 5678,
  "TreeId": "abc-123",
  "ServiceStart": 2,
  "ServiceErrorControl": 1,
  "ServiceType": 16,
  "ServiceGroup": "",
  "ServiceImagePath": "C:\\Windows\\System32\\myservice.exe",
  "ServiceDisplayName": "My Custom Service"
}
```
---
### 5. Example Queries
```xql
// Find all created services with a specific display name
event_simpleName=CreateService ServiceDisplayName="My Custom Service"

// Find services created with a specific image path
event_simpleName=CreateService ServiceImagePath="C:\\Program Files\\MyApplication\\service.exe"

// Find services of a specific type (e.g., SERVICE_WIN32_OWN_PROCESS)
event_simpleName=CreateService ServiceType=16
```
---
