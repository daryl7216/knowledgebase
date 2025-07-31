# tail()

The `tail()` function retrieves a specified number of the most recent events. It sorts events by `@timestamp` or `@ingesttimestamp` to determine which are the newest.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **limit** | number | optional <br> default: 200 | The maximum number of events to return. The parameter name `limit` can be omitted. |

***

## Function Operation

The `tail()` function sorts events in descending order based on their timestamp to find the most recent ones. It defaults to using the `@timestamp` field but will use `@ingesttimestamp` if `@timestamp` is not available. If neither timestamp field is present, the query will report an error.

***

## Example

### Deduplicate Events by a Specific Field

This example uses `tail()` inside a `groupBy()` function to effectively deduplicate events based on a field. For each unique value in the specified field, it keeps only the single most recent event.

* **Query Example**
    ```
    groupBy(field, function=tail(1))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  `groupBy(field, ...)`: This function groups all events by the unique values found in the specified `field`.
    3.  `function=tail(1)`: Within each of those groups, `tail(1)` is executed. This sorts the events in that group by timestamp and keeps only the single most recent one.

* **Summary and Results**
    This query is a powerful technique for creating a unique list of events based on a particular field, where "unique" is defined as the last seen event for that field's value. This is useful for getting the current state of entities when you have many historical events for each one.