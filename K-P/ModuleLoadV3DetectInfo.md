## Crowdstrike Table: ModuleLoadV3DetectInfo

### 1. Table Name

**ModuleLoadV3DetectInfo**

---

### 2. Table Usage

This table provides additional information for detections from the Module Load V3 template. It is useful for understanding the details of module loading events, including file and process information, and digital signature details, specifically on Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to an executable (PE) file. For ProcessRollup2 events, this is the full path to the main executable for the created process. | Windows |
| CommandLine | The command line used to create this process. May be empty in some circumstances. | Windows |
| TargetFileName | The resulting file name that was downloaded. | Windows |
| SHA256HashData | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field. | Windows |
| SignInfoFlags | The image had a self-signed certificate. Values indicate various signature flags such as `SIGNATURE_FLAG_SELF_SIGNED` (0x00000001), `SIGNATURE_FLAG_MS_SIGNED` (0x00000002), etc. | Windows |

---

### 4\. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\drivers\\mymodule.sys",
  "CommandLine": "load driver mymodule.sys",
  "TargetFileName": "mymodule.sys",
  "SHA256HashData": "0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
  "SignInfoFlags": 2 // SIGNATURE_FLAG_MS_SIGNED
}
```

---

### 5. Example Queries

```xql
// Find all ModuleLoadV3DetectInfo events
event_simpleName=ModuleLoadV3DetectInfo

// Find ModuleLoadV3DetectInfo events for a specific image file name
event_simpleName=ModuleLoadV3DetectInfo ImageFileName="*\\temp\\malicious.dll"

// Find ModuleLoadV3DetectInfo events with a specific command line
event_simpleName=ModuleLoadV3DetectInfo CommandLine="*powershell.exe -exec bypass*"

// Find ModuleLoadV3DetectInfo events by a specific SHA256 hash
event_simpleName=ModuleLoadV3DetectInfo SHA256HashData="0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"

// Find ModuleLoadV3DetectInfo events where the module has a self-signed certificate
event_simpleName=ModuleLoadV3DetectInfo SignInfoFlags=1
```

---