# asn()

The `asn()` function enriches an event by adding the Autonomous System (AS) number and organization associated with a given IP address.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: input field name | The prefix to use for the new fields created by the function. |
| **field** | string | optional <br> default: `ip` | The field containing the IP address to look up. The parameter name `field` can be omitted. |

***

## Function Operation

This function takes an IP address from a specified field and uses the MaxMind GeoLite2 database to find its associated AS number and organization. By default, it reads from the `ip` field and creates two new fields: `ip.asn` and `ip.org`. You can specify a different source field and a different prefix for the output fields. The function can process both IPv4 and IPv6 addresses.

***

## Example

### Determine AS Number with Custom Input and Output Fields

This example shows how to use `asn()` with a custom source field for the IP address (`ipaddr`) and a custom prefix for the new fields (`address`).

* **Query Example**
    ```
    asn(field=ipaddr, as=address)
    ```

* **Step-by-Step**
    1.  The query starts with events that have an `ipaddr` field containing an IP address.
    2.  The `asn()` function takes the value from the `ipaddr` field.
    3.  It then adds two new fields to the event: `address.asn` and `address.org`, containing the Autonomous System number and organization name for that IP.