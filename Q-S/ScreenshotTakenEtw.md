## ScreenshotTakenEtw Table: Screenshot Taken Event

### 1. Table Name
**ScreenshotTakenEtw**

---

### 2. Table Usage
This event is generated when a partial or full screenshot is taken on a Windows host. Malware and attackers often take screenshots to gather sensitive information from a user's desktop, such as passwords, personal data, or confidential documents. Monitoring this event is crucial for detecting reconnaissance and information theft activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file that took the screenshot. | Windows Only |
| CommandLine | The command line used to launch the process that took the screenshot. | Windows Only |
| UserName | The name of the user who was active when the screenshot was taken. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\SnippingTool.exe",
  "CommandLine": "SnippingTool.exe",
  "UserName": "CORP\\jsmith"
}
```
---
### 5. Example Queries
```xql
// Find all screenshot events
event_simpleName=ScreenshotTakenEtw

// Find screenshots taken by a specific application
event_simpleName=ScreenshotTakenEtw ImageFileName="*\\SnippingTool.exe"

// Hunt for screenshots taken by processes running from unusual locations, like a Temp folder
event_simpleName=ScreenshotTakenEtw ImageFileName="*\\Temp\\*"

// Find screenshots taken by a specific user
event_simpleName=ScreenshotTakenEtw UserName="*admin*"

// Look for screenshots taken by suspicious or unknown processes
event_simpleName=ScreenshotTakenEtw ImageFileName!="C:\\Windows\\System32\\SnippingTool.exe"
```