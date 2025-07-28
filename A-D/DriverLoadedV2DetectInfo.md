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