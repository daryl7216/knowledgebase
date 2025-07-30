# avg()

The `avg()` function calculates the average for a numeric field over a set of events. The result is returned in a field named `_avg` by default.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_avg` | The name for the output field that will contain the calculated average. |
| **field** | string | required | The field from which to extract numbers to be averaged. The parameter name `field` can be omitted. |

***

## Example

### Calculate the Mean of CPU Time

This example calculates the average CPU time from a field, converts it from nanoseconds to milliseconds, and formats the final result to two decimal places. This is useful for determining processing power or troubleshooting high CPU usage.

* **Query Example**
    ```
    avg(field=cputimeNanos)
    | cputime := (_avg/1000000)
    | format("%,.2f", field=_avg, as=_avg)
    ```

* **Step-by-Step**
    1.  `avg(field=cputimeNanos)`: First, the query calculates the average of the `cputimeNanos` field across all events.
    2.  `cputime := (_avg/1000000)`: The result, stored in `_avg`, is then divided by 1,000,000 to convert the time from nanoseconds to milliseconds, making it easier to read.
    3.  `format("%,.2f", field=_avg, as=_avg)`: Finally, the `format()` function is used to reformat the calculated average to show only two decimal points.

* **Result Event Data**
    The final output is a single field `_avg` containing the formatted average CPU time in milliseconds.
    ```
    _avg
    0.14
    ```