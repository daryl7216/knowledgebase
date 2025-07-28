## Crowdstrike Table: ModuleBlockedEvent

### 1. Table Name
**ModuleBlockedEvent**

---

### 2. Table Usage
This table is sent to inform the cloud that a given module (identified by hash) was blocked. [cite: 1] This event is useful for understanding when specific executable modules or libraries are prevented from loading or executing, which can indicate policy enforcement, malware prevention, or application control actions across Windows, macOS, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter       | Description                                                                                                                                                                                                                                                                                                              | Platforms Affected |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field represents the ID of the process responsible for the activity of the event in focus. | All                |
| SHA256HashData  | The SHA256 hash of a file. In most cases, it's the hash of the file referred to by the ImageFileName field.  | All                |
| TargetFileName  | The resulting file name that was downloaded.                                                                                                                                                                                                                                                                    | All                |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 1234,
  "SHA256HashData": "0x56f3097c4d5bf4c7cffef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8",
  "TargetFileName": "C:\\Program Files\\BlockedApp\\blocked_module.dll"
}
```

---

### 5. Example Queries

```xql
// Find all blocked module events
event_simpleName=ModuleBlockedEvent

// Find blocked module events for a specific target process ID
event_simpleName=ModuleBlockedEvent TargetProcessId=5678

// Find blocked module events by a specific SHA256 hash
event_simpleName=ModuleBlockedEvent SHA256HashData="0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a"

// Find blocked module events by a specific target file name
event_simpleName=ModuleBlockedEvent TargetFileName="*malicious_library.so"
```

---