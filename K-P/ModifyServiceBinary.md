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

