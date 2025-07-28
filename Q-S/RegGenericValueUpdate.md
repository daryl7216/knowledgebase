## RegGenericValueUpdate Table: Registry Value Update Event

### 1. Table Name
**RegGenericValueUpdate**

---

### 2. Table Usage
This event is generated when a registry value is updated on a Windows host. It serves as a general-purpose event for tracking registry modifications. Monitoring these updates is essential for threat hunting, as attackers frequently alter the registry to establish persistence, disable security features, store configuration data, or execute malicious code.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RegObjectName | The full path of the registry key that was modified. | Windows Only |
| RegValueName | The name of the specific value within the key that was updated. | Windows Only |
| RegOperationType | The type of registry operation performed, such as `1` (Set Value) or `2` (Delete Value). | Windows Only |
| RegStringValue (if applicable) | The string data written to the registry value. This is populated when the value type is a string. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "\\REGISTRY\\USER\\S-1-5-21-12345\\Software\\Microsoft\\Windows\\CurrentVersion\\Run",
  "RegValueName": "MaliciousUpdater",
  "RegOperationType": "1",
  "RegStringValue": "C:\\Users\\Public\\evil.exe"
}
```

---

### 5. Example Queries
```xql
// Find all generic registry value updates
event_simpleName=RegGenericValueUpdate

// Hunt for modifications to common persistence keys like Run or RunOnce
event_simpleName=RegGenericValueUpdate RegObjectName="*\\CurrentVersion\\Run*"

// Find instances where a specific registry value is being set or modified
event_simpleName=RegGenericValueUpdate RegValueName="*Backdoor*"

// Search for registry values being set to execute files from suspicious locations
event_simpleName=RegGenericValueUpdate RegStringValue="*\\Temp\\*" OR RegStringValue="*\\Users\\Public\\*"

// Filter for only "Set Value" operations
event_simpleName=RegGenericValueUpdate RegOperationType=1
```
---