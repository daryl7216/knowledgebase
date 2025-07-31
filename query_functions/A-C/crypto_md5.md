# crypto:md5()

The `crypto:md5()` function computes a cryptographic MD5 hash of one or more fields. The output is a hexadecimal string. This is useful for calculating checksums or combining multiple fields into a single, fixed-length identifier.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_md5` | The name of the output field that will store the hash. |
| **field** | array of strings | required | The name of the field or fields to hash. The parameter name `field` can be omitted. |

***

## Function Operation

The function operates on the UTF-8 encoding of the provided field values. When given multiple fields, it concatenates their values before computing the hash. If a field is missing from an event or has an empty value, it is treated as an empty string for the concatenation.

***

## Example

### MD5 Hash Multiple Fields

This example shows how to hash multiple fields together to create a single, reproducible MD5 sum. This can be an effective method for creating a unique ID for a specific set of field values, which can then be used for lookups or joins.

* **Query Example**
    ```
    crypto:md5([a,b,c])
    ```

* **Step-by-Step**
    1.  The query starts with events that contain the fields `a`, `b`, and `c`.
    2.  The `crypto:md5()` function takes the values from these three fields.
    3.  It concatenates the values together into a single string.
    4.  It then computes the MD5 hash of the concatenated string and returns the result in a new field named `_md5`.

* **Summary and Results**
    The query creates a single MD5 hash from the combined values of the supplied fields. This is a reproducible process, meaning the same set of input values will always produce the same MD5 hash. This makes it a useful technique for creating unique identifiers for specific event types or for creating a common key to join two different datasets.