# callFunction()

The `callFunction()` function dynamically calls a specified aggregate function on a field over a set of events. This is particularly useful for creating dynamic dashboards where the aggregation function can be changed via a parameter.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_function` | The name of the output field. |
| **field** | string | required | The field to perform the function on. |
| **function**| string | required | The name of the function to run. The parameter name `function` can be omitted. Possible values are: `avg`, `count`, `max`, `min`, `range`, `sum`. |

***

## Example

### Call a Named Function within a Time Chart

This example demonstrates how to use `callFunction()` inside a `timeChart()` to dynamically select the aggregate function using a query parameter. This allows users to change the function (e.g., from `count` to `avg`) directly from a dashboard widget.

* **Query Example**
    ```
    timeChart(function=[callFunction(?{function=count}, field=value)])
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  The `timeChart()` function is used to plot data over time.
    3.  `callFunction()` is used as the aggregation function for the time chart. It reads the function name from a query parameter called `function`. If no parameter is provided, it defaults to `count`.
    4.  The chosen function is then applied to the `value` field for each time bucket in the chart.

* **Summary and Results**
    The query counts events in the `value` field and displays the results in a time chart. The key feature is that the aggregation function itself is dynamic. By changing the `?function` parameter in a dashboard (e.g., to `?function=avg`), the same widget can be used to display the average, count, sum, etc., without altering the underlying query.