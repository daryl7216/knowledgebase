# parseJson()

The `parseJson()` function parses data or a field as JSON, converting the data into named fields and arrays within the event. Use `parseJson(field=@rawstring)` to parse the original ingested rawstring into JSON. Use the `prefix` parameter to prefix the names of the extracted fields. Furthermore, to exclude some of the extracted fields, use the `exclude` parameter to specify the JSON object structure to be excluded. For example, use `parseJson(field=input,exclude=a.b.c)` to exclude `c` and all of its descendants (a path-based exclusion) or use `parseJson(field=input,exclude="a.b[*].c")` to exclude all inside the array `b` (array-based exclusion). Note that all non-nested fields are also excluded. Use the `include` parameter if you need to keep certain descendants of an otherwise excluded path or an excluded non-nested field in the output.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `exclude` | array of strings | optional \<br\> default: `[]` | Removes specified JSON fields from the output. Use on fields that should be excluded from the result. Supports dot-pathing and array wildcards. If used with `prefix`, the excluded fields will use the specified prefix. The exclusion works as a wildcard match for the given field; for example, the value `query` will match both nested fields (like `query.string`) and similarly non-nested named fields (like `queryString`, `queryStart`, and `queryEnd`). To retain specific fields while excluding others, use the `include` parameter. |
| `excludeEmpty` | boolean | optional \<br\> default: `false` | Whether to exclude if the field is empty. \<br\>**Values:**\<br\>`false`: Do not exclude the field, even if the value is empty. \<br\>`true`: Exclude the field if the value is empty. |
| `field` | string | required \<br\> default: `@rawstring` | Fields that should be parsed as JSON. |
| `handleNull` | string | optional \<br\> default: `keep` | How null values are handled. \<br\>**Values:**\<br\>`discard`: Discard the null value and field. \<br\>`empty`: Replaces a null value with an empty string `""`. \<br\>`keep`: Converts the value to the `"null"` string. |
| `include` | array of strings | optional \<br\> default: `[]` | Retains specific descendants of excluded paths. Use on fields that should be included, even if they had been previously excluded by use of `exclude`. Supports dot-pathing and array wildcards. If used with `prefix`, the included fields will also use the specified prefix. |
| `prefix` | string | optional \<br\> default: (blank) | Prefix the name of the extracted JSON fields with the value of this parameter. |
| `removePrefixes` | array of strings | optional \<br\> default: `[]` | Prefixes that should be removed from the names of the extracted JSON fields; supports dot-pathing. If multiple prefixes are supplied, the longest matching prefix will be used. |

***

## Function Operation

The `parseJson()` function is used to convert JSON-formatted data into structured fields and arrays within a LogScale event. It primarily parses the field specified by `field` (defaulting to `@rawstring`).

When parsing JSON, the following rules apply:

  * **`excludeEmpty=true`**: Key/value pairs where the value is empty (`""`) will be completely discarded.
  * **`handleNull`**:
      * If `handleNull` is set to `discard`, any key/value pair with a `null` value will be discarded entirely, regardless of the `excludeEmpty` setting.
      * If `handleNull` is set to `empty`, `null` values are replaced with an empty string (`""`).
      * If `handleNull` is set to `keep` (default), `null` values are converted to the string `"null"`.

The `prefix` parameter allows adding a custom prefix to the names of all extracted JSON fields, which is useful for avoiding naming collisions with existing fields. Conversely, `removePrefixes` can be used to remove specified prefixes from the extracted field names; if multiple prefixes match, the longest one is removed.

The `exclude` parameter allows for the removal of specific JSON fields or structures from the output. It supports dot-pathing for nested objects and array wildcards (`[*]`) for arrays. When `exclude` is used, it acts as a wildcard match, meaning a value like `query` will exclude both nested fields (e.g., `query.string`) and similarly named non-nested fields (e.g., `queryString`). The `include` parameter can be used in conjunction with `exclude` to selectively retain certain descendants of an otherwise excluded path or a non-nested field. If both `include` and `exclude` are used with `prefix`, the `include` and `exclude` paths must also be prefixed.

***

## Examples

### Parse JSON Content With Specific Parameters

This example parses JSON content with specific parameters, excluding the actual query content while including only the `queryStart` field.

  * **Query Example**

    ```
    #type=audit-log
    | /"type": "alert.update"/
    parseJson(exclude="query", include="queryStart")
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=audit-log`: Filters for all events from the `audit-log` repository.
    3.  `|"type": "alert.update"/`: Filters for events (audit logs) where the `@rawstring` field contains the string `"type": "alert.update"`.
    4.  `parseJson(exclude="query", include="queryStart")`: Parses the JSON content. With specific parameters set, it excludes the `query` field and includes only the `queryStart` field. The exclusion works as a wildcard match for the given field; for example, the value `query` will match both nested fields (like `query.string`) and similarly non-nested named fields (like `queryString`, `queryStart`, and `queryEnd`). In this example, to retain the specific non-nested field `queryStart` while excluding the others, the `include` parameter is used.
    5.  Event Result set.

  * **Summary and Results**
    The query is used to search audit logs for alert update events, specifically looking at the non-nested `queryStart` field, while excluding the actual query content (`query.xxx`). This query is useful if, for example, you want to review alert activity without the overhead of full query contents, track temporal patterns in alert updates, or investigate alert timing issues further.

---

### Create Two Temporary Events for Troubleshooting - Example 2

This example demonstrates creating two temporary events for testing or troubleshooting using the `createEvents()` function with `parseJson()`.

  * **Query Example**

    ```
    createEvents(["{\"animal\":{\"kind\":\"dog\", \"weight\":7.0}}", "{\"animal\":{\"kind\":\"cat\", \"weight\":4.2}}"])
    | parseJson()
    ```

  * **Input Event Data**
    (Representing the raw JSON strings created by `createEvents()`):

    ```json
    [
      {"animal":{"kind":"dog", "weight":7.0}},
      {"animal":{"kind":"cat", "weight":4.2}}
    ]
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `createEvents(["{\"animal\":{\"kind\":\"dog\", \"weight\":7.0}}", "{\"animal\":{\"kind\":\"cat\", \"weight\":4.2}}"])`: Creates two temporary events, one with information about a dog and another about a cat.
    3.  `| parseJson()`: Parses the raw string content of these newly created events as JSON.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to create temporary events and parse their raw string content as JSON.

  * **Result Event Data**

| @timestamp | animal.kind | animal.weight |
| :--- | :--- | :--- |
| 1733311547717 | dog | 7.0 |
| 1733311547717 | cat | 4.2 |