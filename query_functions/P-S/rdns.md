# rdns()

The `rdns()` function resolves hostnames using reverse DNS lookups.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `hostname` | Specifies the field into which the resolved value is stored. |
| `field` | string | required | Specifies the field to run the RDNS lookup against. |
| `server` | string | optional | Specifies a DNS server address. |

***

## Function Operation

The `rdns()` function performs reverse DNS lookups to resolve IP addresses into hostnames.

Key aspects of its operation include:

  * If a lookup fails, the event will be kept, but the designated output field will not be added.
  * The number of resulting events from this function is constrained by the `MAX_STATE_LIMIT` dynamic configuration parameter, which has a default limit of 20,000. If this limit is surpassed, the results will be truncated, and a warning will be issued.
  * To prevent the `rdns()` function from indefinitely blocking query execution, a timeout is applied to all RDNS requests. If a request does not yield a result within this timeout, the lookup is considered to have failed for the associated event.
  * However, if a request eventually returns after the timeout, its result is cached internally by LogScale for a certain period. Consequently, a static query using `rdns()` might fail a lookup on its initial execution but succeed in a subsequent execution.
  * In live queries, this behavior is less problematic as `rdns()` is continuously evaluated, making it preferable for use in such contexts.
  * Reverse DNS lookups are generally not considered authoritative; the owner of an IP address can change it to point to any arbitrary hostname. For an authoritative alternative without these limitations, consider using the `asn()` function instead.
  * If no RDNS server is explicitly specified, a system default is used, which can be overridden by the `server` parameter.

The function disallows reverse DNS lookups for specific IP address ranges (both IPv4 and IPv6), which are listed in the documentation. For self-hosted customers, the allowed IP addresses and servers for reverse DNS queries can be restricted using `IP_FILTER_RDNS` and `IP_FILTER_RDNS_SERVER` dynamic configuration parameters, respectively.

***

## Examples

### Resolve `ipAddress` using the server `8.8.8.8`, and store the resulting DNS name in `dnsName`

This example demonstrates resolving an IP address using a specific DNS server and storing the result in a custom field name.

  * **Query Example**

    ```
    rdns(ipAddress, server="8.8.8.8", as=dnsName)
    ```

  * **Step-by-Step**

    1.  The query targets an `ipAddress` field within events.
    2.  `rdns(ipAddress, server="8.8.8.8", as=dnsName)`: Performs a reverse DNS lookup on the `ipAddress` field. It uses the specified DNS server `8.8.8.8` to perform the lookup. The resolved hostname is then stored in a new field named `dnsName`.

  * **Summary and Results**
    This query is used to resolve an IP address to a hostname, specifying a particular DNS server and a custom output field name.

---

### Resolve `ipAddress` and store the resulting DNS name in `hostname`

This example demonstrates a basic reverse DNS lookup, storing the resolved hostname in the default output field.

  * **Query Example**

    ```
    rdns(ipAddress)
    ```

  * **Step-by-Step**

    1.  The query targets an `ipAddress` field within events.
    2.  `rdns(ipAddress)`: Performs a reverse DNS lookup on the `ipAddress` field. Since no `server` is specified, a system default DNS server is used. The resolved hostname is stored in the default output field, `hostname`.

  * **Summary and Results**
    This query performs a basic reverse DNS lookup on an IP address field, using default settings for the DNS server and output field name.