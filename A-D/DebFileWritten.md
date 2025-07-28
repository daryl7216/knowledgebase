## DebFileWritten Table: Deb File Written Event

### 1. Table Name
**DebFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a DEB (Debian Software Package) file. It provides details about the file, the process involved, and its location, which is useful for investigations into software installation, data transfer, and potential malware drops.

---

### 3. Important Parameter Description

| Parameter                | Description                                                                                                                                                                                                   | Platforms Affected |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| ContextTimeStamp         | System time of event creation.                                                                                                                                                                                | All                |
| ContextProcessId         | UPID of process originating this event.                                                                                                                                                                       | All                |
| ContextThreadId          | UTID of thread originating this event                                                                                                                                                                         | All                |
| TreeId                   | If this event is part of a detection tree, the tree ID it is part of.                                                                                                                                         | All                |
| FileObject               | The file object representing the written file.                                                                                                                                                                | Windows Only       |
| MajorFunction            | The major function code of the IRP (I/O Request Packet).                                                                                                                                                    | Windows Only       |
| MinorFunction            | The minor function code of the IRP.                                                                                                                                                                         | Windows Only       |
| IrpFlags                 | Flags associated with the IRP.                                                                                                                                                                                | Windows Only       |
| OperationFlags           | Flags indicating specific operation characteristics.                                                                                                                                                          | Windows Only       |
| Size                     | The size of the written file in bytes.                                                                                                                                                                        | All                |
| DiskParentDeviceInstanceId | The device instance ID of the parent disk device.                                                                                                                                                             | All                |
| FileIdentifier           | A unique identifier for the file.                                                                                                                                                                             | All                |
| TargetFileName           | The resulting file name that was written.                                                                                                                                                                     | All                |
| IsOnNetwork              | Set to true if the relevant file listed in the event is on a network drive. False otherwise.                                                                                                                  | Windows Only       |
| IsOnRemovableDisk        | If true, it means this file was located on a removable disk.                                                                                                                                                  | All                |
| AuthenticationId         | Values: INVALID_LUID (0) NETWORK_SERVICE (996) LOCAL_SERVICE (997) SYSTEM (999)                                                                                                                               | Windows Only       |
| TokenType                | Values: INVALID_TOKEN (0) PRIMARY_TOKEN (1) IMPERSONATION_TOKEN (2)                                                                                                                                           | Windows Only       |
| UserName                 | The name of the user who performed the write operation.                                                                                                                                                       | All                |
| FileEcpBitmask           | Values: ECP_SRV_OPEN (0x00000001) ECP_CS_SENSOR_OPEN (0x00000002) ECP_CS_SENSOR_FILE_CHARACTERISTICS (0x00000004)                                                                                              | Windows Only       |
| TemporaryFileName        | The temporary name of the file during the write operation.                                                                                                                                                    | Windows Only       |
| FileWrittenFlags         | Values: HASH_IS_VALID (0x00000000) HASH_FAILED (0x00000001) HASH_ABORTED_TOO_LARGE (0x00000002) PAGING_WRITE (0x00000004)                                                                                   | Windows Only       |
| FileOperatorSid          | Security Identifier (SID) of the file operator.                                                                                                                                                               | Windows Only       |
| FileCategory             | Values: OTHER (0) ARCHIVES (1) OFFICE_DOCUMENTS (2) MULTIMEDIA_FILES (3) DESIGN_FILES (4) SOURCE_CODE (5) EXECUTABLE_FILES (6) VIRTUAL_MACHINE_FILES (7) EMAIL_FILES (8) DATA_AND_LOGS (9) ENCRYPTED (10) | All                |
| VolumeSessionUUID        | The UUID of the volume session where the file was written.                                                                                                                                                    | All                |
| FileSourcePath           | The source path of the file if it was copied or moved.                                                                                                                                                        | All                |
| MipInformation           | Microsoft Information Protection (MIP) related information.                                                                                                                                                   | All                |
| SHA256HashData           | The SHA256 hash of the written file.                                                                                                                                                                        | All                |
| ContextBaseFileName      | The base name of the file in the context of the event.                                                                                                                                                        | All                |
| MountNamespaceUniqueId   | Unique ID for the mount namespace.                                                                                                                                                                            | Linux Only         |
| RawProcessId             | The operating system's internal PID. For matching, use the UPID fields which guarantee a unique process identifier.                                                                                         | Linux Only         |
| RUID                     | The real UID for a unix style process.                                                                                                                                                                        | Linux Only         |
| RGID                     | The real GID for a unix style process.                                                                                                                                                                        | Linux Only         |
| FsMagic                  | The magic number of the file system.                                                                                                                                                                          | Linux Only         |
| DeviceMountCounter       | Counter for device mounts.                                                                                                                                                                                    | Linux Only         |
| DeviceId                 | Identifier for the device.                                                                                                                                                                                    | Linux Only         |
| FileSerialNumber         | Serial number of the file.                                                                                                                                                                                    | Linux Only         |
| ChangeTime               | Unix change time, indicating last status change or file modification. NOTE: This is the only timestamp which is not modifiable, unlike the other file timestamps.                                          | Linux Only         |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "FileObject": "0x12345678",
  "MajorFunction": 1,
  "MinorFunction": 0,
  "IrpFlags": 0,
  "OperationFlags": 0,
  "Size": 102400,
  "DiskParentDeviceInstanceId": "disk-123",
  "FileIdentifier": "file-abc",
  "TargetFileName": "/tmp/package.deb",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "AuthenticationId": 999,
  "TokenType": 1,
  "UserName": "user123",
  "FileEcpBitmask": 0,
  "TemporaryFileName": "",
  "FileWrittenFlags": 0,
  "FileOperatorSid": "S-1-5-21-...",
  "FileCategory": 1,
  "VolumeSessionUUID": "vol-uuid-1",
  "FileSourcePath": "",
  "MipInformation": {},
  "SHA256HashData": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2",
  "ContextBaseFileName": "package.deb",
  "MountNamespaceUniqueId": 123,
  "RawProcessId": 54321,
  "RUID": 1000,
  "RGID": 1000,
  "FsMagic": 61267,
  "DeviceMountCounter": 1,
  "DeviceId": 102,
  "FileSerialNumber": 456789,
  "ChangeTime": 1678886400
}
```
---
### 5. Example Queries
```xql
// Find all DEB files written
event_simpleName=DebFileWritten

// Find DEB files written to a specific target file name
event_simpleName=DebFileWritten TargetFileName="/var/cache/apt/archives/my_package.deb"

// Find DEB files written to removable media (Windows, macOS, Linux)
event_simpleName=DebFileWritten IsOnRemovableDisk=true

// Find DEB files written by a specific user (Windows, macOS, Linux)
event_simpleName=DebFileWritten UserName="root"

// Find DEB files with a specific SHA256 hash
event_simpleName=DebFileWritten SHA256HashData="a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2"

// Find DEB files categorized as 'ARCHIVES' (Windows, macOS, Linux)
event_simpleName=DebFileWritten FileCategory=1
```
---
