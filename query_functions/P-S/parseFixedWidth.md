# parseFixedWidth()

The `parseFixedWidth()` function parses a fixed width-encoded field into known columns. It can parse values of the form:
`value 1 value 2 value 3`, widths `[8,8,8]`
`value 1value 2value 3`, widths `[7,7,7]`

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `columns` | array of strings | required | Names of columns to extract from field. |
| `field` | string | optional \<br\> default: `@rawstring` | Field that holds the input in fixed width form. |
| `trim` | boolean | optional \<br\> default: `true` | Remove leading and trailing white-space from fields after extracting. |
| `widths` | array of numbers | required | Widths of columns. |

***

## Function Operation

The `parseFixedWidth()` function is used to extract data from a field where values are arranged in fixed-width columns. It requires two main parameters: `columns`, which specifies the names for the new fields to be extracted, and `widths`, which provides an array of numbers indicating the exact width of each corresponding column in the input string.

By default, the function processes the `@rawstring` field, but a different input field can be specified using the `field` parameter. The `trim` parameter, which defaults to `true`, ensures that leading and trailing whitespace is removed from the extracted field values. If `trim` is set to `false`, whitespace will be retained.

This parsing method is particularly valuable for structured data that maintains strict positional formatting and character length requirements. If an extracted field value is longer than its specified width, the parser truncates the excess data while maintaining the overall structure of the parsed output.

***

## Examples

### Fixed width parse the result field from a log line

This example demonstrates parsing a fixed width-encoded field named `result` from a log line.

  * **Query Example**

    ```
    parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])
    ```

  * **Input Event Data**

    ```
    2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117success 27% 3.14"
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])`: Parses the fixed width-encoded `result` field from the log line.
    3.  Event Result set.

  * **Summary and Results**
    The query adds the following fields to the event: `count` with value `117`, `status` with value `success`, `completion` with value `27%`, and `precision` with value `3.14`. The `sourcetask` field will not be set as the input is too short. Values are trimmed after they have been extracted, for example, " success" will become "success".

  * **Result Event Data**
    (Implicitly added fields based on parsing)

      * `count = 117`
      * `status = success`
      * `completion = 27%`
      * `precision = 3.14`

---

### Parse Fixed Width-encoded Log Lines Fields

This example parses fixed width-encoded fields from log lines into columns using the `parseFixedWidth()` function.

  * **Query Example**

    ```
    parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])
    ```

  * **Input Event Data**
    (Example log line assumed to contain a `result` field in fixed-width format, e.g., similar to "117success 27% 3.14")

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseFixedWidth(result, columns=[count, status, completion, precision, sourcetask], widths=[3,9,4,10,10])`: Parses the fixed width-encoded field in the access log and adds the returned values as known columns in the result field.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to parse compact numeric data consisting of fixed width-encoded fields into columns. In case a field value is longer than, for example, 10 characters, the parser handles overflow by truncating data that exceeds the specified field width while maintaining the structure of the parsed output. As an example, if the original `sourcetask` value was: `SCAN_FILES_WITH_VERY_LONG_NAME` (29 characters), then the extra characters `_WITH_VERY_LONG_NAME` would be truncated. This parsing method is particularly valuable when dealing with structured data that must maintain strict positional formatting and character length requirements.