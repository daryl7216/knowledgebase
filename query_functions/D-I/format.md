# format()

The `format()` function creates formatted strings using printf-style specifiers and adds the result to a new field. It can format a single field or an array of fields.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_format` | The name of the output field for the formatted string. |
| **field** | array of strings | required | The field or fields to format. |
| **format**| string | required | The formatting codes (e.g., `"%s, %.2f"`) for the given fields. The parameter name `format` can be omitted. |
| **timezone**| string | optional | The timezone to use when formatting date and time values. |

***

## Function Operation

The `format()` function uses a format string containing specifiers that define how to represent the values from the `field` parameter. The general syntax for a specifier is `%[flags][width][.precision]type`.

### Supported Types
* **Numbers**: Use `%d` or `%i` for integers, `%f` for floating-point decimals, and `%X` for hexadecimal.
* **Strings**: Use `%s` to include a string value.
* **Date/Time**: Use `%t` followed by a modifier to format timestamps (which must be in milliseconds since epoch). For example, `%tH` extracts the hour, and `%tF` formats the date as `YYYY-MM-DD`.
* **Booleans**: Use `%b` for `true`/`false`.

### Key Flags
* **`,`**: Adds thousands separators to numbers.
* **`.<number>`**: Specifies precision (e.g., number of decimal places for floats).
* **`0`**: Left-pads numbers with zeros instead of spaces.
* **`#`**: Adds a `0x` prefix to hexadecimal numbers.

***

## Examples

### Calculate the Mean of CPU Time and Format the Result

This example demonstrates a common analysis workflow: calculate an average, convert the units, and then use `format()` to make the final number readable.

* **Query Example**
    ```
    avg(field=cputimeNanos)
    | cputime := (_avg/1000000)
    | format("%,.2f", field=_avg, as=_avg)
    ```

* **Step-by-Step**
    1.  `avg()`: First, the query calculates the average of the `cputimeNanos` field.
    2.  The next step converts the average from nanoseconds to milliseconds to make it more readable.
    3.  `format()`: This is the final step. It takes the calculated average and formats it to two decimal places (`.2f`) with a thousands separator (`,`).

* **Result Event Data**
    The final output is a single, cleanly formatted field.
    ```
    _avg
    0.14
    ```

---

### Create a Frequency Count with Formatted, Clickable Links

This example shows how `format()` can be used to create structured text, such as markdown links, making aggregated results interactive.

* **Query Example**
    ```
    top(repo)
    | format("[Link](https://example.com/%s)", field=repo, as=link)
    ```

* **Input Event Data**
    A series of events with a `repo` field.

| @timestamp | repo |
| :--- | :--- |
| 2023-06-15T10:00:00Z | "frontend-app" |
| 2023-06-15T10:05:00Z | "backend-api" |
| 2023-06-15T10:10:00Z | "frontend-app" |
| ... | ... |

* **Step-by-Step**
    1.  `top(repo)`: This function first finds the most frequent values in the `repo` field and provides a count for each.
    2.  `format()`: The results are then piped to `format()`. For each repository name, it constructs a markdown link string, inserting the repository name (from the `repo` field, represented by `%s`) into the URL. The result is stored in a new field named `link`.

* **Result Event Data**
    The output is a table showing each repository, its event count, and a new `link` column containing a clickable markdown link.

| repo | _count | link |
| :--- | :--- | :--- |
| frontend-app | 4 | `[Link](https://example.com/frontend-app)` |
| backend-api | 2 | `[Link](https://example.com/backend-api)` |
| monitoring-tool | 1 | `[Link](https://example.com/monitoring-tool)`|
| database-service | 1 | `[Link](https://example.com/database-service)`|