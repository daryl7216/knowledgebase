# accumulate()

The `accumulate()` function applies an aggregation function cumulatively to a sequence of events. It is useful for calculating running totals, running averages, or other cumulative metrics over time or across a series of events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **current** | enum | optional <br> default: `include` | Controls whether to include the current event in the accumulation. <br> **Values:** `exclude`, `include`. |
| **function** | array of aggregate functions | required | The aggregator function to accumulate (e.g., `sum()`, `avg()`, `count()`). It only accepts functions that output at most a single event. The parameter name `function` can be omitted. |

***

## Function Operation

The `accumulate()` function must be used after an aggregator function (like `head()`, `sort()`, `bucket()`, or `timeChart()`) to ensure event ordering, as it requires a specific order to calculate cumulative values correctly. Only functions that produce a single value per event (e.g., `sum()`, `avg()`, `count()`) can be used within the `accumulate()` function.

***

## Examples

### Calculate Running Average of Field Values

This example uses `accumulate()` with the `avg()` function to compute a running average.

* **Query Example**
    ```
    head()
    | accumulate(avg(value))
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| C | 1 |
| d | 2 |

* **Step-by-Step**
    1.  `head()`: Ensures events are ordered by time, selecting the oldest ones.
    2.  `accumulate(avg(value))`: Computes the running average of the `value` field for all events, including the current one.

* **Result Event Data**

| _avg | key | value |
| :-- | :-- | :-- |
| 5 | a | 5 |
| 5.5 | b | 6 |
| 4 | C | 1 |
| 3.5 | d | 2 |

---

### Compute Cumulative Aggregation Across Buckets

This example shows how to use `accumulate()` with `timeChart()` to aggregate values across time intervals.

* **Query Example**
    ```
    timeChart(span=1000ms, function=sum(value))
    | accumulate(sum(_sum), as=_accumulated_sum)
    ```

* **Input Event Data**

| key @timestamp | value |
| :--- | :-- |
| a 1451606301001 | 5 |
| b 1451606301500 | 6 |
| a 1451606301701 | 1 |
| c 1451606302001 | 2 |
| b 1451606302201 | 6 |

* **Step-by-Step**
    1.  `timeChart(span=1000ms, function=sum(value))`: Groups data into 1-second buckets and sums the `value` field for each bucket, creating a new field called `_sum`.
    2.  `accumulate(sum(_sum), as=_accumulated_sum)`: Calculates a running total of the values in the `_sum` field and stores the result in a new field named `_accumulated_sum`.

* **Result Event Data**

| _bucket | _sum | _accumulated_sum |
| :--- | :-- | :-- |
| 1451606300000 | 0 | 0 |
| 1451606301000 | 12 | 12 |
| 1451606302000 | 8 | 20 |
| 1451606303000 | 0 | 20 |

---

### Compute Cumulative Aggregation For a Specific Group

This example demonstrates using `accumulate()` within the `groupBy()` function to compute a cumulative total for a specific group.

* **Query Example**
    ```
    head()
    | groupBy(key, function = accumulate(sum(value)))
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| a | 1 |
| c | 2 |
| b | 6 |

* **Step-by-Step**
    1.  `head()`: Selects the oldest events, ordered by time.
    2.  `groupBy(key, function = accumulate(sum(value)))`: Groups data by the `key` field and accumulates the sum of the `value` field within each group.

* **Result Event Data**

| key | _sum | value |
| :-- | :-- | :-- |
| a | 5 | 5 |
| a | 6 | 1 |
| b | 6 | 6 |
| b | 12 | 6 |
| c | 2 | 2 |
