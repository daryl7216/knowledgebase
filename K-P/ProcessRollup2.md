## Crowdstrike Table: ProcessRollup2

### 1. Table Name

**ProcessRollup2**

---

### 2. Table Usage

This event (often called "PR2" for short) is generated for a process that is running or has finished running on a host and contains information about that process. For every ProcessRollup2 event, there is a corresponding event that is generated when a process completes. The name of this event is EndOfProcess for Windows and Mac and TerminateProcess for Linux. This event is crucial for understanding process execution, parent-child relationships, and identifying suspicious process activities across ChromeOS, Windows, macOS, Linux, Falcon Container, and Android platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ParentBaseFileName | The base ImageFileName of the parent process. | Windows, macOS, Linux, Falcon Container |
| CommandLine | The command line used to create this process. May be empty in some circumstances. | All |
| ImageFileName | The full path to an executable (PE) file. For ProcessRollup2 events, this is the full path to the main executable for the created process. | All |
| ParentProcessId | The decimal representation of the parent process. | All |
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus. | All |
| AuthenticationId | Values: INVALID\_LUID (0), NETWORK\_SERVICE (996), LOCAL\_SERVICE (997), SYSTEM (999), RESERVED\_LUID\_MAX (1000). | Windows |
| IntegrityLevel | Values: LOW (0x00001000), MEDIUM (0x00002000), MEDIUM\_PLUS (0x00002100), HIGH (0x00003000), SYSTEM (0x00004000), PROTECTED (0x00005000). | Windows |
| TokenType | Values: INVALID\_TOKEN (0), PRIMARY\_TOKEN (1), IMPERSONATION\_TOKEN (2). | Windows |

---

### 4. Sample Log Example

```json
{
  "ParentBaseFileName": "explorer.exe",
  "CommandLine": "cmd.exe /c whoami",
  "ImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "ParentProcessId": 1234,
  "TargetProcessId": 5678,
  "AuthenticationId": 999,
  "IntegrityLevel": 12288,
  "TokenType": 1
}
```

---

### 5. Example Queries

```xql
// Find all ProcessRollup2 events
event_simpleName=ProcessRollup2

// Find processes launched by a specific parent process
event_simpleName=ProcessRollup2 ParentBaseFileName="powershell.exe"

// Find processes with a specific command line
event_simpleName=ProcessRollup2 CommandLine="*evil.exe*"

// Find processes based on their image file name
event_simpleName=ProcessRollup2 ImageFileName="*\\System32\\svchost.exe"

// Find processes with a specific integrity level (Windows only)
event_simpleName=ProcessRollup2 IntegrityLevel=16384 // SYSTEM integrity level

// Find processes with a specific authentication ID (Windows only)
event_simpleName=ProcessRollup2 AuthenticationId=997 // LOCAL_SERVICE

// Find processes with an impersonation token (Windows only)
event_simpleName=ProcessRollup2 TokenType=2
```

---
