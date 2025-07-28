## Crowdstrike Table: NetworkReceiveAcceptIP4

### 1. Table Name

**NetworkReceiveAcceptIP4**

---

### 2. Table Usage

This table is generated when an application using a connection-oriented protocol attempts to accept a remote connection request (e.g., via a call to the accept() function) or a connectionless protocol attempts to process the first datagram received. This event is crucial for monitoring incoming network connections, identifying suspicious communication, and analyzing potential exploitation attempts across macOS, Android, Windows, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| RemoteAddressIP4 | The remote IPv4 address of the connection being received or accepted. | All |
| LocalPort | The local port number on which the connection is being received or accepted. | All |
| Protocol | The network protocol used for the connection. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |
| RemotePort | The remote port number of the connection being received or accepted. | All |
| LocalAddressIP4 | The local IPv4 address on which the connection is being received or accepted. | All |

---

### 4. Sample Log Example

```json
{
  "RemoteAddressIP4": "192.168.1.10",
  "LocalPort": 80,
  "Protocol": 6,
  "RemotePort": 54321,
  "LocalAddressIP4": "10.0.0.1"
}
```

---

### 5. Example Queries

```xql
// Find all IPv4 network receive/accept events
event_simpleName=NetworkReceiveAcceptIP4

// Find applications accepting connections on a specific local port
event_simpleName=NetworkReceiveAcceptIP4 LocalPort=443 Protocol=6

// Find incoming connections from a specific remote IP address
event_simpleName=NetworkReceiveAcceptIP4 RemoteAddressIP4="172.16.0.50"

// Find all accepted UDP connections
event_simpleName=NetworkReceiveAcceptIP4 Protocol=17

// Find connections accepted on a particular local IP address and port
event_simpleName=NetworkReceiveAcceptIP4 LocalAddressIP4="10.0.0.1" LocalPort=80
```

---