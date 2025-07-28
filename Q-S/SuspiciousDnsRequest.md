## SuspiciousDnsRequest Table: Suspicious DNS Request Event

### 1. Table Name
**SuspiciousDnsRequest**

---

### 2. Table Usage
This event is generated when the sensor detects a suspicious DNS request. A request is considered suspicious if it is attempting to resolve a domain that is on the sensor's list of known malicious or suspicious domains. This is often an indicator of command-and-control (C2) communication, malware download, or other malicious activity.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| DomainName | The suspicious domain name that was queried. | All |
| RequestType | The type of DNS record requested (e.g., A, AAAA, CNAME, TXT). | All |

---

### 4. Sample Log Example

```json
{
  "DomainName": "malicious-c2-domain.com",
  "RequestType": "1"
}
```
---

### 5. Example Queries
```xql
// Find all suspicious DNS requests
event_simpleName=SuspiciousDnsRequest

// Hunt for requests to a specific suspicious domain
event_simpleName=SuspiciousDnsRequest DomainName="*evil-domain.net"

// Find suspicious DNS requests for TXT records, which can be used for C2
event_simpleName=SuspiciousDnsRequest RequestType=16

// Find suspicious requests for A records (IPv4)
event_simpleName=SuspiciousDnsRequest RequestType=1
```
---
