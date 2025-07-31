# writeJson()

The `writeJson()` function takes a list of field names, which can include dot notation for nesting and square brackets for arrays, and converts them into a structured JSON object.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_json` | The name of the output field where the JSON object will be stored. |
| **field** | array of strings | optional <br> default: `@rawstring` | The values and fields to be converted to JSON. The parameter name `field` can be omitted. |

***

## Function Operation

This function reads a list of flat field names and intelligently reconstructs a nested JSON structure from them. The field names provided are treated as prefix matches, meaning `writeJson(field=["a.b"])` would match fields like `a.b.c` and `a.b[0]`. It also supports array-globbing patterns, so `writeJson(field=["a.b[*]"])` would match all elements within the `a.b` array.

***

## Example

### Convert Flat Fields to a Nested JSON Object

This example demonstrates how to take several flat fields that represent a nested structure and convert them back into a single, structured JSON object.

* **Query Example**
    ```
    writeJson(["a.b.c", "a.b.e[0]", "a.d", "a.f.g"])
    ```

* **Input Event Data**
    Events with several flat fields that use dot notation to imply a nested structure.

| a.b.c | a.b.e[0] | a.d | a.f.g |
| :--- | :--- | :--- | :--- |
| value1 | value2 | value3 | value4 |
| test1 | test2 | test3 | test4 |
| data1 | data2 | data3 | data4 |

* **Step-by-Step**
    1.  The query starts with events containing the flattened fields.
    2.  `writeJson()` takes the array of field paths as input.
    3.  It processes the dot notation (`a.b.c`) to create nested objects and the square bracket notation (`a.b.e[0]`) to create arrays.
    4.  It combines all the specified paths into a single JSON object and stores it in the default `_json` field.

* **Result Event Data**
    The output shows the original flat fields along with the new `_json` field, which contains the reconstructed nested JSON object.

| _json | a.b.c | a.b.e[0] | a.d | a.f.g |
| :--- | :--- | :--- | :--- | :--- |
| `{"a":{"b":{"c":"value1","e":["value2"]},"d":"value3","f":{"g":"value4"}}}` | value1 | value2 | value3 | value4 |
| `{"a":{"b":{"c":"test1","e":["test2"]},"d":"test3","f":{"g":"test4"}}}` | test1 | test2 | test3 | test4 |
| `{"a":{"b":{"c":"data1","e":["data2"]},"d":"data3","f":{"g":"data4"}}}` | data1 | data2 | data3 | data4 |