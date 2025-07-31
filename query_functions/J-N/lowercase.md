# lowercase()

The `lowercase()` function is deprecated for use in query contexts and is meant for use in parsers only. For uses in queries, the `lower()` function should be used. It lowercases the contents of either or both of the field names or values of a string field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | array of strings | required | The name of the input field or fields (in `[]`) to lowercase. Use the special value `*` as the only field for ALL fields. When in this mode, only the lowercased fields remain. |
| `include` | string | optional \<br\> default: `values` | What to lowercase.\<br\>**Values:**\<br\>`both`: Convert both the values and field names to lowercase.\<br\>`fields`: Convert the field names to lowercase.\<br\>`values`: Convert the values of the fields to lowercase.|
| `locale` | string | optional | The name of the locale to use, as ISO 639 language and an optional ISO 3166 country, such as `da`, `da_DK` or `en_US`. When not specified, uses the system locale.|

***

## Function Operation

The `lowercase()` function is primarily intended for use within parsers, not directly in query contexts. It converts the contents of string field values, field names, or both, to lowercase. If `field` is set to `*`, only the lowercased fields will remain. The `locale` parameter allows specifying a particular language and country for correct lowercase conversion, otherwise, the system locale is used.

***

## Examples

### Lowercase Field Value

This example shows how to lowercase the value of a specific field.

  * **Query Example**

    ```
    lowercase("Bar")
    ```

  * **Input Event Data**
    (Assume an event with a field named `Bar-CONTENTS` where the value is "CONTENTS")

  * **Result Event Data**
    (The `Bar` field's content will be "contents")

---

### Lowercase Field Value (explicit `include`)

This example demonstrates lowercasing only the value of a field, explicitly specifying the `include` parameter.

  * **Query Example**

    ```
    lowercase("BaR", include="values")
    ```

  * **Input Event Data**
    (Assume an event with a field named `BAR CONTENTS` where the value is "CONTENTS")

  * **Result Event Data**
    (The `BaR` field's content will be "contents", while the field name `BAR` remains "CONTENTS")

---

### Lowercase Multiple Fields (both names and values)

This example shows how to lowercase both the field names and their values for specified fields.

  * **Query Example**

    ```
    lowercase(field=["foo", "bar"], include="both")
    ```

  * **Input Event Data**
    (Assume an event with fields `Foo` with value "VALUE1" and `BAR` with value "VALUE2")

  * **Result Event Data**
    (Fields `foo` with value "value1" and `bar` with value "value2" will be present)

---

### Lowercase All Fields (both names and values)

This example demonstrates lowercasing both the field names and their values for all fields in an event.

  * **Query Example**

    ```
    lowercase(field="*", include="both")
    ```

  * **Input Event Data**
    (Assume an event with a field named `BAR CONTENTS` where the value is "CONTENTS")

  * **Result Event Data**
    (The field name will be `bar contents` and its value will be "contents". The original `BAR CONTENTS` field will no longer be present.)