# test()

The `test()` function evaluates an arbitrary expression as a boolean value (`true` or `false`) and filters events, keeping only those for which the expression returns `true`. It allows for complex comparisons between multiple fields and values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **expression** | expression | required | The expression to evaluate. The parameter name `expression` can be omitted. |

***

## Function Operation

This function provides a flexible way to filter events based on custom logic. It's important to remember that within a `test()` expression, unquoted strings are interpreted as field names, while quoted strings are treated as literal values. For example, `test(myField == "myValue")` compares the field `myField` to the string "myValue", whereas `test(myField == myOtherField)` compares the value of the `myField` field to the value of the `myOtherField` field.

***

## Examples

### Check if Fields Have the Same Length

This example uses `test()` with the `length()` function nested inside it to find events where the values in two different fields have the same character length.

* **Query Example**
    ```
    test(length(userid) == length(method))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  For each event, the `test()` function evaluates the expression `length(userid) == length(method)`.
    3.  It calculates the length of the `userid` field and the length of the `method` field.
    4.  It then compares the two lengths. If they are equal, the expression returns `true`, and the event is kept in the result set.

* **Summary and Results**
    This query returns all events where the `userid` and `method` fields have the same length (e.g., a `userid` of "chad" and a `method` of "POST" would both have a length of 4 and would match).

---

### Filter for Events Within a Relative Time Window

This example demonstrates a more complex use case, filtering for events that occurred within the first 30 days of the query's start time.

* **Query Example**
    ```
    test(@timestamp < (start() + (30*24*60*60*1000)))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  The `test()` function evaluates the time comparison for each event.
    3.  `start()`: This function returns the start time of the query search window in milliseconds.
    4.  `(30*24*60*60*1000)`: This is a manual calculation of the number of milliseconds in 30 days.
    5.  The expression `(start() + 30_days_in_ms)` calculates a cutoff timestamp that is exactly 30 days after the query began.
    6.  The event's `@timestamp` is then compared to this cutoff. If the event's timestamp is earlier than the cutoff, the expression returns `true`, and the event is kept.

* **Summary and Results**
    This query is a practical way to filter for events that occurred within a specific time window relative to the start of the search, allowing for dynamic and precise time-based analysis.