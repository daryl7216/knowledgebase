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