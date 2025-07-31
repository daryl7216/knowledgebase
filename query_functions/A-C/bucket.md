# bucket()

The `bucket()` function extends `groupBy()` to group events by time, dividing the search time interval into a specified number of time-based "buckets." Each event is placed into a bucket based on its timestamp, and the function creates a new `_bucket` field containing the start time of that bucket in milliseconds.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **buckets** | number | optional | Defines the number of buckets to divide the query time interval into (1 to 1500). |
| **field** | string | optional | Specifies one or more fields to group by in addition to the time bucket. |
| **function**| array of aggregate functions | optional <br> default: `count(as=_count)` | Specifies the aggregate function(s) to perform on each group. |
| **limit** | integer | optional <br> default: 10 | The maximum number of series to produce (up to 500), prioritizing the series with the highest numerical values. |
| **minSpan** | long | optional | Sets the minimum allowed time span for each bucket, using Relative Time Syntax (e.g., `1hour`). |
| **span** | relative-time | optional <br> default: auto | Defines a fixed time span for each bucket (e.g., `1min`, `3weeks`). |
| **timezone**| string | optional | Defines the time zone for bucketing, such as `UTC` or `'+02:00'`. |
| **unit** | array of strings | optional | Applies a unit conversion to the output columns (e.g., `bytes/span` to `Kbytes/day`). |

***

## Function Operation

The `bucket()` function groups data into time-based intervals. It's important to note that you cannot use anchored time units (like `/d` for start of day) for the `span` parameter. The function calculates buckets by dividing the query time interval. Due to how time boundaries are handled, this can sometimes result in more buckets being created than specified. For example, a one-hour query with a one-minute span might produce 61 buckets to include partial data at the beginning and end of the time range.

***

## Examples

### Calculate Linear Relationship Between Two Variables

This example uses `bucket()` to prepare data for a linear regression analysis, calculating the relationship between the total size of data sent and the average server load over time.

* **Query Example**
    ```
    bucket(function=[sum(bytes_sent, as=x), avg(server_load_pct, as=y)])
    | linReg(x=x, y=y)
    ```

* **Input Event Data**
    A series of events with `bytes_sent` and `server_load_pct` values.

| @timestamp | bytes_sent | server_load_pct |
| :--- | :--- | :--- |
| 2024-01-15T09:00:00Z | 156780 | 45.2 |
| 2024-01-15T09:05:00Z | 234567 | 52.8 |
| 2024-01-15T09:10:00Z | 189234 | 48.6 |
| 2024-01-15T09:15:00Z | 345678 | 65.3 |
| ... | ... | ... |

* **Step-by-Step**
    1.  `bucket()`: This function first groups the data into time buckets. Within each bucket, it calculates two things:
        * The `sum()` of `bytes_sent`, renaming it to `x`.
        * The `avg()` of `server_load_pct`, renaming it to `y`.
    2.  `linReg()`: The results are then piped to the linear regression function, which correlates the sum of bytes sent (`x`) with the average server load (`y`).

* **Result Event Data**
    The output shows the statistical relationship between the two variables.

| _slope | _intercept | _r2 | _n |
| :--- | :--- | :--- | :--|
| 0.000106... | 28.934... | 0.991... | 10 |

---

### Calculate Linear Relationship Grouped by a Field

This more advanced example analyzes how different HTTP request types impact server load by performing a separate linear regression for each type.

* **Query Example**
    ```
    bucket(function=[avg(server_load_pct, as=y), groupBy(request_type, function=count(as=x))])
    | groupBy(request_type, function=linReg(x=x, y=y))
    ```

* **Input Event Data**
    Events containing server load and request type information.

| @timestamp | server_load_pct | request_type |
| :--- | :--- | :--- |
| 2024-01-15T09:05:00.000Z | 52.8 | GET |
| 2024-01-15T09:05:00.000Z | 52.8 | PUT |
| 2024-01-15T09:10:00.000Z | 48.6 | GET |
| ... | ... | ... |

* **Step-by-Step**
    1.  `bucket()`: Within each time bucket, the function calculates the average `server_load_pct` (as `y`) and also performs a nested `groupBy()` to count the number of requests for each `request_type` (as `x`).
    2.  `groupBy()`: The results are then grouped again by `request_type`.
    3.  `linReg()`: For each request type group, a linear regression is performed to correlate the request count (`x`) with the average server load (`y`).

* **Result Event Data**
    The output provides separate linear regression results for each request type, helping to identify which types have the strongest impact on server performance.

| request_type | _slope | _intercept | _r2 |
| :--- | :--- | :--- | :--- |
| DELETE | <no value> | <no value> | <no value> |
| GET | -13.749... | 72.799... | 0.594... |
| POST | 16.299... | 32.700... | 0.719... |
| PUT | <no value> | <no value> | <no value> |