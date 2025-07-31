# communityId()

The `communityId()` function computes the Community ID, a standard for hashing network flows, which can be used to easily correlate and join related network traffic across different systems.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_community_id` | The name of the field where the output Community ID will be stored. |
| **destinationip**| string | required | The name of the field containing the destination IP address (IPv4 or IPv6). |
| **destinationport**| integer | optional | The name of the field containing the destination port (0-65535). |
| **icmpcode** | integer | optional | The name of the field for the ICMP code (0-65535). Requires `icmptype` to be set. |
| **icmptype** | integer | optional | The name of the field for the ICMP type (0-65535). Requires `icmpcode` to be set. |
| **proto** | string | required | The name of the protocol field. The value must be an IANA protocol number (0-255) or a recognized protocol keyword. |
| **seed** | integer | optional <br> default: 0 | The seed value (0-65535) used when computing the hash. |
| **sourceip** | string | required | The name of the field containing the source IP address (IPv4 or IPv6). |
| **sourceport**| string | optional | The name of the field containing the source port (0-65535). |

***

## Function Operation

This function generates a consistent hash for a network flow based on its source/destination IPs and ports, protocol, and a seed value. It follows the Community ID specification. If the protocol is ICMP, it maps the `icmptype` and `icmpcode` fields to port equivalents as described in the specification. If the function receives invalid inputs, such as an invalid IP address or a port number out of range, it will assign an empty string to the output field.

***

## Example

### Compute Community ID for Netflow Logs

This example calculates the Community ID for standard netflow events by providing the necessary 5-tuple information (source/destination IP, source/destination port, and protocol).

* **Query Example**
    ```
    communityId(
        proto=flow.protocolIdentifier,
        sourceip=flow.sourceIPv4Address,
        sourceport=flow.sourceTransportPort,
        destinationip=flow.destinationIPv4Address,
        destinationport=flow.destinationTransportPort
    )
    ```

* **Step-by-Step**
    1.  The query starts with netflow events that contain fields for protocol, source IP/port, and destination IP/port.
    2.  The `communityId()` function takes the values from these fields to compute the flow hash.
    3.  The resulting Community ID is stored in a new field named `_community_id` by default.

* **Summary and Results**
    The query generates a consistent hash for each unique network flow, which allows for easy tracking and correlation of all events related to a single flow across different systems. This is highly useful for network analysis and security monitoring.