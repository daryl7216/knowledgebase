# timeChart()

The `timeChart()` function is a powerful aggregation tool that groups events into time-based buckets and creates a time-series chart. It is an extension of `groupBy()` designed specifically for visualizing data over time.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **buckets** | number | optional | Defines the number of time buckets (1 to 1500) to divide the query interval into. |
| **function**| array of aggregate functions | optional <br> default: `count()` | The aggregate function(s) to perform on each time bucket. |
| **limit** | number | optional <br> default: 10 | The maximum number of series to produce (up to 500). Prioritizes series with the highest numerical aggregate values. |
| **minSpan** | string (Relative Time) | optional | Sets the minimum allowed time span for each bucket (e.g., `5h`). |
| **series** | string | optional | A field whose unique values will each become a separate series (line) on the chart. |
| **span** | string (Relative Time) | optional <br> default: `auto` | Defines a fixed time span for each bucket (e.g., `1h`, `15m`). |
| **timezone**| string | optional | The timezone to use for bucketing (e.g., `UTC`, `+02:00`). |
| **unit** | string | optional | Applies a unit conversion to the output (e.g., `bytes/bucket to Mbytes/hour`). |

***

## Function Operation

The `timeChart()` function is the primary tool for creating time-series visualizations. It groups events into time intervals (`buckets`) and then applies one or more aggregate functions to the events in each bucket.

> **Important**
> You cannot use anchored time units (like `/d` for the start of a day) when defining the `span` length in `timeChart()`.

### Series Selection
When using the `series` and `limit` parameters, `timeChart()` prioritizes which series to display based on the numerical values produced by the aggregate `function`. It does not consider the series names themselves. When multiple functions are used, it considers the combined numerical output to determine the top series.

***

## Examples

### Calculate a Percentage of Successful Status Codes

This advanced example demonstrates how to perform a multi-step calculation directly within the `function` parameter of `timeChart()` to produce a success rate percentage.

* **Query Example**
    ```
    success := if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
          [sum(success, as=success), count(as=total)]
          | pct_successful := (success / total) * 100
          | drop([success, total])
        }
      ], span=15m, limit=100)
    ```

* **Step-by-Step**
    1.  An `if()` statement first creates a binary `success` field: `1` for successful requests (status < 500) and `0` for server errors.
    2.  `timeChart()` then creates a series for each `customer`.
    3.  Within its embedded aggregation `{...}` for each time bucket:
        * It calculates the `sum()` of the `success` field (total successes) and the `count()` of all events (total requests).
        * It uses these two temporary values to calculate the `pct_successful`.
        * Finally, it uses `drop()` to remove the temporary `success` and `total` fields, leaving only the final percentage.

---

### Compute a Cumulative Aggregation Across Buckets

This example shows how `timeChart()` can be used to prepare data for a sequence function like `accumulate()`, allowing you to calculate a running total over time.

* **Query Example**
    ```
    timeChart(span=1000ms, function=sum(value))
    | accumulate(sum(_sum), as=_accumulated_sum)
    ```

* **Input Event Data**

| @timestamp | value |
| :--- | :--- |
| 1451606301001 | 5 |
| 1451606301500 | 6 |
| 1451606301701 | 1 |
| 1451606302001 | 2 |
| 1451606302201 | 6 |

* **Step-by-Step**
    1.  `timeChart(span=1000ms, function=sum(value))`: This function groups data into 1-second buckets and calculates the `sum()` of the `value` field for each bucket, creating a `_sum` field.
    2.  `accumulate(...)`: The `accumulate()` function then takes these time-bucketed results and calculates a running total of the `_sum` field across the buckets.

* **Result Event Data**
    The output is a time series that includes both the sum for each individual bucket (`_sum`) and the cumulative running total up to that point (`_accumulated_sum`).

| _bucket | _sum | _accumulated_sum |
| :--- | :-- | :--- |
| 1451606300000 | 0 | 0 |
| 1451606301000 | 12 | 12 |
| 1451606302000 | 8 | 20 |
| 1451606303000 | 0 | 20 |

---

### Find Top N Series Using Multiple Functions

This example illustrates how the `limit` parameter selects the top series based on the combined numerical output of multiple functions, not just the single highest value.

* **Query Example**
    ```
    timeChart(series=key, function=[max(value), {foo := value % 41 | selectLast(foo)}], limit=2)
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 42 |
| b | 41 |
| c | 40 |

* **Step-by-Step**
    1.  `timeChart()` creates a series for each `key` (`a`, `b`, `c`).
    2.  For each series, it calculates two values:
        * `max(value)`: The maximum value.
        * `foo := value % 41 | selectLast(foo)`: A second value based on a modulus calculation.
    3.  The top 2 series are selected based on the *sum* of these calculated values for each key:
        * **'a'**: `max` is 42, `foo` is `42 % 41 = 1`. Total score = 43.
        * **'b'**: `max` is 41, `foo` is `41 % 41 = 0`. Total score = 41.
        * **'c'**: `max` is 40, `foo` is `40 % 41 = 40`. Total score = 80.
    4.  Because `limit=2`, the function keeps the series with the highest total scores: 'c' (80) and 'a' (43).

* **Result Event Data**
    The output shows only the data for series 'a' and 'c', demonstrating how the combined metrics determined the "top" series.

| _bucket | key | _max | foo |
| :--- | :-: | :-- | :-: |
| 1747109790000 | a | 42 | 1 |
| 1747109790000 | c | 40 | 40 |