## TokenImpersonated Table: Token Impersonation Event

### 1. Table Name
**TokenImpersonated**

---

### 2. Table Usage
This event is generated when a token impersonation action occurs on a Windows host. Token impersonation is a technique used by adversaries to escalate privileges or move laterally by using the security context of another user. Monitoring this event provides detailed "before and after" information, which is critical for detecting and investigating potential credential theft and privilege escalation attacks.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| ImpersonatedUserName | The name of the user account whose token is being used for impersonation. | Windows Only |
| OriginalUserName | The name of the user account that is performing the impersonation. | Windows Only |
| OriginalUserSid | The Security Identifier (SID) of the original user account that initiated the impersonation. | Windows Only |

---

### 4. Sample Log Example

```json
{
  "ImpersonatedUserName": "NT AUTHORITY\\SYSTEM",
  "OriginalUserName": "CORP\\jsmith",
  "OriginalUserSid": "S-1-5-21-1234567890-123456789-1234567890-1001"
}
```

### 5. Example Queries
```xql
// Find all token impersonation events
event_simpleName=TokenImpersonated

// Find instances where a specific user is performing impersonation
event_simpleName=TokenImpersonated OriginalUserName="CORP\\suspicious_user"

// Find instances where a high-privilege account is being impersonated
event_simpleName=TokenImpersonated ImpersonatedUserName="*SYSTEM"

// Investigate token impersonation by a specific original user SID
event_simpleName=TokenImpersonated OriginalUserSid="S-1-5-21-1234567890-123456789-1234567890-500"
```
---
