# base64Encode()

The `base64Encode()` function performs Base64 encoding of a field's value. This process converts the input string to UTF-8 to ensure that values with special characters are transformed into a known set of non-special characters.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_base64Encode` | The name of the field where the encoded value will be stored. |
| **field** | string | required | The field containing the value to be encoded. The parameter name `field` can be omitted. |

***

## Function Operation

This function always encodes input strings as UTF-8 and does not accept a `charset` parameter, which is a key difference from its counterpart, `base64Decode()`. It's designed to make data handling easier for systems that only accept ASCII characters. It is important to note that while Base64 encoding is useful for data transport, it is not a secure method of encryption.

***

## Example

### Perform Base64 Encoding of a Field

This example takes a simple string, "Hello, World!", and converts it to its Base64 representation.

* **Query Example**
    ```
    base64Encode(a)
    ```

* **Input Event Data**
    An event with a field `a` containing the string `Hello, World!`.

* **Step-by-Step**
    1.  The query starts with an event containing the field `a`.
    2.  `base64Encode()` takes the input string `Hello, World!` from field `a`.
    3.  It converts the string to UTF-8 and then encodes it with Base64.
    4.  The result is stored in the default output field, `_base64Encode`.

* **Result Event Data**
    The output is a new field containing the Base64-encoded string.
    ```
    _base64Encode="SGVsbG8sIFdvcmxkIQ=="
    ```