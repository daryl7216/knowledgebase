## DmpFileWritten Table: Dump File Written Event

### 1. Table Name
**DmpFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a dump file. PidFromCommandLine will be enhanced in some cases to extract out the pid from the commandline of the writing process if possible. This event is useful for investigations into memory dumping, potential credential theft, or debugging activities.

---

### 3. Important Parameter Description

| Parameter                | Description                                                                                                                                                       | Platforms Affected |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| ContextTimeStamp         | System time of event creation.                                                                                                                                    | All                |
| ContextProcessId         | UPID of process originating this event.                                                                                                                           | All                |
| ContextThreadId          | UTID of thread originating this event                                                                                                                             | All                |
| TreeId                   | If this event is part of a detection tree, the tree ID it is part of.                                                                                             | All                |
| FileObject               | The file object representing the written dump file.                                                                                                               | Windows Only       |
| MajorFunction            | The major function code of the IRP (I/O Request Packet).                                                                                                          | Windows Only       |
| MinorFunction            | The minor function code of the IRP.                                                                                                                               | Windows Only       |
| IrpFlags                 | Flags associated with the IRP.                                                                                                                                    | Windows Only       |
| OperationFlags           | Flags indicating specific operation characteristics.                                                                                                              | Windows Only       |
| Size                     | The size of the written dump file in bytes.                                                                                                                       | All                |
| DiskParentDeviceInstanceId | The device instance ID of the parent disk device.                                                                                                                 | All                |
| FileIdentifier           | A unique identifier for the file.                                                                                                                                 | All                |
| TargetFileName           | The resulting file name of the dump file that was written.                                                                                                        | All                |
| IsOnNetwork              | Set to true if the relevant file listed in the event is on a network drive. False otherwise.                                                                      | Windows Only       |
| IsOnRemovableDisk        | If true, it means this file was located on a removable disk.                                                                                                      | All                |
| AuthenticationId         | Values: INVALID_LUID (0) NETWORK_SERVICE (996) LOCAL_SERVICE (997) SYSTEM (999) RESERVED_LUID_MAX (1000)                                                          | Windows Only       |
| UserName                 | The name of the user who performed the dump file write operation.                                                                                                 | All                |
| FileEcpBitmask           | Values: ECP_SRV_OPEN (0x00000001) ECP_CS_SENSOR_OPEN (0x00000002) ECP_CS_SENSOR_FILE_CHARACTERISTICS (0x00000004)                                                 | Windows Only       |
| FileWrittenFlags         | Values: HASH_IS_VALID (0x00000000) HASH_FAILED (0x00000001) HASH_ABORTED_TOO_LARGE (0x00000002) PAGING_WRITE (0x00000004)                                         | Windows Only       |
| FileOperatorSid          | Security Identifier (SID) of the file operator.                                                                                                                   | Windows Only       |
| FileCategory             | Values: OTHER (0) ARCHIVES (1) OFFICE_DOCUMENTS (2) MULTIMEDIA_FILES (3) DESIGN_FILES (4) SOURCE_CODE (5) EXECUTABLE_FILES (6) VIRTUAL_MACHINE_FILES (7) EMAIL_FILES (8) DATA_AND_LOGS (9) ENCRYPTED (10) | All                |
| VolumeSessionUUID        | The UUID of the volume session where the file was written.                                                                                                        | macOS Only         |
| FileSourcePath           | The source path of the file if it was copied or moved.                                                                                                            | All                |
| MipInformation           | Microsoft Information Protection (MIP) related information.                                                                                                       | All                |
| SHA256HashData           | The SHA256 hash of the written dump file.                                                                                                                         | All                |
| ContextBaseFileName      | The base name of the file in the context of the event.                                                                                                            | All                |
| MountNamespaceUniqueId   | Unique ID for the mount namespace.                                                                                                                                | Linux Only         |
| RawProcessId             | The operating system's internal PID. For matching, use the UPID fields which guarantee a unique process identifier.                                             | Linux Only         |
| RUID                     | The real UID for a unix style process.                                                                                                                            | Linux Only         |
| RGID                     | The real GID for a unix style process.                                                                                                                            | Linux Only         |
| FsMagic                  | The magic number of the file system.                                                                                                                              | Linux Only         |
| DeviceMountCounter       | Counter for device mounts.                                                                                                                                        | Linux Only         |
| DeviceId                 | Identifier for the device.                                                                                                                                        | Linux Only         |
| FileSerialNumber         | Serial number of the file.                                                                                                                                        | Linux Only         |
| ChangeTime               | Unix change time, indicating last status change or file modification. NOTE: This is the only timestamp which is not modifiable, unlike the other file timestamps. | Linux Only         |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 54321,
  "ContextThreadId": 98765,
  "TreeId": "dump-tree-id",
  "FileObject": "0xABCDEF01",
  "MajorFunction": 1,
  "MinorFunction": 0,
  "IrpFlags": 0,
  "OperationFlags": 0,
  "Size": 2048000,
  "DiskParentDeviceInstanceId": "disk-xyz",
  "FileIdentifier": "dump-file-id",
  "TargetFileName": "C:\\Windows\\Temp\\lsass.dmp",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "AuthenticationId": 999,
  "UserName": "Administrator",
  "FileEcpBitmask": 0,
  "FileWrittenFlags": 0,
  "FileOperatorSid": "S-1-5-21-...",
  "FileCategory": 9,
  "VolumeSessionUUID": "vol-session-123",
  "FileSourcePath": "",
  "MipInformation": {},
  "SHA256HashData": "abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890",
  "ContextBaseFileName": "lsass.dmp",
  "MountNamespaceUniqueId": 456,
  "RawProcessId": 54321,
  "RUID": 0,
  "RGID": 0,
  "FsMagic": 61267,
  "DeviceMountCounter": 1,
  "DeviceId": 102,
  "FileSerialNumber": 12345,
  "ChangeTime": 1678886400
}
```
---
### 5. Example Queries
```xql
// Find all dump files written
event_simpleName=DmpFileWritten

// Find dump files written with a specific target file name
event_simpleName=DmpFileWritten TargetFileName="*lsass.dmp*"

// Find dump files written by the Administrator user
event_simpleName=DmpFileWritten UserName="Administrator"

// Find dump files larger than 1MB
event_simpleName=DmpFileWritten Size > 1048576

// Find dump files written to a network drive (Windows only)
event_simpleName=DmpFileWritten IsOnNetwork=true
```
---
