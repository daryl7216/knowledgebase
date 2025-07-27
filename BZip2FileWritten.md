## BZip2FileWritten Table: BZip2 File Written Event

### 1. Table Name
**BZip2FileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a BZip2 file. BZip2 files are common compressed archive formats, and monitoring their creation can help detect data compression activities, exfiltration attempts, or malware packaging.

---

### 3. Important Parameter Description

| Parameter                  | Description                                                                                                                                                                                                                                                                                                 | Platforms Affected |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| ContextTimeStamp           | System time of event creation.                                                                                                                                                                                                                                                                    | All                |
| ContextProcessId           | UPID of process originating this event.                                                                                                                                                                                                                                                           | All                |
| ContextThreadId            | UTID of thread originating this event                                                                                                                                                                                                                                                             | All                |
| TreeId                     | If this event is part of a detection tree, the tree ID it is part of.                                                                                                                                                                                                                             | All                |
| FileIdentifier             |                                                                                                                                                                                                                                                                                                   | All                |
| TargetFileName             | The resulting file name that was downloaded                                                                                                                                                                                                                                                       | All                |
| IsOnRemovableDisk          | If true, it means this file was located on a removable disk.                                                                                                                                                                                                                                      | All                |
| UserName                   |                                                                                                                                                                                                                                                                                                   | All                |
| Size                       |                                                                                                                                                                                                                                                                                                   | All                |
| DiskParentDeviceInstanceId |                                                                                                                                                                                                                                                                                                   | All                |
| FileCategory               | Values: OTHER (0) ARCHIVES (1) OFFICE_DOCUMENTS (2) MULTIMEDIA_FILES (3) DESIGN_FILES (4) SOURCE_CODE (5) EXECUTABLE_FILES (6) VIRTUAL_MACHINE_FILES (7) EMAIL_FILES (8) DATA_AND_LOGS (9) ENCRYPTED (10)                                                                                          | All                |
| VolumeSessionUUID          |                                                                                                                                                                                                                                                                                                   | All                |
| FileSourcePath             |                                                                                                                                                                                                                                                                                                   | All                |
| MipInformation             |                                                                                                                                                                                                                                                                                                   | All                |
| SHA256HashData             | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field. Values: STATIC_SHA256_DOPPELGANGING (0x56f3097c4d5bf4c7c ff ef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8) STATIC_SHA256_SYSTEM (0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a) | All                |
| ContextBaseFileName        |                                                                                                                                                                                                                                                                                                   | All                |
| IsOnNetwork                | Set to true if the relevant file listed in the event is on a network drive. False otherwise.                                                                                                                                                                                                    | Windows Only       |
| AuthenticationId           | Values: INVALID_LUID (0) SYSTEM (999) NETWORK_SERVICE (996) LOCAL_SERVICE (997) RESERVED_LUID_MAX (1000)                                                                                                                                                                                           | Windows Only       |
| TokenType                  | Values: INVALID_TOKEN (0) PRIMARY_TOKEN (1) IMPERSONATION_TOKEN (2)                                                                                                                                                                                                                               | Windows Only       |
| FileObject                 |                                                                                                                                                                                                                                                                                                   | Windows Only       |
| MajorFunction              |                                                                                                                                                                                                                                                                                                   | Windows Only       |
| MinorFunction              |                                                                                                                                                                                                                                                                                                   | Windows Only       |
| IrpFlags                   |                                                                                                                                                                                                                                                                                                   | Windows Only       |
| OperationFlags             |                                                                                                                                                                                                                                                                                                   | Windows Only       |
| FileEcpBitmask             | Values: ECP_SRV_OPEN (0x00000001) ECP_CS_SENSOR_OPEN (0x00000002) ECP_CS_SENSOR_FILE_CHARACTERISTICS (0x00000004)                                                                                                                                                                                | Windows Only       |
| TemporaryFileName          | Values: HASH_IS_VALID (0x00000000) HASH_FAILED (0x00000001)                                                                                                                                                                                                                                       | Windows Only       |
| FileWrittenFlags           | HASH_ABORTED_TOO_LARGE (0x00000002) PAGING_WRITE (0x00000004)                                                                                                                                                                                                                                     | Windows Only       |
| FileOperatorSid            |                                                                                                                                                                                                                                                                                                   | Windows Only       |
| MountNamespaceUniqueId     |                                                                                                                                                                                                                                                                                                   | Linux Only         |
| RawProcessId               | The operating system's internal PID. For matching, use the UPID fields which guarantee a unique process identifier                                                                                                                                                                                | Linux Only         |
| RUID                       | The real UID for a unix style process.                                                                                                                                                                                                                                                            | Linux Only         |
| RGID                       | The real GID for a unix style process.                                                                                                                                                                                                                                                            | Linux Only         |
| FsMagic                    |                                                                                                                                                                                                                                                                                                   | Linux Only         |
| DeviceMountCounter         |                                                                                                                                                                                                                                                                                                   | Linux Only         |
| DeviceId                   |                                                                                                                                                                                                                                                                                                   | Linux Only         |
| FileSerialNumber           |                                                                                                                                                                                                                                                                                                   | Linux Only         |
| ChangeTime                 | Unix change time, indicating last status change or file modification. NOTE: This is the only timestamp which is not modifiable, unlike the other file timestamps.                                                                                                                                  | Linux Only         |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "1701388800000",
  "ContextProcessId": "12345",
  "ContextThreadId": "67890",
  "TreeId": "abc-123-def-456",
  "FileIdentifier": "123456789",
  "TargetFileName": "/home/user/documents/report.bz2",
  "IsOnRemovableDisk": false,
  "UserName": "testuser",
  "Size": 51200,
  "DiskParentDeviceInstanceId": "DEV-001",
  "FileCategory": "ARCHIVES",
  "VolumeSessionUUID": "vol-uuid-1",
  "FileSourcePath": "/tmp/source_data.txt",
  "MipInformation": "some_mip_info",
  "SHA256HashData": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2",
  "ContextBaseFileName": "report.bz2",
  "IsOnNetwork": false,
  "AuthenticationId": 999,
  "TokenType": "PRIMARY_TOKEN",
  "FileObject": "0x123abc",
  "MajorFunction": 1,
  "MinorFunction": 0,
  "IrpFlags": 0,
  "OperationFlags": 0,
  "FileEcpBitmask": "0x00000001",
  "TemporaryFileName": "temp_report.tmp",
  "FileWrittenFlags": "0x00000000",
  "FileOperatorSid": "S-1-5-21-12345-67890-..."
}
```
---
### 5. Example Queries
```xql
// Find all BZip2 files written on any platform
event_simpleName=BZip2FileWritten

// Find BZip2 files written to a removable disk
event_simpleName=BZip2FileWritten IsOnRemovableDisk=true

// Find BZip2 files written to a network drive (Windows only)
event_simpleName=BZip2FileWritten IsOnNetwork=true

// Investigate BZip2 file writes by a specific username
event_simpleName=BZip2FileWritten UserName="*malicious_user*"

// Find BZip2 files with a specific SHA256 hash
event_simpleName=BZip2FileWritten SHA256HashData="a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2"

// Find BZip2 files categorized as archives and created by a specific process ID
event_simpleName=BZip2FileWritten FileCategory=ARCHIVES ContextProcessId="12345"
```
---