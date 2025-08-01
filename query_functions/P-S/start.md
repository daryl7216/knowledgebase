# start()

The `start()` function assigns a timestamp to an output field specified by the `as` parameter. This timestamp represents the beginning of the search time interval in milliseconds since January 1, 1970 (UTC). For live queries (where the search time interval is forever moving), `start()` equals the current time minus the search interval. For subqueries in `defineTable()` or joins, `start()` equals the start time of the subquery's search interval.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `start` | Name of output field. |

***

## Function Operation

The `start()` function retrieves and assigns the beginning timestamp of the current search interval to a specified output field. This timestamp is represented in milliseconds since January 1, 1970 (UTC).

The value returned by `start()` varies depending on the query context:

  * **Historical Queries**: It represents the exact start time of the time range chosen for the query.
  * **Live Queries**: Since the search interval is continuously moving, `start()` returns a dynamically updated timestamp that equals the current time minus the defined live search interval.
  * **Subqueries (e.g., within `defineTable()` or joins)**: It reflects the specific start time of that subquery's search interval, which might differ from the main query's interval.

It's important to note that the `start()` function is **not compatible with parser operations** because parsers do not operate with search intervals. The `as` parameter can be used to customize the name of the output field (defaulting to `start`).

***

## Examples

### Assign the value of `start()` to a field `s`

This is a basic example of assigning the search interval's start time to a new field.

  * **Query Example**
    ```
    s := start()
    ```

### Use `start()` in an assignment for a comparison

This example uses `start()` to calculate if an event's timestamp is "old" relative to the beginning of the search interval.

  * **Query Example**

    ```
    isold := (@timestamp - start()) < 1000
    ```

  * **Step-by-Step**

    1.  `(@timestamp - start())`: Calculates the difference between the event's `@timestamp` and the start time of the query interval.
    2.  `< 1000`: Checks if this difference is less than 1000 milliseconds (1 second).
    3.  `isold := ...`: Assigns a boolean result (`true` or `false`) to the field `isold`.

### Search Relative Time to Query Execution

This example demonstrates writing a query that filters events based on a time range relative to when the query is executed using the `start()` function.

  * **Query Example**

    ```
    test(@timestamp < (start() + (30 * 24 * 60 * 60 * 1000)))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `test(@timestamp < (start() + (30 * 24 * 60 * 60 * 1000)))`: Tests whether the `@timestamp` for an event is less than (earlier than) the query's start time plus 30 days.
          * `start()`: Returns the start time of the query.
          * `(30 * 24 * 60 * 60 * 1000)`: This calculation determines the number of milliseconds in 30 days:
              * 30 (days) \* 24 (hours) \* 60 (minutes) \* 60 (seconds) \* 1000 (milliseconds) = 2,592,000,000 milliseconds.
          * The condition checks if `@timestamp` is earlier than the query's start time plus 30 days.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to filter events that occurred within the first 30 days after the query's start time. This is a practical way of querying with a relative time from the query execution. The 30 days (and calculation) used in the example could be updated with any time calculation to achieve the required result.