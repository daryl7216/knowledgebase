## Crowdstrike Table: ModuleDetectInfo

### 1. Table Name

**ModuleDetectInfo**

---

### 2. Table Usage

This table identifies a scenario that would be a blocking prevention if more rigorous settings were active. This event is useful for understanding potential threats that were detected but not blocked due to current prevention policies.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus. For example, the TargetProcessId of a process that performed thread injection in an InjectedThread event. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| SHA256HashData | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field. | All |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 12345,
  "TargetFileName": "/tmp/potential_malware.dll",
  "SHA256HashData": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
}
```

---

### 5. Example Queries

```xql
// Find all module detection information events
event_simpleName=ModuleDetectInfo

// Find module detection events for a specific target process ID
event_simpleName=ModuleDetectInfo TargetProcessId=9876

// Find module detection events related to a specific file name
event_simpleName=ModuleDetectInfo TargetFileName="*suspicious_module.so"

// Find module detection events with a particular SHA256 hash
event_simpleName=ModuleDetectInfo SHA256HashData="0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
```

---