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
