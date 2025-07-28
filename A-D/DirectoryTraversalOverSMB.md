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
