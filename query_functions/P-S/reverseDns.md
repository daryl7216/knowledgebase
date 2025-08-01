# reverseDns()

The `reverseDns()` function performs reverse DNS lookups to find hostnames associated with IP addresses. It adds hostname information to events by querying DNS servers.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `hostname` | Specifies the field into which the resolved value is stored. |
| `field` | string | required | Specifies the field to run the RDNS lookup against. |
| `limit` | number | optional \<br\> default: `ReverseDnsDefaultLimit` | Maximum number of unique IPs to process. If the input events contain more unique IP addresses than the limit allows, a warning is issued and the rest of the events are not annotated with hostname information. The default is the value of the dynamic configuration `ReverseDnsDefaultLimit` (or 5000 if not set). |
| `server` | string | optional | Name or address of the DNS server to use. The default is the value of the configuration value `RDNS_DEFAULT_SERVER` or as otherwise configured in the system. |

***

## Function Operation

The `reverseDns()` function is used to perform reverse DNS lookups, resolving IP addresses to their corresponding hostnames.

Key operational characteristics and limitations include:

  * It functions only in the result phase of a query (after the first aggregator) and only at the top-level.
  * It cannot be used in subqueries.
  * It processes a limited number of results.
  * It cannot run in filter alerts and aggregate alerts; for alerts, scheduled searches with appropriate actions should be used instead.
  * The function attempts to resolve as many IP addresses as possible within specified limits, such as the maximum number of addresses or a timeout period.
  * If resolution fails for any addresses (e.g., no response before timeout, too many addresses), it emits a warning and returns events without hostname information.
  * The `reverseDns()` function uses an external network service for resolution, which means it can be affected by DNS server or network unreliability, or by DNS server throttling/blocking of the cluster's IP range. Therefore, LogScale does not recommend using the annotated hostname for filtering.
  * There is a cluster-wide rate limit (default 1000 requests per second, distributed among nodes) on the number of requests, which can be controlled by `ReverseDnsRequestsPerSecond`.
  * The number of concurrent requests a node can make is also limited (default 10), controlled by `ReverseDnsConcurrentRequests`.
  * If a lookup fails, the event is kept without modification. To exclude such events, a field test like `hostname=*` can be used after the function.
  * A timeout is applied to prevent indefinite blocking of query execution. If a result isn't received from the DNS server before the timeout (default 5 seconds), the hostname is not annotated. This timeout is controlled by `ReverseDnsDefaultTimeoutInMs`.
  * The number of unique IP addresses resolved per query is limited by the `limit` parameter, the timeout, and the node-wide rate limit.
  * Reverse DNS is generally informational, not authoritative, as IP owners can change hostname mappings. For an authoritative alternative, `asn()` is suggested.
  * If no DNS server is specified, the `RDNS_DEFAULT_SERVER` environment variable is used; otherwise, the system default DNS server is used.
  * The function caches resolved hostnames for a limited time to reduce external calls and provide faster results.
  * The function has a default list of disallowed IP address ranges (IPv4 and IPv6) that it will not resolve.
  * For self-hosted customers, allowed IP addresses and servers for reverse DNS queries can be restricted by `IP_FILTER_RDNS` and `IP_FILTER_RDNS_SERVER`.

***

## Examples

### Resolve the hostname for the IP address in the field `ip`, using the default DNS server. The aggregator function `tail()` is used to restrict the number of events.

  * **Query Example**

    ```
    tail(100)
    | reverseDns(ip)
    ```

  * **Step-by-Step**

    1.  `tail(100)`: Restricts the number of events to the last 100, ensuring `reverseDns()` operates on a limited dataset.
    2.  `| reverseDns(ip)`: Performs a reverse DNS lookup on the `ip` field for the limited set of events, using the default DNS server.

---

### Using `groupBy()` instead of `tail()` to restrict events for `reverseDns()`

  * **Query Example**

    ```
    groupBy(ip, function=[], limit=100)
    reverseDns(ip)
    ```

  * **Step-by-Step**

    1.  `groupBy(ip, function=[], limit=100)`: Groups events by `ip` and limits the number of groups to 100, effectively restricting the unique IPs processed by `reverseDns()`.
    2.  `reverseDns(ip)`: Performs the reverse DNS lookup on the unique `ip` values.

---

### Specifying a custom DNS server when using `groupBy()`

  * **Query Example**

    ```
    groupBy(ip, function=[], limit=100)
    reverseDns(ip, server=8.8.8.8)
    ```

  * **Step-by-Step**

    1.  `groupBy(ip, function=[], limit=100)`: Groups events by `ip` and limits the number of groups.
    2.  `reverseDns(ip, server=8.8.8.8)`: Performs the reverse DNS lookup on `ip` using `8.8.8.8` as the specified DNS server.

---

### Limiting the number of unique IPs processed by `reverseDns()`

  * **Query Example**

    ```
    tail(100)
    reverseDns(ip, limit=10)
    ```

  * **Step-by-Step**

    1.  `tail(100)`: Restricts the input events to the last 100.
    2.  `reverseDns(ip, limit=10)`: Performs reverse DNS lookups on the `ip` field, but only processes a maximum of 10 unique IP addresses from the input.