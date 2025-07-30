# bitfield:extractFlagsAsArray()

The `bitfield:extractFlagsAsArray()` function decodes an integer from a field into its bit representation and creates an array containing the names of the flags that correspond to "true" (set) bits.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **asArray** | string | optional <br> default: `_flags[]` | The name of the output array. |
| **field** | string | required | The name of the field containing the integer to be decoded. |
| **flagNames**| array of arrays of strings | required | A list of pairs, where each pair contains a bit index (0-63) and the corresponding flag name. |

***

## Function Operation

This function reads an integer value, converts it to its binary form, and then checks which bits are set to 1. For each bit that is set, the function takes the corresponding name from the `flagNames` list and adds it to the output array. The names in the output array are sorted in order from the lowest bit to the highest bit.

Important validation rules for the `flagNames` parameter:
* Bit indices must be non-negative numbers less than 64.
* Both the bit indices and the flag names must be unique within the list.

If an array with the specified output name already exists, it will be overwritten.

***

## Example

### Decode and Extract True Bits as an Array

This example decodes an integer from the `MemoryDescriptionFlags` field and creates an array containing the names of all the flags that are enabled.

* **Query Example**
    ```
    bitfield:extractFlagsAsArray(field="MemoryDescriptionFlags", flagNames=[
        [0, INVALID_ADDRESS],
        [1, CURRENT_STACK],
        [2, JIT_DOTNET],
        [3, MZ],
        [4, CODE],
        [5, MODULE],
        [6, MAPPED],
        [7, REFLECTIVE_PE],
        [8, JIT_FLASH],
        [9, PRIMARY_MODULE],
        [10, PRIVATE_MEMORY],
        [11, KNOWN_FUNCTION],
        [12, FREE_MEMORY]
    ], asArray="flagsArray[]")
    ```

* **Input Event Data**
    An event where the field `MemoryDescriptionFlags` has a value of `1234`.
    ```
    MemoryDescriptionFlags, 1234
    // 1234 in binary is 10011010010
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `MemoryDescriptionFlags` field.
    2.  `bitfield:extractFlagsAsArray()` reads the integer `1234` and converts it to its binary representation.
    3.  It then identifies which bits are set to `1` and collects the corresponding names from the `flagNames` list.
    4.  These names are placed into a new array called `flagsArray[]`.

* **Result Event Data**
    The output is a new array containing the names of all the flags that were set in the input value.

| flagsArray[0] | flagsArray[1] | flagsArray[2] | flagsArray[3] | flagsArray[4] |
| :--- | :--- | :--- | :--- | :--- |
| CURRENT_STACK | CODE | MAPPED | REFLECTIVE_PE | PRIVATE_MEMORY |