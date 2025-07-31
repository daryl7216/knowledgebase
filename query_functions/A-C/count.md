# count()

The `count()` function counts the number of events. It can be used to count all events, only events containing a specific field, or the number of distinct values for a specified field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_count` | The name of the output field. |
| **distinct** | boolean | optional | If `true`, counts only the distinct values of the specified `field`. |
| **field** | string | optional | If specified, the function only counts events that contain this field. |

***

## Function Operation

The `count()` function provides a simple event count. When using `distinct=true`, the function counts the number of unique values for a given `field`. For performance reasons, when there are many distinct values, the system uses an estimation algorithm. This means the result will be a very close approximation but may not be perfectly exact. The typical accuracy is less than 2%, and for fewer than 100 distinct values, the count is exact.

***

## Examples

### Alert Query for Parser Issues

This complex example uses `count()` as part of a multi-stage query to create an alert for ongoing parser problems. The goal is to trigger an alert only when a parser has been generating a high volume of errors over a significant period.

* **Query Example**
    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    | groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff := _max - _min
    | timeDiff > 300000 and _count > 10
    ```

* **Step-by-Step**
    1.  The query first filters for specific warning logs related to parsers rejecting ingested events.
    2.  `groupBy(id, ...)` groups these errors by the parser `id`. For each parser, it performs three aggregations:
        * `count()`: Counts the total number of error events (`_count`).
        * `min(@timestamp)`: Finds the time of the first error (`_min`).
        * `max(@timestamp)`: Finds the time of the last error (`_max`).
    3.  `timeDiff := _max - _min` calculates the duration of the error period for each parser.
    4.  The final line filters these results, creating an alert only for parsers where the errors have persisted for more than 5 minutes (`timeDiff > 300000` ms) AND there have been more than 10 error events (`_count > 10`).

* **Summary and Results**
    This query is used to proactively identify and alert on significant parser issues. By using `count()` in combination with time analysis, it avoids generating alerts for minor, transient problems and focuses only on persistent, high-volume failures.

---

### Calculate a Percentage of Successful Status Codes Over Time

This example shows `count()` being used inside an embedded aggregation within `timeChart()` to calculate a success rate percentage over time.

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
    1.  The query first creates a temporary field `success`, assigning `1` for successful HTTP status codes (less than 500) and `0` for server errors (500 or greater).
    2.  `timeChart()` creates a time series chart.
    3.  Inside the `function` parameter, an embedded aggregation `{...}` is used:
        * `sum(success, as=success)` adds up all the `1`s, giving a total of successful requests.
        * `count(as=total)` counts all requests, regardless of success.
    4.  These two values are then used to calculate `pct_successful`.
    5.  The temporary `success` and `total` fields are dropped, leaving only the final percentage for the time chart.

* **Summary and Results**
    This query generates a time chart showing the percentage of successful requests for each customer over 15-minute intervals. It demonstrates how `count()` can serve as the denominator (the total number of events) in a rate or percentage calculation within a more complex aggregation.