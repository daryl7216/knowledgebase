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
