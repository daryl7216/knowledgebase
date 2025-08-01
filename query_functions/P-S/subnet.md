# subnet()

The `subnet()` function computes a subnet from an IPv4 field; by default, it emits it into a subnet field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `subnet` | Specifies the name of the output field. |
| `bits` | number | required | Specifies the prefix bits to include in the subnet, for example, 23. |
| `field` | string | required | Specifies the input field. |

***

## Function Operation

The `subnet()` function takes an IPv4 address from a specified input `field` and calculates its corresponding subnet. The size of the subnet is determined by the `bits` parameter, which defines the number of prefix bits to include (e.g., 24 for a /24 subnet). The computed subnet address is then outputted into a new field. By default, this output field is named `subnet`, but a custom name can be assigned using the `as` parameter. This function is useful for network analysis, security logging, and grouping events by network segments.

***

## Examples

### Compute subnet for `ipAddress` using 23-bit prefix; emit into `subnet` field

This example demonstrates how to compute a subnet from an IP address field using a 23-bit prefix and store the result in a field named `subnet`.

  * **Query Example**

    ```
    subnet(ipAddress, bits=23, as=subnet)
    ```

  * **Step-by-Step**

    1.  The query targets an `ipAddress` field within events.
    2.  `subnet(ipAddress, bits=23, as=subnet)`: Calculates the subnet of the `ipAddress` field using a 23-bit prefix. The resulting subnet address is stored in a new field named `subnet`.

  * **Summary and Results**
    This query computes a subnet for a given IPv4 address field, specifying the prefix length and the output field name.