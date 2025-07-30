# bitfield:extractFlags()

The `bitfield:extractFlags()` function decodes an integer into its bit representation and extracts the bits at specified positions, creating new boolean fields for each specified bit.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | string | required | The name of the field containing the integer to be decoded. |
| **onlyTrue** | boolean | optional <br> default: `false` | If set to `true`, new fields will only be created for bits that are `true`. Bits that are `false` will be ignored. |
| **output** | array of arrays of strings | required | A list of pairs, where each pair contains a bit index (0-63) and the corresponding field name to be created. |

***

## Function Operation

This function reads an integer value from the specified `field`, converts it into its binary form, and then checks the state of specific bits as defined in the `output` parameter. For each bit index and name pair provided, a new field is created with a boolean value (`true` or `false`) representing whether that bit was set (1) or not (0). The bits are indexed from 0 and can handle integers up to 64 bits long.

***

## Example

### Decode and Extract Bit Flags

This example decodes an integer from the `MemoryDescriptionFlags` field and extracts several named flags, creating new fields only for the flags that are true.

* **Query Example**
    ```
    bitfield:extractFlags(field="MemoryDescriptionFlags", onlyTrue=true, output=[
        [0, INVALID_ADDRESS],
        [1, CURRENT_STACK],
        [2, JIT_DOTNET],
        [3, MZ],
        [4, CODE],
        [5, MODULE],
        [6, MAPPED],
        [7, REFLECIVE_PE],
        [8, JIT_FLASH],
        [9, PRIMARY_MODULE],
        [10, PRIVATE_MEMORY],
        [11, KNOWN_FUNCTION],
        [12, FREE_MEMORY]
    ])
    ```

* **Input Event Data**
    An event where the field `MemoryDescriptionFlags` has a value of `1234`.
    ```
    MemoryDescriptionFlags, 1234
    // 1234 in binary is 10011010010
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `MemoryDescriptionFlags` field.
    2.  `bitfield:extractFlags()` reads the integer `1234`.
    3.  It checks the state of bits 0 through 12 as defined in the `output` array.
    4.  Because `onlyTrue=true` is set, it creates new fields only for the bits that are set to 1 in the binary representation of 1234.

* **Result Event Data**
    The resulting event has new boolean fields corresponding to the "on" bits from the input value.

| CODE | CURRENT_STACK | FREE_MEMORY | INVALID_ADDRESS |
| :--- | :--- | :--- | :--- |
| true | true | false | false |