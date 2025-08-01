# parseInt()

The `parseInt()` function converts an integer from any radix (or number base), such as from hexadecimal or octal, to base 10, the decimal radix, expected as input by all other functions. For example, converting the hexadecimal FF to 255 using radix 16 or 77 to 63 using radix 8. The conversion is always unsigned. If the input field has a prefix (other than `0x` and `16#`) then use `regex()` to remove the prefix before using `parseInt()`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: (same as input field) | The output name of the field to set. |
| `endian` | string | optional \<br\> default: `big` | Input Digit-pair ordering (little, big) for hexadecimal. |
| `field` | string | required | The name of the input field. |
| `radix` | number | optional \<br\> default: `16` | Input Integer base (2 to 36). |

***

## Function Operation

The `parseInt()` function converts numerical strings from various radices (number bases) into their equivalent base 10 (decimal) integer representation. This conversion is always unsigned. It supports radices from 2 to 36. For hexadecimal inputs, you can specify the byte order (`endian`) as either `little` or `big`. If the input string has prefixes other than `0x` or `16#`, these must be removed prior to using `parseInt()` (e.g., with `regex()`). The converted decimal integer is then outputted to a new field, which by default will have the same name as the input field, but can be renamed using the `as` parameter.

***

## Examples

### Shows how to parse a hexadecimal string in little endian as an integer

This example demonstrates how to parse a hexadecimal string in little-endian format as an integer.

  * **Query Example**

    ```
    parseInt(hexval, as="centigrades", radix="16", endian="little")
    ```

  * **Input Event Data**
    An input event with the field `hexval` with the value `8001`.

  * **Result Event Data**
    The field `centigrades` having the value $(1 \* 256) + 128 = 384$.

---

### Shows how to parse a hexadecimal string in big endian as an integer

This example demonstrates how to parse a hexadecimal string in big-endian format as an integer.

  * **Query Example**

    ```
    parseInt(hexval, as="centigrades", radix="16", endian="big")
    ```

  * **Input Event Data**
    An input event with the field `hexval` with the value `8001`.

  * **Result Event Data**
    The field `centigrades` having the value $(128 \* 256) + 1 = 32769$.

---

### Shows how to parse a binary string as an integer

This example demonstrates how to parse a binary string as an integer.

  * **Query Example**

    ```
    parseInt(bitval, as="flags", radix="2")
    ```

  * **Input Event Data**
    An input event with the field `bitval` with the value `00011001`.

  * **Result Event Data**
    The field `flags` having the value $16 + 8 + 1 = 25$.