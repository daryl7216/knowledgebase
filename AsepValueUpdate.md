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