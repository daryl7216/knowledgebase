# getField()

The `getField()` function retrieves the value of a field whose name is not known beforehand but is instead computed dynamically at runtime.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_getField` | The name of the output field. |
| **source**| expression | required | An expression that evaluates to the name of the field you want to read. |

***

## Function Operation

This function works by first evaluating the `source` expression to get a string, and then it uses that resulting string as the name of the field to read from the event. This is particularly useful in two scenarios:
1.  When a field's name is constructed dynamically, for example, based on the a value from another field.
2.  When a field's name contains special characters like spaces or hyphens (e.g., `"host-name"`), which cannot be referenced directly in other functions.

***

## Examples

### Get the Last Element of an Array

This example demonstrates how to dynamically construct a field name to get the last element of an array, whatever its length may be.

* **Query Example**
    ```
    index := array:length("foo[]") - 1
    | fieldName := format("foo[%s]", field=[index])
    | result := getField(fieldName)
    ```

* **Input Event Data**
    An event with an array field:
    `{"foo": ["a", "b", "c", "d"]}`

* **Step-by-Step**
    1.  `array:length("foo[]") - 1`: First, the query calculates the index of the last element in the `foo` array (in this case, `4 - 1 = 3`).
    2.  `format("foo[%s]", ...)`: Next, it uses the calculated index to construct the full field name as a string: `"foo[3]"`. This string is stored in a new field called `fieldName`.
    3.  `getField(fieldName)`: Finally, `getField()` uses the value of the `fieldName` field (`"foo[3]"`) as the name of the field to look up, retrieving the value `d`.

* **Result Event Data**

| fieldName | result |
| :--- | :--- |
| foo[3] | d |

---

### Get a Value with a Direct vs. Indirect Field Name

This example clearly shows the difference between providing a static string (direct lookup) and providing a field name (indirect/dynamic lookup) to `getField()`.

* **Input Event Data**
    An event with two fields, where the value of `foo` is the name of another field, `bar`.
    ```
    foo: "bar"
    bar: 123
    ```

* **Query 1: Direct Lookup (Static Name)**
    ```
    result := getField("foo")
    ```
    **Result**: This looks for a field literally named "foo" and returns its value. `result` will be `"bar"`.

* **Query 2: Indirect Lookup (Dynamic Name)**
    ```
    result := getField(foo)
    ```
    **Result**: This first evaluates the `foo` field to get its value (`"bar"`). It then looks for a field with that name (`bar`) and returns *its* value. `result` will be `123`.