# kvParse()

The `kvParse()` function in LogScale is used to parse events encoded with key/value pairs into individual fields and values. It can run an extra key/value parser on events.

***

## Parameters:

| Parameter | Type | Required | Default Value | Description |
| :-------- | :------------- | :-------- | :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `as` | string | optional | (none) | Prefix for all resolved field keys. |
| `excludeEmpty` | array of strings | optional | `false` | If the value of a key is empty, exclude the field. |
| `field` | array of strings | optional | `@rawstring` | Fields that should be key-value parsed. The parameter name `field` can be omitted. |
| `override` | boolean | optional | `false` | Override existing values for keys that already exist in the event. |
| `separator` | string | optional | `=` | The token that separates the key from the value (a single char only). |
| `separatorPadding` | string | optional | `unknown` | Helps the function recognize unquoted empty values and parse them by specifying whether there is a whitespace around the key-value separator (typically). Values: `no` (assumes no whitespace), `unknown` (whether padding is not recognized), `yes` (indicates whitespace). |

***

## Function Operation:

The `kvParse()` function is used to parse key/values that follow specific formats, such as `key=value`, `key="value"`, `key='value'`, or `key value`. Both the key and value can be quoted using single or double quotes, and if quotes are used, they must be terminated. For a log line like `2017-02-22T13:14:01.917+0000 [main thread] INFO UserService creating new user id=123, name='john doe' email=john@doe`, the key/value parser extracts fields like `id=123`, `name=john doe`, and `email=john@doe`. The `field` parameter specifies which fields should be key/value parsed, with `@rawstring` being the default for parsing the raw string.

***

## Examples:

### 1\. Key/value parse a nested field (JSON input)

This example demonstrates how to key/value parse a nested field within a JSON input.

* **Query Example:**

```
parseJson()
kvParse(metadata)
```

* **Input Event Data:**

```json
{
  "service": "paymentService",
  "type": "payment",
  "metadata": "host=server5, transactionID=123, processingTime=100"
}
```

* **Step-by-Step:**

1.  The `parseJson()` function parses the JSON input, making `service`, `type`, and `metadata` accessible as fields.
2.  The `kvParse(metadata)` function then parses the `metadata` field, which contains key/value pairs.

* **Result Event Data:**
(The output event data is not explicitly provided in the source for this example, but the `metadata` field would be parsed into `host`, `transactionID`, and `processingTime` fields.)

---

### 2\. Keep the first value for duplicated keys, with a preset field

This example shows how to handle duplicated keys when a field has a preset value, ensuring the first encountered value is kept.

* **Query Example:**

```
kvParse(onDuplicate=keepFirst, override=true)
```

* **Input Event Data:**

```
name='john doe' name='jane doe'
```

* **Step-by-Step:**

1.  The `kvParse()` function is applied to the input string.
2.  `onDuplicate=keepFirst`: This parameter specifies that if duplicate keys are encountered, the first value for that key should be retained.
3.  `override=true`: This parameter ensures that if a field named `name` already exists (e.g., if it was preset to 'alice'), the value from the `kvParse` operation will override the existing value.

* **Result Event Data:**

```
name: john doe
```

---

### 3\. Parse String as CSV

This example demonstrates how to parse a string that contains key/value pairs and then a CSV-encoded field, using `kvParse()` followed by `parseCsv()`.

* **Query Example:**

```
kvParse()
parseCsv(result, columns=[count, status, completion, precision, sourcetask])
```

* **Input Event Data:**

```
2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117, success,27%,3.14"
```

* **Step-by-Step:**

1.  `kvParse()`: This parses the initial string into key-value pairs. In this input, it would likely extract `result="117, success,27%,3.14"`.
2.  `parseCsv(result, columns=[count, status, completion, precision, sourcetask])`: This then takes the `result` field (extracted by `kvParse()`) and parses its CSV-encoded content into the specified columns. It adds fields like `count` with value `117`, `status` with `success`, `completion` with `27%`, and `precision` with `3.14`.

* **Result Event Data:**

| completion | count | precision | result | status |
| :--------- | :---- | :-------- | :----- | :------- |
| 27% | 117 | 3.14 | 117, success, 27%, 3.14 | success |