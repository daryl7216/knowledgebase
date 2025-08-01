# now()

The `now()` function assigns the current time to the field provided by parameter `as`. The time is represented as milliseconds since January 1, 1970 (UTC). In historical queries, the current time is when the query is issued. In live queries, the current time is when `now()` is processed for each event. The value, therefore, depends on where in the query `now()` is placed. If it is placed before the first aggregate function, it is only evaluated the first time the query sees the event. If it is placed after the first aggregate function, it is evaluated continuously, and gives the live value of the current system time, which can divert between LogScale nodes.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `now` | Name of output field. |

***

## Function Operation

The `now()` function captures the current time and assigns it to a specified field. This time is expressed as milliseconds since January 1, 1970 (UTC). The behavior of `now()` differs between historical and live queries. In historical queries, it reflects the time the query was initiated. In live queries, it captures the time each event is processed by the `now()` function itself, meaning its value can vary depending on its position within the query pipeline and can even differ across LogScale nodes if placed after the first aggregate function. If `now()` is placed before the first aggregate function, it is only evaluated once per event.

***

## Examples

### Assign Current Time of Search Time Interval to Field

This example uses the `now()` function in a comparison to determine if an event is "old" (more than 1000 milliseconds old) by comparing the difference between the end time of the search interval and the event's `@timestamp` field.

  * **Query Example**

    ```
    $isold:=(now()-@timestamp)>1000$
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `isold:=(now()-@timestamp)>1000`: Returns results where the current time minus the ingestion timestamp are greater than 1000, and assigns the returned results to a new field named `isold`. More specifically, `now()` returns the end time of the search interval, `@timestamp` is the timestamp of the individual event. `(now()-@timestamp)` calculates the difference between these two times, giving the age of the event relative to the end of the search interval. This difference is compared to 1000 milliseconds (1 second). If the difference is greater than 1000 milliseconds, `isold` will be set to `true`; otherwise, it will be `false`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to quickly identify events that are more than 1 second older than the end of the search interval. Assigning the current time of the search time interval to a field is useful when, for example, searching for security events. When running a query, the events you are searching do not know the span of the search (the system just returns a list of the events in the given time interval), but you might want to show the relative time of the event timestamp compared to the search window. For example, if you search for a bunch of events that happened yesterday and you are searching from yesterday 00:00 to 23:59, you then want to calculate '3 hours before' or even '2s before' because when searching for security events that time difference may be important.

---

### Process Current Time in Live Queries

This example uses the `now()` function in live queries to process the current time for each event.

  * **Query Example**

    ```
    $curr:=now()$
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `$curr:=now()`: Processes current time for each event and returns the timestamps in a field named `curr` field. It records when events occur. The timestamp represents milliseconds.
    3.  Event Result set.

  * **Summary and Results**
    The query is used in live queries to process the current time for each event. Note that in live queries, this query returns timestamps based on the `now()` function's location:

      * Before the first aggregate function: Returns the initial event processing timestamp (it is only evaluated the first time the query sees the event).
      * After the first aggregate function: Returns continuously updated timestamps from each LogScale node (it gives the live value of the current system time, which can divert between LogScale nodes when `now()` is placed after the first aggregate function).
        The `now()` function and capturing current timestamps is useful in security contexts for incident response timing, threat detection timestamps, and security event correlation.