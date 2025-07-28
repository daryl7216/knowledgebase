## Crowdstrike Table: NetworkConnectIP4

### 1. Table Name

**NetworkConnectIP4**

---

### 2. Table Usage

This event is generated when an application attempts a remote connection. It provides details about the source and destination IP addresses and ports, and the protocol used, which is useful for monitoring network activity, identifying suspicious connections, and analyzing communication patterns across various platforms including ChromeOS, macOS, Android, Windows, iOS, and Linux.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP4 | The remote IPv4 address of the connection. | All |
| RemotePort | The remote port number of the connection. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |
| LocalAddressIP4 | The local IPv4 address of the connection. | All |
| LocalPort | The local port number of the connection. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP4": "192.168.1.100",
  "RemotePort": 443,
  "Protocol": 6,
  "LocalAddressIP4": "10.0.0.5",
  "LocalPort": 54321
}
```

---

### 5. Example Queries

```xql
// Find all network connections to a specific remote IP address
event_simpleName=NetworkConnectIP4 RemoteAddressIP4="1.2.3.4"

// Find all outbound TCP connections
event_simpleName=NetworkConnectIP4 Protocol=6 ConnectionDirection=0

// Find all network connections on a specific remote port
event_simpleName=NetworkConnectIP4 RemotePort=8080

// Find network connections originating from a specific local IP and port
event_simpleName=NetworkConnectIP4 LocalAddressIP4="10.0.0.10" LocalPort=12345

// Find all UDP connections
event_simpleName=NetworkConnectIP4 Protocol=17
```

---