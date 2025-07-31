# groupBy()

The `groupBy()` query function groups events together based on the unique values in one or more specified fields and then performs an aggregate function on each group. By default, it counts the events in each group.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | array of strings | required | The field or fields to group by. The parameter name `field` can be omitted. |
| **function**| array of aggregate functions | optional <br> default: `count(as=_count)` | The aggregate function(s) to apply to each group. To get only unique values without aggregation, use an empty list: `function=[]`. |
| **limit** | string or integer | optional <br> default: 20,000 | The maximum number of groups to create. Can be set to `max` to use the system's maximum limit (default 1,000,000). |

***

## Function Operation

The `groupBy()` function is a powerful tool for summarizing data. It creates a new event for each unique combination of values in the specified `field`(s).

### Limits and Memory
It's important to be aware of the limits. `groupBy()` operates entirely in memory and has a maximum number of groups it can handle, controlled by system settings (`GroupDefaultLimit` and `GroupMaxLimit`). If the number of unique groups exceeds this limit, the function may return incomplete results and a warning. Similarly, if the function exceeds its internal memory allocation, it will stop adding new groups, which can lead to inconsistent results.

### Series Selection
When a `limit` is applied, `groupBy()` prioritizes keeping the "top-N" groups. The "top" groups are determined by the highest numerical values produced by the aggregate functions across all fields, not by the names of the groups themselves.

***

## Examples

### Nested groupBy() for Sub-Totals

This advanced example shows a `groupBy()` nested within another `groupBy()` to produce both a total count for a primary field (`method`) and sub-counts for a secondary field (`statuscode`) within each primary group.

* **Query Example**
    ```
    [groupBy(method, function=[count(as=method_total), groupBy(statuscode, function=count(as=method_status_count))])]
    ```

* **Step-by-Step**
    1.  The outer `groupBy(method, ...)` groups all events by the HTTP method (e.g., GET, POST).
    2.  For each method, it performs two aggregations:
        * `count(as=method_total)`: Counts all events for that method to get a total.
        * `groupBy(statuscode, ...)`: A nested grouping then takes the events for that method and further groups them by `statuscode`, counting the events for each status.
    
* **Result Event Data**
    The result is a table that shows the total count for each method, repeated for each sub-grouped status code.

| method | method_total | statuscode | method_status_count |
| :--- | :--- | :--- | :--- |
| GET | 14078 | 200 | 7326 |
| GET | 14078 | 301 | 1246 |
| GET | 14078 | 404 | 5371 |
| HEAD | 139 | 200 | 14 |
| HEAD | 139 | 404 | 97 |

---

### Alert Query for Parser Issues

This example uses `groupBy()` in a real-world security monitoring scenario to create a high-fidelity alert for persistent parser errors.

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
    1.  The query first filters logs to find specific warnings about parsers rejecting events.
    2.  `groupBy(id, ...)` groups these errors by the parser `id`.
    3.  For each parser, it calculates three metrics: the total `count()` of errors, the `min()` timestamp of the first error, and the `max()` timestamp of the last error.
    4.  The query then calculates the `timeDiff` (duration of the errors) and filters the results to alert only when a parser has had more than 10 errors (`_count > 10`) over a period of more than 5 minutes (`timeDiff > 300000`).
* **Summary and Results**
    This query effectively uses `groupBy()` to aggregate statistics that allow for intelligent alerting, ignoring minor glitches and focusing on significant, ongoing problems.

---

### Detect Brute-Force Attacks using `partition()`

This example showcases using `groupBy()` to apply a complex event sequence analysis with the `partition()` function to each potential actor (identified by `key`).

* **Query Example**
    ```
    head()
    | groupBy(
        key,
        function=partition(
          condition=test(status=="success"),
          split="after",
          [
            { status="failure" | count(as=failures) },
            range(@timestamp, as=timespan),
            selectLast(status)
          ]
        )
      )
    | failures >= 3
    | status = "success"
    ```
* **Step-by-Step**
    1.  `head()` ensures events are sorted by time.
    2.  `groupBy(key, ...)` applies the following logic to each unique `key` (e.g., a user ID or IP address).
    3.  `partition()`: This function splits the sequence of events for a given key every time a "success" event occurs.
    4.  Within each partition (a sequence of failures followed by one success), it counts the number of failures, calculates the time span of the failures, and selects the final status.
    5.  The final two lines filter these partitions to find only those that represent a potential brute-force attack: 3 or more failures followed by a success.
* **Result Event Data**
    The output is a list of successful login events that were preceded by at least three failures.

| key | failures | timespan | status |
| :--- | :--- | :--- | :--- |
| a | 5 | 1600 | success |
| c | 3 | 3100 | success |

---

### Create a Pivot Table

This example uses a nested `groupBy()` to prepare data, which is then piped to `transpose()` to pivot the results, turning rows into columns for a summary view.

* **Query Example**
    ```
    groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})
    | transpose(header=type)
    | drop(column)
    ```

* **Step-by-Step**
    1.  The nested `groupBy` first finds the `max(@timestamp)` for each `actor.user.id`.
    2.  The outer `groupBy` then groups this result by `type` (the action performed). This creates a flat table of `type`, `actor.user.id`, and `max` timestamp.
    3.  `transpose(header=type)` pivots this table. It uses the values from the `type` column as the new column headers.
    4.  `drop(column)` removes a leftover metadata field from the transpose operation, cleaning up the final output.
* **Result Event Data**
    The final output is a pivot table where the rows might be user IDs and timestamps, and the columns are the different types of actions, showing the last time each user performed each action.

| alert.clear-error | alert.create | alert.update |
| :--- | :--- | :--- |
| 1700546666592 | 1699004139419 | 1700546666676 |
| 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH |