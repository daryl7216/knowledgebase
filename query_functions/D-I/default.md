# default()

The `default()` function sets a default value for one or more fields. If a specified field does not exist in an event, this function creates it with the given value. If the field already exists, its original value is kept.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | string or array | required | The field or fields to set the default value for. |
| **replaceEmpty** | boolean | optional <br> default: `false` | If set to `true`, the function will also override fields that exist but have an empty string as their value. |
| **value** | string | required | The default value to assign to the field(s). The parameter name `value` can be omitted. |

***

## Example

### Set a Default Value and Replace Empty Fields

This example shows how to ensure a `message` field always has a value by setting a default. It uses the `replaceEmpty` parameter to handle cases where the field exists but is an empty string, which is important for data normalization.

* **Query Example**
    ```
    default(field=message, value="N/A", replaceEmpty=true)
    ```

* **Step-by-Step**
    1.  The query starts with a set of events.
    2.  For each event, the `default()` function checks the `message` field.
    3.  If the `message` field does not exist, it is created with the value `"N/A"`.
    4.  Because `replaceEmpty=true` is set, if the `message` field exists but its value is an empty string (`""`), its value is also changed to `"N/A"`.
    5.  If the `message` field exists and has a non-empty value, it remains unchanged.

* **Summary and Results**
    This query is crucial for data preparation, especially before using functions like `eval()` that require fields to be present. It ensures that every event will have a consistent, non-empty `message` field, which can be critical for analysis and quick triage in security or operational logs.