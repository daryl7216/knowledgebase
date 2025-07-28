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
