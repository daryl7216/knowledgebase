# parseHexString()

The `parseHexString()` function parses the input from hex encoded bytes, decoding the resulting bytes as a string using the selected character set. If the input field has a prefix (other than `0x` and `16#`), then use `regex()` or `replace()` to remove the prefix before using `parseHexString()`. Any non-hex characters in the input are ignored; the decoding attempts to decode all characters in the field that match `0-9` or `A-F`. Case is ignored.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_parsehexstring` | Name of output field. |
| `charset` | string | optional \<br\> default: `UTF-8` | The charset to use when transforming bytes to string. \<br\>**Values:** `ISO-8859-1`, `UTF-16BE`, `UTF-16LE`, `UTF-8`. |
| `field` | string | required | Specifies the field containing the hex string to use as input. |

***

## Function Operation

The `parseHexString()` function converts a hexadecimal string from a specified input field into a readable string format. It decodes the hex-encoded bytes using the character set defined by the `charset` parameter (defaulting to `UTF-8`). The function is flexible in handling input, ignoring any non-hexadecimal characters and processing only those that match `0-9` or `A-F`, regardless of case. If the hexadecimal input string has prefixes other than `0x` or `16#`, these should be removed beforehand using functions like `regex()` or `replace()` for proper parsing. The resulting decoded string is then stored in a new output field, which by default is named `_parsehexstring` but can be customized with the `as` parameter.

***

## Examples

### Parses a hex string from field `foo` into field `text`

This example parses the hex string `48656c6c6f576f726c64` from the field named `foo` into the field `text`, resulting in the value "Helloworld" using `ISO-8859-1` character set.

  * **Query Example**

    ```
    foo: 48656c6c6f576f726c64
    parseHexString(foo, as=text, charset="ISO-8859-1")
    ```

  * **Step-by-Step**

    1.  The query starts by defining a field `foo` with the hexadecimal string `48656c6c6f576f726c64`.
    2.  `parseHexString(foo, as=text, charset="ISO-8859-1")` then parses the content of the `foo` field, interprets it as a hexadecimal string, and decodes it into a human-readable string using the `ISO-8859-1` character set. The result is stored in a new field named `text`.

  * **Result Event Data**
    (Implicitly, an event will contain the field `text` with the value "Helloworld")

---

### Parses a complex hex string from field `hex` into field `text`

This example parses a string containing `0x` prefix, spaces, and non-hex characters, along with a smiley emoji hex code, from the field `hex` into the field `text` using `UTF-8` charset, getting the value "Helloworld" followed by a smiley.

  * **Query Example**

    ```
    hex := "0x4 865 6c6c6f576f726c6420 plus F09F 98 80"
    | text := parseHexString(hex)
    ```

  * **Step-by-Step**

    1.  The query starts by assigning the string `"0x4 865 6c6c6f576f726c6420 plus F09F 98 80"` to a field named `hex`.
    2.  `| text := parseHexString(hex)` then parses the content of the `hex` field. The `parseHexString` function automatically handles the `0x` prefix and ignores non-hexadecimal characters like spaces and "plus ". It decodes the valid hexadecimal sequences (`48656c6c6f576f726c6420` and `F09F9880`) into a string. The result is stored in a new field named `text`.

  * **Result Event Data**
    (Implicitly, an event will contain the field `text` with the value "Helloworld 😊")
    (The smiley is the result of decoding the trailing digits `F09F9880` in UTF-8).