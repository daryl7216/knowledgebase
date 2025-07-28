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