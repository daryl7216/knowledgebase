# max()

The `max()` function finds the largest number for the specified field over a set of events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `max` | Name of output field. |
| `field` | string | required | Field to extract a number from. |
| `type` | string | optional | description |

***

## Function Operation

The `max()` function is an aggregate function used to determine the highest numerical value present in a specified field across a collection of events. When used, it returns a single event containing this maximum value in an output field. By default, this output field is named `max`, but you can assign a custom name using the `as` parameter. The `field` parameter, which can be omitted if the function is used in a context where the input field is clear, indicates which field's values should be evaluated.

***

## Examples

### Return what was the maximum responsetime

This is a basic example of using `max()` to find the highest response time.

  * **Query Example**
    ```
    max(responsetime)
    ```

### Filter for events in the repository with maximum responsetime values greater than 5 seconds

This example demonstrates how to filter events based on the maximum response time.

  * **Query Example**
    ```
    max(responsetime)
    _max > 5
    ```

### Alert Query for Parser Issues

This alert query tries to balance reacting when there are problems with parsers without being too restrictive.

  * **Query Example**

    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff:=_max - _min
    timeDiff > 300000 and count > 10
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=humio #kind=logs`: Filters on all logs across all hosts in the cluster.
    3.  `| loglevel=WARN`: Filters for all events where the `loglevel` is equal to `WARN`.
    4.  `| class = c.h.d.ParserLimitingJob`: Assigns the value `c.h.d.ParserLimitingJob` to the `class` for the logs having the `loglevel` value `WARN`.
    5.  `| "Setting reject ingest for"`: Filters for events containing the string "Setting reject ingest for". This is the error message generated when ingested events are rejected.
    6.  `groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])`: Groups the returned result by the field `id`, makes a count on the events, and returns the minimum timestamp and maximum timestamp. This returns a new event set, with the fields `id`, `_count`, `_min`, and `_max`.
    7.  `| timeDiff:=_max - _min`: Calculates the time difference between the maximum timestamp values (`_max`) and the minimum timestamp values (`_min`) and returns the result in a new field named `timeDiff`.
    8.  `| timeDiff > 300000 and count > 10`: Returns all events where the value of `timeDiff` is greater than 300000 and where there are more than 10 occurrences.
    9.  Event Result set.

  * **Summary and Results**
    This query is used to set up alerts for parser issues. Setting up alerts for parser issues will allow to proactively reach out to customers where their queries are being throttled and help them.

---

### Calculate Minimum and Maximum Response Times

This example calculates minimum and maximum response times using multiple aggregate functions in square brackets.

  * **Query Example**

    ```
    [min_response:= min(responsetime), max_response:= max(responsetime)]
    ```

  * **Input Event Data**

    | @timestamp | endpoint | responsetime | status\_code |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | /api/users | 145 | 200 |
    | 1686837826000 | /api/products | 892 | 200 |
    | 1686837827000 | /api/orders | 167 | 200 |
    | 1686837828000 | /api/payment | 1290 | 500 |
    | 1686837829000 | /api/users | 156 | 200 |
    | 1686837830000 | /api/items | 78 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `[min_response:= min(responsetime), max_response = max(responsetime)]`: In a single operation, calculates the minimum value from the `responsetime` field and returns the results in a field named `min_response`, and calculates the maximum value from the `responsetime` field and returns the results in a field named `max_response`. Square brackets allow multiple aggregations to be performed in a single operation.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the range of response times by calculating both the minimum and maximum values. The results are returned in fields with names specified in the field assignments. This query is useful, for example, to monitor service performance, identify outliers in response times, or establish performance baselines.

  * **Result Event Data**

    | min\_response | max\_response |
    | :--- | :--- |
    | 78 | 1290 |

    Note that only one row is returned containing both calculated values.

---

### Compute Aggregate Value for Each Array Element With Same Index

This example computes an aggregate value for each array element with the same index using the `array::reduceColumn()` function.

  * **Query Example**

    ```
    maxTimes := array::reduceColumn(times, var=x, function={time := max(x)})
    ```

  * **Input Event Data**

    | times\[0] | times\[1] | times\[2] |
    | :--- | :--- | :--- |
    | 1 | 2 | 3 |
    | 5 | 1 | 0 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `maxTimes := array::reduceColumn(times, var=x, function={time := max(x)})`: Computes the maximum time for each array element with the same index in the array and reduces it to one value.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the maximum time for each array element with the same index in a flat array.

  * **Result Event Data**

    | \_reduceColumn\[0] | \_reduceColumn\[1] | \_reduceColumn\[2] |
    | :--- | :--- | :--- |
    | 5 | 2 | 3 |

---

### Compute an Aggregated Value of an Array on All Events

This example computes an aggregated value of a flat array on all events using the `array::reduceAll()` function.

  * **Query Example**

    ```
    array::reduceAll("values[]", var=x, function=max(x))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `array::reduceAll("values[]", var=x, function=max(x))`: Computes the maximum value over all the values within the array `values[]` by using `max()` on each element, and then across each event in the event set.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to compute a value from all events and array elements of a specified array. The `reduce()` method is recommended when you need to have a single value returned from iterating over your array. Only aggregate functions that return a single event with a single field (such as `avg()`, `count()`, `sum()`, `max()` etc.) are allowed as the function argument.

---

### Create a Pivot Table

This example creates a view of LogScale activity by creating a pivot table.

  * **Query Example**

    ```
    groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})
    transpose(header=type)
    | drop(column)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})`: The first step to creating a pivot table is the base query that will create the initial summary of the information. In this fragment, a nested `groupBy()` aggregation is used. The embedded aggregation creates a group of the maximum access time for a given user, by using `max()` on the `@timestamp` against the `actor.user.id`. This creates a table of the last event by the user. The outer `groupBy()` then creates an aggregation of this maximum user time against the `type` which defines the operation performed. The result is a table of the last user and time for a specific operation.
    3.  `transpose(header=type)`: The `transpose()` function will convert individual columns into rows, switching the orientation. For example, the `type` column will now become the `type` row. The `header` parameter is used to specify `type` as the column header.
    4.  `drop(column)`: The table created contains the summarized information pivoted around the user ID and last event time further summarized by the type of the event. However, there is a `column` field in the event stream that was generated from the old row before the table was pivoted. That column can be removed by dropping the `column` field from the events using `drop()`.
    5.  Event Result set.

  * **Summary and Results**
    Pivoting an event set of data allows for the information to be displayed and summarized in a format that may make more logical sense as a display format. The query allows for creation of a pivot table showing the last user and time for a specific operation.

  * **Result Event Data (after first groupBy)**

    | type | actor.user.id | \_max |
    | :--- | :--- | :--- |
    | alert.clear-error | 007WGPBX9YbvZbK0rBMd5fgH | 1700546666592 |
    | alert.create | 007WGPBX9YbvZbK0rBMd5fgH | 1699004139419 |
    | alert.update | 007WGPBX9YbvZbK0rBMd5fgH | 1700546666676 |
    | dashboard.create | 007WGPBX9YbvZbK0rBMd5fgH | 1698417330709 |
    | dataspace.query | 007WGPBX9YbvZbK0rBMd5fgH | 1700721296197 |

  * **Result Event Data (after transpose)**

    | alert.clear-error | alert.create | alert.update |
    | :--- | :--- | :--- |
    | 1700546666592 | 1699004139419 | 1700546666676 |
    | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKO |

  * **Result Event Data (final after drop)**

    | alert.clear-error | alert.create | alert.update |
    | :--- | :--- | :--- |
    | 1700546666592 | 1699004139419 | 1700546666676 |
    | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd5fgH | 007WGPBX9YbvZbKOrBMd\! |

---

### Rounding Within a Timechart

This example rounds down a number in a field and displays information in a timechart using the `round()` and `timeChart()` functions.

  * **Query Example**

    ```
    timeChart(function={max(value) | round(_max, how=floor)})
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `timeChart(function={max(value) | round(_max, how=floor)})`: Creates a time chart using `max()` as the aggregate function for a field named `value` to find the highest value in each time bucket, and returns the result in a field named `_max`. It then rounds the implied field `_max` from the aggregate `max()` using the `floor` option to round down the value. For example, if original `_max` values were 10.8, 15.3, 20.7, after floor rounding they would be 10, 15, 20.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to round down maximum values over time to the nearest integer (whole value). This is useful when displaying information in a time chart. Rounding to the nearest integer will make it easier to distinguish the differences between values when used on a graph for time-based visualization. The query simplifies the data presentation. To round to a specific decimal accuracy, the `format()` function must be used.