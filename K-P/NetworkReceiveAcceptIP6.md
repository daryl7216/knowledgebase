## Crowdstrike Table: NetworkReceiveAcceptIP6

### 1. Table Name

**NetworkReceiveAcceptIP6**

---

### 2. Table Usage

This table is created whenever an application using a connection-oriented protocol attempts to accept a remote connection request (e.g., via a call to the accept() function) or a connectionless protocol attempts to process the first datagram received. This event is crucial for monitoring incoming network connections, identifying suspicious communication, and analyzing potential exploitation attempts across macOS, Android, Windows, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP6 | The remote IPv6 address of the connection being received or accepted. | All |
| LocalPort | The local port number on which the connection is being received or accepted. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58). | All |
| RemotePort | The remote port number of the connection being received or accepted. | All |
| LocalAddressIP6 | The local IPv6 address on which the connection is being received or accepted. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP6": "fe80::1ff:fe23:4567:890a",
  "LocalPort": 80,
  "Protocol": 6,
  "RemotePort": 54321,
  "LocalAddressIP6": "fe80::abcd:ef12:3456:7890"
}
```

---

### 5. Example Queries

```xql
// Find all IPv6 network receive/accept events
event_simpleName=NetworkReceiveAcceptIP6

// Find applications accepting connections on a specific local IPv6 port
event_simpleName=NetworkReceiveAcceptIP6 LocalPort=443 Protocol=6

// Find incoming connections from a specific remote IPv6 address
event_simpleName=NetworkReceiveAcceptIP6 RemoteAddressIP6="fe80::cdef:1234:5678:90ab"

// Find all accepted UDP connections over IPv6
event_simpleName=NetworkReceiveAcceptIP6 Protocol=17

// Find connections accepted on a particular local IPv6 address and port
event_simpleName=NetworkReceiveAcceptIP6 LocalAddressIP6="fe80::abcd:ef12:3456:7890" LocalPort=80
```

---