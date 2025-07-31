# formatDuration()

The `formatDuration()` function converts a numerical duration into a human-readable string (e.g., `6h30m52s`).

> **Important**
> The value of the field being formatted must be an integer. If the input value is a floating-point number, it will not be converted. [cite_start]You can use the `round()` or `format()` functions to convert it to an integer first. [cite: 3801, 3802, 3803]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: input field name | The name of the output field. |
| **field** | string | required | The name of the input field containing the duration. Must be an integer. [cite_start]The parameter name `field` can be omitted. [cite: 3792] |
| **from** | string | optional <br> default: `ms` | The time unit of the input field. [cite_start]Possible values are: `d` (days), `h` (hours), `m` (minutes), `s` (seconds), `ms` (milliseconds), `ns` (nanoseconds), `us` (microseconds). [cite: 3792] |
| **precision**| number | optional <br> default: 0 | The number of time units to include in the output string. [cite_start]A value of 0 includes all units. [cite: 3792] |

***

## Example

### Format a Duration with Limited Precision

This example demonstrates how to format a duration from a field, limiting the output to the three most significant time units to improve readability.

* **Query Example**
    ```
    formatDuration(duration, precision=3)
    ```

* **Step-by-Step**
    1.  The query starts with events that have a field named `duration` containing an integer value in milliseconds. [cite_start]For example, `23452553`. [cite: 3821, 3822]
    2.  [cite_start]`formatDuration()` reads the integer from the `duration` field. [cite: 3820]
    3.  [cite_start]The `precision=3` parameter instructs the function to display the result using only the three largest time units. [cite: 3823, 3824] [cite_start]For the value `23452553` ms (which is 6 hours, 30 minutes, 52 seconds, and 553 milliseconds), a precision of 3 would likely result in `6h30m52s`. [cite: 3822, 3824]

* **Summary and Results**
    This query is used to improve the readability of log entries that contain duration fields. By limiting the precision, you can present the most significant parts of the time duration, making it easier to understand at a glance. [cite_start]For instance, `6h30m` might be shown if `precision=2` was used. [cite: 3825, 3828]