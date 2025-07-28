## Crowdstrike Table: NetworkConnectIP6

### 1. Table Name

**NetworkConnectIP6**

---

### 2. Table Usage

This event is created whenever an application using a connection-oriented protocol attempts a remote connection (e.g., via a call to the connect() function) or a connectionless protocol first attempts to transmit a message (e.g., via a call to the sendto() function). This event is generated when an application attempts a remote connection. It provides details about the source and destination IPv6 addresses and ports, and the protocol used, which is useful for monitoring network activity, identifying suspicious connections, and analyzing communication patterns across various platforms including ChromeOS, macOS, Android, Windows, iOS, and Linux.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP6 | The remote IPv6 address of the connection. | All |
| RemotePort | The remote port number of the connection. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |
| LocalAddressIP6 | The local IPv6 address of the connection. | All |
| LocalPort | The local port number of the connection. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP6": "fe80::1ff:fe23:4567:890a",
  "RemotePort": 80,
  "Protocol": 6,
  "LocalAddressIP6": "fe80::abcd:ef12:3456:7890",
  "LocalPort": 54321
}
```

---

### 5. Example Queries

```xql
// Find all network connections to a specific remote IPv6 address
event_simpleName=NetworkConnectIP6 RemoteAddressIP6="fe80::1ff:fe23:4567:890a"

// Find all outbound TCP connections over IPv6
event_simpleName=NetworkConnectIP6 Protocol=6 ConnectionDirection=0

// Find all network connections on a specific remote port for IPv6
event_simpleName=NetworkConnectIP6 RemotePort=443

// Find network connections originating from a specific local IPv6 and port
event_simpleName=NetworkConnectIP6 LocalAddressIP6="fe80::abcd:ef12:3456:7890" LocalPort=12345

// Find all UDP connections over IPv6
event_simpleName=NetworkConnectIP6 Protocol=17
```

---