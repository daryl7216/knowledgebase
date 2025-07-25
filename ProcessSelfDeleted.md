## Crowdstrike Table: ProcessSelfDeleted

### 1. Table Name
**ProcessSelfDeleted**

---

### 2. Table Usage
This event indicates when a process self-delete situation is detected. It will be thrown when a process deletes a file that was previously loaded as the primary module in an ancestor process. If the self-delete was accomplished via injecting into an already running process, ContextProcessId will be the injector as well.

---

### 3. Important Parameter Description

| Parameter                    | Description                                                            |
|------------------------------|------------------------------------------------------------------------|
| SourceProcessId              | UPID of creating process.                                              |
| ContextProcessId             | UPID of process originating this event.                                |
| TargetProcessId              | The unique ID of a target process (in decimal, non-hex format).        |
| ImageFileName                | The full path to an executable (PE) file.                              |
| SHA256HashData               | The SHA256 hash of a file.                                             |
| TreeId                       | If this event is part of a detection tree, the tree ID it is part of.  |
| ContextTimeStamp             | System time of event creation.                                         |
---

### 4. Sample Log Example

```json
{
  "event_platform": "Windows",
  "event_simpleName": "ProcessSelfDeleted",
  "aid": "4a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d",
  "cid": "2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d",
  "event_type": "ProcessActivity",
  "timestamp": 1678886400000,
  "local_timestamp": "2023-03-15T10:00:00.000Z",
  "device_id": "Device-ABC123XYZ",
  "computer_name": "WORKSTATION001",
  "user_sid": "S-1-5-21-1234567890-123456789-123456789-1001",
  "user_name": "DOMAIN\\User.Name",
  "process_id": 12345,
  "process_name": "malicious_script.exe",
  "process_path": "C:\\Users\\User.Name\\Downloads\\malicious_script.exe",
  "image_hash_md5": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6",
  "image_hash_sha256": "abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789",
  "parent_process_id": 9876,
  "parent_process_name": "cmd.exe",
  "parent_process_path": "C:\\Windows\\System32\\cmd.exe",
  "command_line": "\"C:\\Users\\User.Name\\Downloads\\malicious_script.exe\" --cleanup",
  "target_filename": "malicious_script.exe",
  "detection_name": "Suspicious_Process_Self_Deletion",
  "tactic": "Defense Evasion",
  "technique": "Process Tampering",
  "severity": "High",
  "description": "A process executed from a suspicious location self-deleted.",
  "sensor_version": "6.29.12345.0"
  ...
}
```

---

### 5. Example Queries

```xql
// Find all self deleting process events
#event_simpleName="ProcessSelfDeleted"

// Find all events of processes that were self deleted with severity high or above
#event_simpleName:"ProcessSelfDeleted" AND (severity:"High" OR severity:"Critical")

// Find all self deleting process events but exclude common paths for self deleting processes
#event_simpleName:"ProcessSelfDeleted" AND NOT process_path:"C\:\\Windows\\System32\\*"AND NOT process_path:"C\:\\Program Files\\*"

// Find all deleted by self process events that contain "cleanup" in the command line
#event_simpleName:"ProcessSelfDeleted" AND command_line:"*cleanup*"
```

---
