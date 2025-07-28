## RegCrowdstrikeValueUpdate Table: Crowdstrike Registry Value Update Event

### 1. Table Name
**RegCrowdstrikeValueUpdate**

---

### 2. Table Usage
This event is generated when a registry value related to the CrowdStrike sensor's configuration is updated on a Windows host. Monitoring these events is critical for detecting potential sensor tampering, misconfigurations, or attempts by adversaries to disable or bypass security controls.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RegObjectName | The full path of the CrowdStrike-related registry key that was modified. | Windows Only |
| RegValueName | The name of the specific value within the key that was updated. | Windows Only |
| RegOperationType | The type of registry operation performed, such as `1` (Set Value) or `2` (Delete Value). | Windows Only |
| RegStringValue (if applicable) | The string data written to the registry value. This is populated when the value type is a string. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "\\REGISTRY\\MACHINE\\SYSTEM\\CurrentControlSet\\Services\\CSAgent\\Sim",
  "RegValueName": "Trace",
  "RegOperationType": "1",
  "RegStringValue": "none"
}
```
---

### 5. Example Queries
```xql
// Find all updates to CrowdStrike-related registry values
event_simpleName=RegCrowdstrikeValueUpdate

// Hunt for modifications to specific CrowdStrike sensor configuration values
event_simpleName=RegCrowdstrikeValueUpdate RegValueName="ProvisioningToken"

// Find any deletions of CrowdStrike registry values
event_simpleName=RegCrowdstrikeValueUpdate RegOperationType=2

// Search for specific strings being written to CrowdStrike registry keys
event_simpleName=RegCrowdstrikeValueUpdate RegStringValue="*disabled*"
```
---