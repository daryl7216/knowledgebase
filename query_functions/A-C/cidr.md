# cidr()

The `cidr()` function filters events by checking if an IPv4 or IPv6 address in a specified field belongs to one or more subnet ranges. These subnet ranges can be provided directly in the query or loaded from a lookup file.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **column** | string | optional | The column name in a lookup file that contains the subnet list. Used with the `file` parameter. |
| **field** | string | required | The field containing the IP address to check. The parameter name `field` can be omitted. |
| **file** | string | optional | The name of the lookup file (CSV or JSON) containing the subnet list. Used with the `column` parameter. |
| **negate** | boolean | optional (deprecated) <br> default: `false` | Deprecated parameter. Use the `!cidr()` syntax for negation instead. |
| **subnet** | array of strings | optional | A list of subnet ranges to match against. |

***

## Examples

### Check if a Field Contains a Valid IP Address

This example uses `cidr()` in a creative way to validate if a field contains any valid IPv4 or IPv6 address. It does this by checking against the subnets that encompass all possible addresses (`0.0.0.0/0` for IPv4 and `::/0` for IPv6).

* **Query Example**
    ```
    case {
      cidr("address", subnet=["0.0.0.0/0", "::/0"]) | ip := address;
      *
    }
    ```

* **Step-by-Step**
    1.  The query uses a `case` statement to structure the logic.
    2.  `cidr()` checks if the value in the `address` field falls within the entire possible range of IPv4 or IPv6 addresses.
    3.  If it's a valid IP, a new field `ip` is created and assigned the value from the `address` field.

* **Summary and Results**
    This query effectively acts as an IP address validator. Events with a valid IP in the `address` field will have a new `ip` field created, while events with non-IP data in that field will be ignored by the `cidr()` condition.

---

### Filter Events Using Subnets from a File

This example demonstrates how to filter events by matching an IP address against a list of subnets loaded from an external CSV file. This is a powerful way to manage large or dynamic network groups.

* **Query Example**
    ```
    cidr(field=SRC, file="cidrfile.csv", column="cidr-block")
    ```

* **Step-by-Step**
    1.  The query starts with events that have a `SRC` field containing an IP address.
    2.  The `cidr()` function is instructed to use the lookup file `cidrfile.csv`.
    3.  It reads the list of subnets from the column named `cidr-block` within that file.
    4.  The function then keeps only the events where the IP address in the `SRC` field matches one of the subnets from the file.

* **Summary and Results**
    This query allows you to maintain network groups, allowlists, or blocklists in an external file instead of hard-coding them into your queries. The search will only be performed on events where the `SRC` IP falls within one of the ranges specified in the `cidrfile.csv`.