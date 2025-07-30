# accumulate()

[cite_start]The `accumulate()` function applies an aggregation function cumulatively to a sequence of events[cite: 3]. [cite_start]It is useful for calculating running totals, running averages, or other cumulative metrics over time or across a series of events[cite: 3, 4].

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| [cite_start]**current** | enum | optional <br> default: `include` [cite: 6, 7] | [cite_start]Controls whether to include the current event in the accumulation[cite: 6]. [cite_start]<br> **Values:** `exclude`, `include`[cite: 6]. |
| [cite_start]**function** | array of aggregate functions | required [cite: 6] | [cite_start]The aggregator function to accumulate (e.g., `sum()`, `avg()`, `count()`)[cite: 6]. [cite_start]It only accepts functions that output at most a single event[cite: 6]. [cite_start]The parameter name `function` can be omitted[cite: 8]. |

***

## Function Operation

[cite_start]The `accumulate()` function must be used after an aggregator function (like `head()`, `sort()`, `bucket()`, or `timeChart()`) to ensure event ordering, as it requires a specific order to calculate cumulative values correctly[cite: 12, 13, 14]. [cite_start]Only functions that produce a single value per event (e.g., `sum()`, `avg()`, `count()`) can be used within the `accumulate()` function[cite: 15].

***

## Examples

### Calculate Running Average of Field Values

[cite_start]This example uses `accumulate()` with the `avg()` function to compute a running average[cite: 18, 19].

* [cite_start]**Query Example** [cite: 20]
    ```
    head()
    | accumulate(avg(value))
    ```

* [cite_start]**Input Event Data** [cite: 29, 30]

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| C | 1 |
| d | 2 |

* **Step-by-Step**
    1.  [cite_start]`head()`: Ensures events are ordered by time, selecting the oldest ones[cite: 36].
    2.  [cite_start]`accumulate(avg(value))`: Computes the running average of the `value` field for all events, including the current one[cite: 39].

* [cite_start]**Result Event Data** [cite: 45, 47]

| _avg | key | value |
| :-- | :-- | :-- |
| 5 | a | 5 |
| 5.5 | b | 6 |
| 4 | C | 1 |
| 3.5 | d | 2 |

---

### Compute Cumulative Aggregation Across Buckets

[cite_start]This example shows how to use `accumulate()` with `timeChart()` to aggregate values across time intervals[cite: 48, 49].

* [cite_start]**Query Example** [cite: 50, 52]
    ```
    timeChart(span=1000ms, function=sum(value))
    | accumulate(sum(_sum), as=_accumulated_sum)
    ```

* [cite_start]**Input Event Data** [cite: 57, 58]

| key @timestamp | value |
| :--- | :-- |
| a 1451606301001 | 5 |
| b 1451606301500 | 6 |
| a 1451606301701 | 1 |
| c 1451606302001 | 2 |
| b 1451606302201 | 6 |

* **Step-by-Step**
    1.  [cite_start]`timeChart(span=1000ms, function=sum(value))`: Groups data into 1-second buckets and sums the `value` field for each bucket, creating a new field called `_sum`[cite: 63].
    2.  [cite_start]`accumulate(sum(_sum), as=_accumulated_sum)`: Calculates a running total of the values in the `_sum` field and stores the result in a new field named `_accumulated_sum`[cite: 67].

* [cite_start]**Result Event Data** [cite: 73, 74]

| _bucket | _sum | _accumulated_sum |
| :--- | :-- | :-- |
| 1451606300000 | 0 | 0 |
| 1451606301000 | 12 | 12 |
| 1451606302000 | 8 | 20 |
| 1451606303000 | 0 | 20 |

---

### Compute Cumulative Aggregation For a Specific Group

[cite_start]This example demonstrates using `accumulate()` within the `groupBy()` function to compute a cumulative total for a specific group[cite: 92, 93, 99].

* [cite_start]**Query Example** [cite: 94, 97]
    ```
    head()
    | groupBy(key, function = accumulate(sum(value)))
    ```

* [cite_start]**Input Event Data** [cite: 102, 103]

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| a | 1 |
| c | 2 |
| b | 6 |

* **Step-by-Step**
    1.  [cite_start]`head()`: Selects the oldest events, ordered by time[cite: 108].
    2.  [cite_start]`groupBy(key, function = accumulate(sum(value)))`: Groups data by the `key` field and accumulates the sum of the `value` field within each group[cite: 112].

* [cite_start]**Result Event Data** [cite: 119, 120]

| key | _sum | value |
| :-- | :-- | :-- |
| a | 5 | 5 |
| a | 6 | 1 |
| b | 6 | 6 |
| b | 12 | 6 |
| c | 2 | 2 |
