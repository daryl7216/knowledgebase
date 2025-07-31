# ipLocation()

The `ipLocation()` function adds geolocation data to events based on an IPv4 or IPv6 address. It uses the MaxMind GeoLite2 database to enrich events with four new fields: `.country`, `.city`, `.lon` (longitude), and `.lat` (latitude).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | A prefix to use for the new geolocation fields. Defaults to the name of the input field. |
| **field** | string | optional <br> default: `ip` | The field containing the IP address to look up. The parameter name `field` can be omitted. |

***

## Function Operation

This function reads an IP address from the specified `field` and adds geolocation data to the event. For example, if the input field is `ip`, the function will add `ip.country`, `ip.city`, `ip.lon`, and `ip.lat`. The underlying GeoLite2 database is updated automatically if the cluster has a valid license. Note that city information may not be available for all IP addresses.

***

## Example

### Retrieve Location Data From a Specified Field

This example demonstrates how to use `ipLocation()` to get geolocation data for IP addresses that are located in the `@rawstring` of an event.

* **Query Example**
    ```
    ipLocation(field=@rawstring)
    ```

* **Input Event Data**
    A list of events where the raw string is an IP address.

| @rawstring |
| :--- |
| 165.225.194.1|
| 1.2.3.4 |
| 4.3.2.1 |
| 8.8.8.8 |

* **Step-by-Step**
    1.  The query starts with the source events.
    2.  `ipLocation(field=@rawstring)` tells the function to read the IP address from the `@rawstring` field of each event.
    3.  The function enriches each event with four new fields, using `@rawstring` as the prefix: `@rawstring.country`, `@rawstring.city`, `@rawstring.lon`, and `@rawstring.lat`.

* **Result Event Data**
    The output shows the original events now enriched with the new geolocation fields.

| @rawstring | @rawstring.city | @rawstring.country| @rawstring.lat |
| :--- | :--- | :--- | :--- |
| 165.225.194.1| Copenhagen | DK | 55.674 |
| 1.2.3.4 | <no value> | AU | -33.494 |
| 4.3.2.1 | <no value> | US | 37.751 |
| 8.8.8.8 | <no value> | US | 37.751 |