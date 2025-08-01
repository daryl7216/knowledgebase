# selectFromMin()

The `selectFromMin()` function identifies the event with the minimum value in a specified field and returns selected fields from that event. The resulting event contains only the fields specified in the `include` parameter. If multiple events share the same minimum value, the `selectFromMin()` function returns one of those events randomly (non-deterministic way).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | required | The name of the field that is used to find the minimum value. |
| `include` | array of strings | required | The names of the fields to include in the generated event. |

***

## Function Operation

The `selectFromMin()` function efficiently identifies and extracts specific fields from the event that possesses the minimum value of a designated field. It processes events to find the single event (or one of them if there are multiple events with the same minimum value) that has the lowest value in the `field` parameter. A new event is then constructed containing only the fields specified in the `include` parameter.

This function is particularly efficient for retrieving the "oldest" or "first" value of a field, especially when the `@timestamp` field is used as the basis for finding the minimum. It eliminates the need for sorting all results or using other aggregation functions, as it directly selects the event with the earliest timestamp that also contains the specified field. Timestamps are typically stored as numerical values (often in Unix epoch format), where lower numbers indicate older times.

If multiple events share the same minimum value for the specified `field`, `selectFromMin()` will return one of those events randomly, making its selection non-deterministic in such cases.

***

## Examples

### Find Oldest (First) Value of Field X

This example uses the `selectFromMin()` function to find the oldest (first) value of a field `x` and return the timestamp when that value was recorded.

  * **Query Example**

    ```
    selectFromMin(@timestamp, include=[x, @timestamp])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selectFromMin(@timestamp, include=[x, @timestamp])`: Sorts all events by timestamp, then selects the event in field `x` with the oldest (lowest) timestamp, returning only the specified fields `x` and `@timestamp`. In this example, `selectFromMin()` filters for the "minimum value" of `@timestamp`, and finds the event with the oldest/first timestamp in the event set that also contains the specified field `x`. Timestamps are typically stored as numerical values (often in Unix epoch format), where lower numbers represent older times. The `include` parameter is used to specify which fields to include in the output.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the oldest value of field `x` by selecting the event with the lowest (oldest) timestamp value. Using this query is an efficient way to find the first value since it does not require sorting all results or using other aggregation functions - the query directly selects the oldest matching event.