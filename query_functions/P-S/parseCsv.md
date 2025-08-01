# parseCsv()

The `parseCsv()` function parses a CSV-encoded field into known columns. It can parse values of the form:

  * `value 1, value 2, value 3`
  * `"value 1", "value 2", value 3` (Quoted values. Quotes are optional.)
  * `"value 1"; "value 2"; value 3` (Using `;` as delimiter. Delimiter is configurable.)

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `columns` | string or array | required | Names of columns to extract from field. |
| `delimiter` | string | optional \<br\> default: ` ,  ` | Delimiter character to split records by. |
| `excludeEmpty` | boolean | optional \<br\> default: `false` | If the value of a column is empty, exclude the field. |
| `field` | string | required \<br\> default: `@rawstring` | Field that holds the input in CSV form. |
| `trim` | boolean | optional \<br\> default: `false` | Allows to ignore whitespace before and after values. If the value is quoted, the quotes can start after the spaces. Useful for parsing data created by sources that do not adhere to the CSV standard. |

***

## Function Operation

The `parseCsv()` function takes a field containing CSV-formatted data and extracts its values into new, named fields. You must provide the expected column names using the `columns` parameter. The function can handle both unquoted and quoted values, and the delimiter character is configurable via the `delimiter` parameter (default is a comma).

The `excludeEmpty` parameter, if set to `true`, will prevent the creation of new fields for columns that have empty values. The `trim` parameter (default `false`) controls whitespace handling:

  * If `trim` is `true`, spaces around the delimiter are ignored, but spaces *within* quoted columns are retained. Quotes can appear after spaces. This is useful for parsing data that might not strictly adhere to CSV standards regarding whitespace.
  * If `trim` is `false` (default), spaces around the delimiter will be included as part of the field's value.

You can specify the input field to be parsed using the `field` parameter; if omitted, `@rawstring` is parsed by default.

***

## Examples

### Parse String as CSV

This example uses `parseCsv()` to parse a CSV-encoded field into known columns.

  * **Query Example**

    ```
    kvParse()
    parseCsv(result, columns=[count, status, completion, precision, sourcetask])
    ```

  * **Input Event Data**

    ```
    2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117, success,27%,3.14"
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs, extracting `result="117, success,27%,3.14"`.
    3.  `parseCsv(result, columns=[count, status, completion, precision, sourcetask])`: CSV parses the `result` field (extracted by `kvParse()`) and adds the following fields to the event: `count` with the value `117`, `status` with the value `success`, `completion` with the value `27%`, and `precision` with the value `3.14`. The `sourcetask` field will not be assigned a value as there were too few columns in the input for it.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to parse a string as CSV.

  * **Result Event Data**

| completion | count | precision | result | status |
| :--- | :--- | :--- | :--- | :--- |
| 27% | 117 | 3.14 | 117, success, 27%,3.14 | success |

---

### Parse String as CSV - Example 2

This example uses `parseCsv()` to parse a log line with spaces and quotes and trim the output by setting the `trim` parameter to `true`.

  * **Query Example**

    ```
    parseCsv(columns=[status, hosts, rest], trim=true)
    ```

  * **Input Event Data**
    (Assume `@rawstring` contains):
    `117, " crowdstrike.com, logscale.com ", 3.14`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseCsv(columns=[status, hosts, rest], trim=true)`: CSV parses the `@rawstring` (or implied `field` if used in a pipeline) and adds the following fields to the event: `status` with the value `117`, `hosts` with the value `" crowdstrike.com, logscale.com "`, `rest` with the value `3.14`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to parse a string as CSV. When `trim` is set to `true`, spaces around the separation character (e.g., a comma) are ignored, but spaces are retained within quoted columns. For example:

      * `117, crowdstrike.com, humio.com` would identify three columns: `117`, `crowdstrike.com`, `humio.com`.
      * `117," crowdstrike.com, humio.com ",3.14` would retain the spaces at the beginning and end of the quoted column.
        Without `trim` (`trim=false`), spaces around the character separator would be included in the values. For example, `117, "crowdstrike.com, humio.com "` would identify three columns where the comma between the two hostnames is interpreted as a separator, if the quotation mark after the space does not start a quoted value.

---

### Parse a CSV-encoded Field Into Known Columns

This example parses a CSV-encoded field into known columns and adds it to the event using the `parseCsv()` function.

  * **Query Example**

    ```
    parseCsv(result, columns=[count, status, completion, precision, sourcetask])
    ```

  * **Input Event Data**
    (Assume a log line from which a `result` field can be extracted, e.g., similar to "2017-02-22T13:14:01.917+0000 [main thread] INFO statsModule got result="117, success,27%,3.14"")

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseCsv(result, columns=[count, status, completion, precision, sourcetask])`: Parses a CSV-encoded field into known columns. In this example, where the parsed field is from a log line, using `parseCsv(result, columns=[count, status, completion, precision, sourcetask])` will add these fields to the event: `count: 117`, `status: success`, `completion: 27%`, `precision: 3.14`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to parse a CSV-encoded field into known columns. CSV files are often used to exchange data between systems.