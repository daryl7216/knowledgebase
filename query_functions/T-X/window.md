# window()

The `window()` function computes an aggregate (like an average or sum) over a sliding window of data. It can only be used as the `function` argument within a `timeChart()` or `bucket()` function.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **buckets** | integer | optional | Defines the size of the sliding window in terms of the number of `timeChart` buckets to include in the aggregate. |
| **function**| array of aggregate functions | optional <br> default: `count()` | The aggregate function(s) to perform on each window. |
| **span** | string (Relative Time) | optional | Defines the width of the sliding window as a fixed time duration (e.g., `15m`, `1h`). |

**Note**: You must define exactly one of either `span` or `buckets`.

***

## Function Operation

The `window()` function calculates a running aggregate. For each time bucket created by the parent `timeChart()` function, `window()` looks back over a specified period (defined by `span` or `buckets`) and computes an aggregate value from all the events in that window. This is useful for smoothing out data or calculating moving averages. If the query's overall time interval is less than the specified window `span`, no result will be computed.

***

## Examples

### Find Outliers Using a Dynamic Threshold

This complex example uses `window()` to calculate a running average and standard deviation, which are then used to define a dynamic threshold for identifying outliers.

* **Query Example**
    ```
    timeChart(function=[max(m1), window(function=[stddev(m1), avg(m1)], span=15m)])
    | groupBy(_bucket, function={ limit := _avg + 2 * _stddev | table([_max, limit]) })
    ```

* **Step-by-Step**
    1.  `timeChart()`: The query first creates a time chart that calculates two things for each time bucket:
        * `max(m1)`: The simple maximum value of the `m1` metric in that bucket.
        * `window(...)`: A 15-minute sliding window that calculates the running `avg(m1)` and `stddev(m1)` (standard deviation).
    2.  `groupBy(_bucket, ...)`: The results are then re-grouped by the time bucket.
    3.  `limit := _avg + 2 * _stddev`: Inside the `groupBy`, a new field `limit` is calculated. This creates a dynamic threshold that is two standard deviations above the running average.
    4.  `table([_max, limit])`: The final output is a table comparing the actual maximum value (`_max`) in a bucket to the calculated dynamic `limit`, making it easy to spot outliers.

---

### Calculate Multiple Running Aggregates per Series

This example demonstrates how to calculate multiple running aggregates (average and maximum) for different series (hosts) simultaneously.

* **Query Example**
    ```
    timeChart(host, function=window(function=[avg(cpu_load), max(cpu_load)], buckets=3))
    ```

* **Step-by-Step**
    1.  `timeChart(host, ...)`: The query creates a time chart with a separate series for each unique `host`.
    2.  `function=window(...)`: For each time bucket, the `window()` function is applied.
    3.  `buckets=3`: The window size is defined as the current bucket plus the two preceding buckets.
    4.  `function=[avg(cpu_load), max(cpu_load)]`: Within this 3-bucket window, it calculates both the `avg()` and the `max()` of the `cpu_load` field.

* **Summary and Results**
    This query produces a time chart showing the 3-bucket moving average and the 3-bucket moving maximum CPU load for each host. This is useful for monitoring performance, as it can smooth out temporary spikes and highlight sustained high CPU usage.