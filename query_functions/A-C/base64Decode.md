# base64Decode()

The `base64Decode()` function performs Base64 decoding of a field. After decoding from Base64, the value is interpreted as a binary representation of a string according to the specified character set.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_base64Decode` | The name of the field where the decoded value will be stored. |
| **charset** | string | optional <br> default: `UTF-8` | The character set to use when converting the decoded bytes into a string. Possible values are `UTF-8`, `UTF-16BE`, `UTF-16LE`, and `ISO-8859-1`. |
| **field** | string | required | The field containing the Base64 value to be decoded. The parameter name `field` can be omitted. |

***

## Function Operation

The function first decodes a Base64 string into its binary representation. It then transforms these bytes into a string using the specified `charset`. To work with raw binary data, the `ISO-8859-1` character set should be used.

### Behavior on Invalid Encoding
* Characters outside the standard Base64 alphabet are ignored.
* If the input string is not a valid Base64 encoding, the output field will not be created.
* If a decoded code point is invalid for the selected `charset`, it is replaced with a placeholder character.

***

## Example

### Perform Base64 Decoding of a Field

This example demonstrates how to decode a Base64-encoded string from a field named `encoded` and store the result in a new field named `decoded`.

* **Query Example**
    ```
    decoded := base64Decode(encoded)
    ```

* **Step-by-Step**
    1.  The query starts with events that have a field named `encoded` containing a Base64 string.
    2.  The `base64Decode()` function reads the value from the `encoded` field.
    3.  It decodes the Base64 value and stores the resulting string in a new field called `decoded`.