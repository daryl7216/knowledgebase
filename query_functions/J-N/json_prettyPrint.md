# json:prettyPrint()

The `json:prettyPrint()` function in LogScale formats a JSON field for improved readability. This can be an expensive operation, so it is recommended to apply it after filtering your data at the end of the query to prevent unnecessary formatting of data that will be discarded.

***

## Parameters:

| Parameter | Type | Required | Default Value | Description |
| :-------- | :----- | :-------- | :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `as` | string | optional | (none) | The name of the field to store the output in. |
| `field` | string | optional | `@rawstring` | The name of the field to format. This parameter name can be omitted. |
| `step` | number | optional | 2 | The indentation in number of characters, with a minimum of 2 spaces. |
| `strict` | boolean | optional | `false` | If set to `true`, only valid JSON input will produce a value in the output field. By default, invalid JSON is copied to the output field unmodified. |

***

## Function Operation:

The `json:prettyPrint()` function takes a JSON field and formats it for better readability. If no field is specified, the `@rawstring` field will be formatted. If the specified field does not contain valid JSON, the unmodified input value is stored in the output field, unless the `strict` parameter is set to `true`. When `strict` is `true`, invalid JSON input will not produce a value in the output field. It is advised to use this function after filtering data to reduce processing overhead.

***

## Examples:

### 1\. Format JSON

This example demonstrates how to format JSON in the `@rawstring` field using the `json:prettyPrint()` function.

* **Query Example:**

```
#type=json
| account = 123
| json:prettyPrint()
```

* **Input Event Data:**
(No input event data provided in the source for this example, only the query and steps.)

* **Step-by-Step:**

1.  Start with the source repository events.
2.  `#type=json`: This filters for events with the JSON type.
3.  `| account = 123`: This filters for events related to account `123`. It is recommended to filter the event set as much as possible before using `json:prettyPrint()` to prevent unnecessary formatting of discarded data.
4.  `| json:prettyPrint()`: This formats the JSON content for improved readability. Without a specified field, it formats the `@rawstring` field. If the input is not valid JSON, it returns unmodified values; to prevent this, the `strict` parameter can be set.
5.  Event Result set.

* **Result Event Data:**
The query is used to make JSON data more readable in the results. Formatting JSON in the `@rawstring` field after filtering data is important as it is a resource-intensive operation. Without the `strict` parameter set to `true`, the `json:prettyPrint()` function attempts to format even invalid JSON, which might lead to unexpected results.

---

### 2\. Format Only Valid JSON

This example shows how to format only valid JSON data using the `json:prettyPrint()` function with the `strict` parameter.

* **Query Example:**

```
formattedJson := json:prettyPrint(field=message, strict=true)
```

* **Input Event Data:**
(No input event data provided in the source for this example, only the query and steps.)

* **Step-by-Step:**

1.  Start with the source repository events.
2.  `formattedJson := json:prettyPrint(field=message, strict=true)`: This uses the `field` parameter to specify the `message` field as the source of JSON data and sets the `strict` parameter to `true` to only process valid JSON. It formats the valid JSON data for improved readability and assigns the results to a new field named `formattedJson`. If the JSON in the `message` field is invalid, the `formattedJson` field will not be created for that event because the `strict` parameter is set to `true`.
3.  Event Result set.

* **Result Event Data:**
The query is used to make valid JSON data more readable in the results. Without the `strict` parameter set to `true`, the `json:prettyPrint()` function attempts to format even invalid JSON, which might lead to unexpected results.