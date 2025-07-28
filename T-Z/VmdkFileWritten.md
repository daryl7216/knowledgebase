## VmdkFileWritten Table: File Written Event

### 1. Table Name
**VmdkFileWritten**

---

### 2. Table Usage
This table records events when a VMDK (Virtual Machine Disk) file is written. VMDK files are used by virtualization software such as VMware. Monitoring the creation of these files can be important for tracking the setup of virtual environments, potential data exfiltration where data is packaged into a virtual disk, or the introduction of unauthorized virtual machines into the environment.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |
| TemporaryFileName | The temporary file name used during the write operation, if any. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\VMs\\New_Machine.vmdk",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "New_Machine.vmdk",
  "TemporaryFileName": "C:\\VMs\\~tmp_vmdk_1a2b3c.tmp"
}
```

---

### 5. Example Queries
```xql
// Find all VMDK files written to disk
event_simpleName=VmdkFileWritten TargetFileName="*.vmdk"

// Find VMDK files written to a network drive (Windows only)
event_simpleName=VmdkFileWritten IsOnNetwork=true

// Find VMDK files written to a removable disk (Windows only)
event_simpleName=VmdkFileWritten IsOnRemovableDisk=true

// Investigate VMDK file writes by a specific file operator SID (Windows only)
event_simpleName=VmdkFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find VMDK files that were created using a temporary file (Windows only)
event_simpleName=VmdkFileWritten a.TemporaryFileName!=""
```
---
