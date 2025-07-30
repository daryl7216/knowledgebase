# bitfield:extractFlagsAsString()

The `bitfield:extractFlagsAsString()` function decodes an integer into its bit representation and extracts the names of the "true" (set) bits into a single, delimited string.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_flags` | The name of the output field that will store the string of flag names. |
| **field** | string | required | The name of the field containing the integer to be decoded. |
| **flagNames**| array of arrays of strings | required | A list of pairs, where each pair contains a bit index (0-63) and the corresponding flag name. |
| **separator**| string | optional <br> default: ` ` (space) | The separator to use between the names of the true flags in the output string. |

***

## Function Operation

This function reads an integer value, converts it to its binary form, and identifies which bits are set to 1. It then collects the corresponding names from the `flagNames` list and joins them into a single string, using the specified `separator`. The flag names in the output string are always listed in ascending order, from the lowest bit to the highest.

**Validation:**
* In the `flagNames` parameter, bit indices must be unique, non-negative numbers below 64.
* The flag names themselves must also be unique.

***

## Example

### Decode and Extract True Bits as a Comma-Separated String

This example decodes an integer, extracts the names of the true bits, and formats them as a comma-separated string in a custom field named `trueFlags`.

* **Query Example**
    ```
    bitfield:extractFlagsAsString(
        field="MemoryDescriptionFlags", 
        flagNames=[
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
        ], 
        as="trueFlags", 
        separator=", "
    )
    ```

* **Input Event Data**
    An event where the field `MemoryDescriptionFlags` has a value of `1234`.
    ```
    MemoryDescriptionFlags, 1234
    // 1234 in binary is 10011010010
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `MemoryDescriptionFlags` field.
    2.  `bitfield:extractFlagsAsString()` reads the integer `1234` and identifies the bits that are set to `1`.
    3.  It collects the names for these true bits (e.g., `CURRENT_STACK`, `CODE`, `MAPPED`, etc.).
    4.  The function then joins these names into a single string, separated by a comma and a space, and stores it in the `trueFlags` field.

* **Result Event Data**
    The resulting event has a new field, `trueFlags`, containing the comma-separated string of active flags.
    ```
    trueFlags="CURRENT_STACK, CODE, MAPPED, REFLECIVE_PE, PRIVATE_MEMORY"
    ```