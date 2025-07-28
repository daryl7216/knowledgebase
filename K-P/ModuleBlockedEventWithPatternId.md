## Crowdstrike Table: ModuleBlockedEventWithPatternId

### 1. Table Name

**ModuleBlockedEventWithPatternId**

---

### 2. Table Usage

This event adds a potential reason for blocking (as a PatternId) to module blocking information. This is useful for understanding specific detection patterns that led to a module being blocked across Windows, macOS, Linux, and Falcon Container platforms. 

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus.  | All |
| SHA256HashData | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field.  | All |
| TargetFileName | The resulting file name that was downloaded. | All |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 1234,
  "SHA256HashData": "0x56f3097c4d5bf4c7cffef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8",
  "TargetFileName": "/usr/local/bin/blocked_module"
}
```

---

### 5. Example Queries

```xql
// Find all ModuleBlockedEventWithPatternId events
event_simpleName=ModuleBlockedEventWithPatternId

// Find blocked module events for a specific target process ID
event_simpleName=ModuleBlockedEventWithPatternId TargetProcessId=5678

// Find blocked module events by a specific SHA256 hash
event_simpleName=ModuleBlockedEventWithPatternId SHA256HashData="0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a"

// Find blocked module events by a specific target file name
event_simpleName=ModuleBlockedEventWithPatternId TargetFileName="*malicious_library.dll"
```

---
