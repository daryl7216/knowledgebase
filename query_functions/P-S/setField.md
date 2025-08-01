# setField()

The `setField()` function takes two expressions, `target` and `value`, and sets the field named by the result of the `target` expression to the result of the `value` expression. It can be used to manipulate fields whose names are not statically known, but computed at runtime. This function is available from v1.127.0.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `target` | expression | required \<br\> default: `setField` | Evaluates the name of the field to set. |
| `value` | expression | required | An expression whose result becomes the value for the field. |

***

## Function Operation

The `setField()` function is used to dynamically create or modify fields within an event, where the name of the target field can be determined at runtime based on an expression. This distinguishes it from direct field assignments (`fieldName := value`), which require the field name to be static.

The function takes two primary arguments:

  * `target`: An expression that, when evaluated, produces the name of the field to be set. This allows for flexible field naming.
  * `value`: An expression whose evaluation result becomes the content of the target field. This expression can be a simple literal, another field's value, or the result of a more complex calculation, including the use of other functions like `if()`.

`setField()` enables advanced field manipulation scenarios where field names are not known beforehand or need to be constructed based on event data.

***

## Examples

### Set Values for Multiple Fields

This example demonstrates how to set values for multiple fields, including one where the target field name is derived from another field's content, and another where the value is conditionally determined by an `if()` function.

  * **Query Example**

    ```
    item := 4
    | bar := "baz"
    | setField(target=bar, value=item + 10)
    | setField(target="foo", value=item + 20)
    | setField(target="baaz", value=if(item == 4, then="OK", else="not OK"))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `item := 4`: In a test event, the field `item` is initialized with the value `4`.
    3.  `| bar := "baz"`: A field `bar` is created and assigned the string value `"baz"`.
    4.  `| setField(target=bar, value=item + 10)`: This `setField()` call takes the *value* of `bar` (which is "baz") as its `target` field name. It then sets the field named `baz` to the result of `item + 10` (which is `4 + 10 = 14`).
    5.  `| setField(target="foo", value=item + 20)`: This call sets a new field explicitly named `"foo"` to the result of `item + 20` (which is `4 + 20 = 24`).
    6.  `| setField(target="baaz", value=if(item == 4, then="OK", else="not OK"))`: This adds an `if()` function to determine the value of the new target field `baaz`. If `item` is equal to `4` (which it is), then `baaz`'s value is "OK"; otherwise, it would be "not OK".
    7.  Event Result set.

  * **Summary and Results**
    This query demonstrates setting values for different target fields as the result of given expressions. It highlights how functions can be embedded within the `value` parameter to determine the content of dynamically named fields.

  * **Result Event Data**

| baaz | bar | baz | foo | item |
| :--- | :--- | :--- | :--- | :--- |
| OK | baz | 14 | 24 | 4 |

---

### Set the Value of a Field

This example sets the value of a target field based on the result of an expression. This is equivalent to direct field assignment.

  * **Query Example**

    ```
    item := 4
    setField(target="foo", value=item + 10)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `item := 4`: In a test event, the field `item` is set to `4`.
    3.  `setField(target="foo", value=item + 10)`: Sets the value of the target field `foo` as the result of the expression `item + 10` (which is `4 + 10 = 14`). This is equivalent to `foo := item + 10`.
    4.  Event Result set.

  * **Summary and Results**
    This query uses `setField()` to assign a calculated value to a specific field.

  * **Result Event Data**

| foo | item |
| :--- | :--- |
| 14 | 4 |