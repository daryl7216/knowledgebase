## Crowdstrike Table: NetworkListenIP6

### 1. Table Name

**NetworkListenIP6**

---

### 2. Table Usage

This table is created whenever an application using a connection-oriented protocol establishes a socket in listening mode (e.g., via a call to the listen() function). It provides details about the local IPv6 address, port, and protocol of the listening socket, which is useful for identifying services exposed on a host and detecting unexpected or malicious listener processes across ChromeOS, macOS, Windows, Android, Linux, and Falcon Container platforms.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| LocalAddressIP6 | The local IPv6 address the socket is listening on. | All |
| LocalPort | The local port number the socket is listening on. | All |
| Protocol | The network protocol used for the listening socket. Values: ICMP (1), TCP (6), UDP (17), IPV6 (41), ICMPV6 (58), UNKNOWN (255). | All |

---

### 4. Sample Log Example

```json
{
  "LocalAddressIP6": "::",
  "LocalPort": 8080,
  "Protocol": 6
}
```

---

### 5. Example Queries

```xql
// Find all IPv6 network listen events
event_simpleName=NetworkListenIP6

// Find applications listening on a specific local IPv6 port
event_simpleName=NetworkListenIP6 LocalPort=9000

// Find TCP listening sockets on any local IPv6 address
event_simpleName=NetworkListenIP6 Protocol=6 LocalAddressIP6="::"

// Find all UDP listening sockets over IPv6
event_simpleName=NetworkListenIP6 Protocol=17

// Find applications listening on a specific local IPv6 address and port
event_simpleName=NetworkListenIP6 LocalAddressIP6="fe80::abcd:ef12:3456:7890" LocalPort=8443
```

---