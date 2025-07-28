## RegSystemConfigValueUpdate Table: System Registry Modification Event

### 1. Table Name
**RegSystemConfigValueUpdate**

---

### 2. Table Usage
This event is generated when a registry value related to system configuration or security is updated on a Windows host. Attackers often modify these registry values to disable security controls (like UAC or Windows Defender), establish persistence, or otherwise alter the system's behavior to their advantage. Monitoring these updates is crucial for detecting tampering and defense evasion techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RegObjectName | The full path of the registry key that was modified. | Windows Only |
| RegValueName | The name of the specific value within the key that was updated. | Windows Only |
| RegNumericValue/RegStringValue | The new data written to the registry value. This can be a number (`RegNumericValue`) or a string (`RegStringValue`). | Windows Only |
| RegOperationType | The type of registry operation performed, such as `1` (REG_SET_VALUE_KEY) or `2` (REG_DELETE_VALUE_KEY). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "\\REGISTRY\\MACHINE\\SYSTEM\\CurrentControlSet\\Services\\SharedAccess\\Parameters\\FirewallPolicy\\StandardProfile",
  "RegValueName": "EnableFirewall",
  "RegNumericValue": "0",
  "RegOperationType": "1"
}
```

---

### 5. Example Queries
```xql
// Find all system configuration registry value updates
event_simpleName=RegSystemConfigValueUpdate

// Hunt for attempts to disable the Windows Firewall by modifying the EnableFirewall value
event_simpleName=RegSystemConfigValueUpdate RegValueName="EnableFirewall" RegNumericValue=0

// Find modifications to the LSA configuration, which could be related to credential protection
event_simpleName=RegSystemConfigValueUpdate RegObjectName="*\\Lsa"

// Search for changes to the UAC (User Account Control) settings
event_simpleName=RegSystemConfigValueUpdate RegValueName="EnableLUA"

// Filter for only "Set Value" operations on system configuration keys
event_simpleName=RegSystemConfigValueUpdate RegOperationType=1
```
---