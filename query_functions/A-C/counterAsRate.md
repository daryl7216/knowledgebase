# counterAsRate()

The `counterAsRate()` function calculates the rate of change for a counter field. The result is returned in a new field, with the unit being events per second by default. It's typically used within `timeChart()` or `groupBy()`.

> **Important**
> This function requires at least two data points to calculate a rate. It also expects the counter field to have values that are always increasing over time (monotonically increasing). If the counter resets (e.g., due to a server restart), no result is returned for the time bucket in which the reset occurred.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_rate` | The name of the output field. |
| **field** | string | required | The counter field from which to calculate the rate. The parameter name `field` can be omitted. |

***

## Example

### Show the Rate of a Counter Per Host

This example shows the rate of change for a counter, creating a separate time series for each host and converting the final unit from events per second to events per minute.

* **Query Example**
    ```
    timeChart(host, function=counterAsRate(counter), unit="1/sec to 1/min")
    ```

* **Summary**
    This query is used to monitor the rate of a counter on a per-host basis.
    - `timeChart(host, ...)`: This creates a time chart with a separate series for each unique value in the `host` field.
    - `function=counterAsRate(counter)`: For each host's series, it calculates the rate of change of the `counter` field.
    - `unit="1/sec to 1/min"`: This converts the default output rate from "per second" to "per minute" for easier interpretation.