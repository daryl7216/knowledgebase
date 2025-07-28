## DnsRequest Table: DNS Request Event

### 1. Table Name
**DnsRequest**

---

### 2. Table Usage
This event is generated when a process on a host makes a DNS query. The sensor captures the request and the corresponding response, including the queried domain and any returned IP addresses. Monitoring DNS requests is crucial for detecting a wide range of malicious activities, such as command-and-control (C2) communication, malware downloads, and connections to phishing sites.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| DomainName | The domain name that was queried. | All |
| RequestType | The type of DNS record requested (e.g., A, AAAA, CNAME, TXT). A value of `1` corresponds to an A record (IPv4). | All |
| FirstIP4Record | The first IPv4 address returned in the DNS response. | Windows |
| IP4Records | A list of all IPv4 addresses returned in the DNS response. | Windows, Linux, macOS |
| QueryStatus | The status code of the DNS query. A status of `0` typically indicates success, while others can indicate errors like `9003` (NXDOMAIN/Non-Existent Domain). | Windows, Linux, macOS |

---

### 4. Sample Log Example

```json
{
  "DomainName": "[www.google.com](https://www.google.com)",
  "RequestType": "1",
  "FirstIP4Record": "142.250.191.196",
  "IP4Records": [
    "142.250.191.196"
  ],
  "QueryStatus": "0"
}
```

---
### 5. Example Queries

```xql
// Find all DNS requests for a specific domain
event_simpleName=DnsRequest DomainName="*evil-c2-domain.com"

// Hunt for DNS requests for TXT records, which can be used for data exfiltration or C2
event_simpleName=DnsRequest RequestType=16

// Find DNS requests that resulted in a non-existent domain error (NXDOMAIN)
event_simpleName=DnsRequest QueryStatus=9003

// Search for domains that resolve to a specific suspicious IP address
event_simpleName=DnsRequest IP4Records="*198.51.100.55*"

// Look for A record requests (RequestType=1) and show the first IP address returned
event_simpleName=DnsRequest RequestType=1 | table DomainName, FirstIP4Record
```
---