# min()

The `min()` function finds the smallest number for the specified field over a set of events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `min` | Name of output field. |
| `field` | string | required | Field to extract a number from. |
| `type` | string | optional | description |

***

## Function Operation

The `min()` function is an aggregate function used to determine the lowest numerical value present in a specified field across a collection of events. When used, it returns a single event containing this minimum value in an output field. By default, this output field is named `min`, but you can assign a custom name using the `as` parameter. The `field` parameter indicates which field's values should be evaluated.

***

## Examples

### Return what was the minimum responsetime

This is a basic example of using `min()` to find the lowest response time.

  * **Query Example**
    ```
    min(responsetime)
    ```

### Filter for events with a responsetime greater than 5 seconds

This example demonstrates how to filter events based on the minimum response time.

  * **Query Example**
    ```
    min(responsetime)
    _min > 5
    ```

### Alert Query for Parsers Issues

This alert query tries to balance reacting when there are problems with parsers, without being too restrictive.

  * **Query Example**

    ```
    #type=humio #kind=logs
    | loglevel=WARN
    | class = c.h.d.ParserLimitingJob
    | "Setting reject ingest for"
    groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])
    | timeDiff:=_max - _min
    | timeDiff > 300000 and count > 10
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=humio #kind=logs`: Filters on all logs across all hosts in the cluster.
    3.  `| loglevel=WARN`: Filters for all events where the `loglevel` is equal to `WARN`.
    4.  `| class = c.h.d.ParserLimitingJob`: Assigns the value `c.h.d.ParserLimitingJob` to the class for the logs having the `loglevel` value `WARN`.
    5.  `| "Setting reject ingest for"`: Filters for events containing the string "Setting reject ingest for". This is the error message generated when ingested events are rejected.
    6.  `groupBy(id, function=[count(), min(@timestamp), max(@timestamp)])`: Groups the returned result by the field `id`, makes a count on the events, and returns the minimum timestamp and maximum timestamp. This returns a new event set, with the fields `id`, `_count`, `_min`, and `_max`.
    7.  `| timeDiff:=_max - _min`: Calculates the time difference between the maximum timestamp values and the minimum timestamp values and returns the result in a new field named `timeDiff`.
    8.  `| timeDiff > 300000 and count > 10`: Returns all events where the values of `timeDiff` are greater than 300000 and where there are more than 10 occurrences.
    9.  Event Result set.

  * **Summary and Results**
    This query is used to set up alerts for parser issues. Setting up alerts for parser issues will allow proactively reaching out to customers where their queries are being throttled and help them.

### Calculate Minimum and Maximum Response Times

This example calculates minimum and maximum response times using multiple aggregate functions in square brackets.

  * **Query Example**

    ```
    [min_response:= min(responsetime), max_response := max(responsetime)]
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