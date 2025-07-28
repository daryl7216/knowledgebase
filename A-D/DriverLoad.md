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
