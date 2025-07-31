# duration()

The `duration()` function computes the number of milliseconds in a given time period. It makes timestamp comparisons easier to read and less error-prone by allowing the use of human-readable time syntax (e.g., `5m`, `2d`).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_duration` | The name of the output field that will store the millisecond value. |
| **duration**| string | required | The time duration specification, defined using Relative Time Syntax. The parameter name `duration` can be omitted. |

***

## Function Operation

This function takes a string representing a time duration (like `"5m"` for five minutes) and returns the equivalent number of milliseconds. This numeric value can then be used in calculations and comparisons, which is especially useful for creating dynamic dashboard parameters.

***

## Example

### Narrow the Search Interval

This example demonstrates how to use `duration()` to filter events, keeping only those that have occurred within the last two days.

* **Query Example**
    ```
    test(@timestamp > now() - duration("2d"))
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  The `duration("2d")` function is called, which calculates the total number of milliseconds in two days.
    3.  This millisecond value is subtracted from the current time (`now()`) to establish a cutoff timestamp for two days ago.
    4.  The `test()` function then compares each event's `@timestamp` to this cutoff, keeping only the events that are more recent.

* **Summary and Results**
    This query effectively filters the search results to a rolling two-day window. Using `duration()` makes the query more readable and maintainable than hard-coding the millisecond value for two days (`172800000`). It is a convenient way to define relative time windows in searches and dashboards.