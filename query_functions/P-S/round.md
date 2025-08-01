# round()

The `round()` function rounds a numeric input field to the nearest integer, with an optional method to set the rounding type.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: Same name as input field | The output name of the field to round. |
| `field` | string | required | The name of the field to round. |
| `how` | string | optional \<br\> default: `round` | Method used to round the number. \<br\>**Values:**\<br\>`ceil`: Round up to the nearest whole number. \<br\>`floor`: Round down to the nearest whole number. \<br\>`round`: Standard rules; for example, \<0.5 rounds down, \>0.5 rounds up. |

***

## Function Operation

The `round()` function is used to convert a numerical value in a specified field to the nearest integer. It offers three methods for rounding, controlled by the `how` parameter:

  * **`round` (default)**: Applies standard mathematical rounding rules. Values with a fractional part of 0.5 or greater are rounded up, while those less than 0.5 are rounded down.
  * **`ceil`**: Always rounds the number up to the smallest integer that is greater than or equal to the original value.
  * **`floor`**: Always rounds the number down to the largest integer that is less than or equal to the original value.

The result of the rounding operation is outputted to a new field. By default, this output field retains the same name as the input field. However, a custom name can be assigned using the `as` parameter. This function simplifies numbers, making them easier to work with, especially for display or comparison purposes. To round to a specific decimal accuracy (rather than a whole number), the `format()` function should be used instead.

***

## Examples

### Basic Rounding

This example uses the `round()` function to round a number to the nearest integer using standard math rules.

  * **Query Example**

    ```
    round(myvalue)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `round(myvalue)`: Rounds the number in `myvalue` using standard rounding rules (numbers greater than 0.5 are rounded up, numbers lower than 0.5 are rounded down).
    3.  Event Result set.

  * **Summary and Results**
    The query is used to round a floating-point number to the nearest integer. Rounding is used to simplify numbers, making them easier to work with. To format a number or round to a specific decimal accuracy, use `format()` (e.g., `format("%.2f", field=value)`).

---

### Rounding Within a Timechart

This example uses the `round()` function with the `floor` parameter to round down a field value to an integer and display information within a timechart.

  * **Query Example**

    ```
    timeChart(function={max(value) | round(_max, how=floor)})
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `timeChart(function={max(value) | round(_max, how=floor)})`: Creates a time chart using `max()` as the aggregate function for a field named `value` to find the highest value in each time bucket, and returns the result in a field named `_max`. It then rounds the implied field `_max` from the aggregate `max()` using the `floor` option to round down the value. For example, if original `_max` values were 10.8, 15.3, 20.7, after floor rounding they would be 10, 15, 20.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to round down maximum values over time to the nearest integer (whole value). This is useful when displaying information in a time chart. Rounding to the nearest integer will make it easier to distinguish the differences between values when used on a graph for time-based visualization. The query simplifies the data presentation. To round to a specific decimal accuracy, the `format()` function must be used.

---

### Rounding to n Decimal Places

This example (from the document but explicitly stating `format()` is used instead of `round()`) shows how to round a number to a specific number of decimal places.

  * **Query Example**

    ```
    format("%.2f", field=value)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `format("%.2f", field=value)`: Rounds the `field` value to two decimal places.
    3.  Event Result set.

  * **Summary and Results**
    When using `format()`, rounding is performed using standard math rules. The `format()` function rounds a number to a specific decimal accuracy. To round a number to the nearest integer, `round()` should be used.