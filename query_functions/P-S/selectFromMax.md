# selectFromMax()

The `selectFromMax()` function identifies the event with the maximum value in a specified field and returns selected fields from that event. The resulting event contains only the fields specified in the `include` parameter. If multiple events share the same maximum value, the `selectFromMax()` function returns one of those events randomly (non-deterministic way).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | required | The name of the field that is used to find the maximum value. |
| `include` | array of strings | required | The names of the fields to include in the generated event. |

***

## Function Operation

The `selectFromMax()` function is designed to efficiently find and return specific fields from the event that contains the maximum value of a designated field. It processes events to identify the single event (or one of them if there are ties) with the highest value in the `field` parameter. The function then constructs a new event containing only the fields specified in its `include` parameter.

This function is particularly efficient for finding the "latest" or "newest" value of a field when using a timestamp field (like `@timestamp`) as the basis for finding the maximum. It avoids the need for sorting all results or using other aggregation functions, as it directly selects the most recent matching event. Timestamps are typically stored as numerical values (often in Unix epoch format), where larger numbers represent more recent times.

If multiple events share the same maximum value for the specified `field`, `selectFromMax()` will return one of those events randomly, meaning the selection is non-deterministic.

***

## Examples

### Find Most Recent (Latest) Value of Field X

This example uses the `selectFromMax()` function to find the most recent (latest) value of a field `x` and return the timestamp when that value was recorded.

  * **Query Example**

    ```
    selectFromMax(@timestamp, include=[x, @timestamp])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selectFromMax(@timestamp, include=[x, @timestamp])`: Sorts all events by timestamp, then selects the event in field `x` with the highest (most recent) timestamp, returning only the specified fields `x` and `@timestamp`. In this example, `selectFromMax()` filters for the "maximum value" of `@timestamp`, and finds the event with the newest/latest timestamp in the event set that also contains the specified field `x`. The `include` parameter is used to specify which fields to include in the output.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the most recent value of field `x` by selecting the event with the highest (most recent) timestamp value. Using this query is an efficient way to find the latest value since it does not require sorting all results or using other aggregation functions - the query directly selects the most recent matching event.