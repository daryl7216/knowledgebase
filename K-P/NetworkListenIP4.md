## Crowdstrike Table: NetworkListenIP4

### 1. Table Name

**NetworkListenIP4**

---

### 2. Table Usage

This table is generated when an application establishes a socket in listening mode. This event is crucial for identifying services exposed on a host, detecting unexpected or malicious listener processes, and understanding the network attack surface across ChromeOS, macOS, Windows, Android, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| LocalAddressIP4 | The local IPv4 address the socket is listening on. | All |
| LocalPort | The local port number the socket is listening on. | All |
| Protocol | The network protocol used for the listening socket. Values: IP (0), ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |

---

### 4. Sample Log Example

```json
{
  "LocalAddressIP4": "0.0.0.0",
  "LocalPort": 80,
  "Protocol": 6
}
```

---

### 5. Example Queries

```xql
// Find all IPv4 network listen events
event_simpleName=NetworkListenIP4

// Find applications listening on a specific local port
event_simpleName=NetworkListenIP4 LocalPort=4444

// Find TCP listening sockets on any local IPv4 address
event_simpleName=NetworkListenIP4 Protocol=6 LocalAddressIP4="0.0.0.0"

// Find all UDP listening sockets
event_simpleName=NetworkListenIP4 Protocol=17

// Find applications listening on a specific local IP address and port
event_simpleName=NetworkListenIP4 LocalAddressIP4="192.168.1.10" LocalPort=8080
```

---
