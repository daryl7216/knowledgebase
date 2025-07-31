# end()

[cite_start]The `end()` function assigns a timestamp to an output field, representing the end of the search time interval in milliseconds since the UTC epoch. [cite: 646, 647]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_end` | [cite_start]The name of the output field. [cite: 654] [cite_start]The parameter name can be omitted. [cite: 655] |

***

## Function Operation

[cite_start]This function provides the timestamp for the end of the current search window. [cite: 647] [cite_start]For **live queries**, `end()` is equivalent to `now()`, returning the current time. [cite: 648] [cite_start]For **subqueries**, it returns the end time of that specific subquery's search interval. [cite: 649] [cite_start]It's important to note that `end()` is **not compatible** with parsers because they do not operate within a search interval. [cite: 651]

***

## Example

### Calculate Event Age Relative to the End of the Search

This example uses `end()` to calculate the age of an event relative to the end of the search window, creating a new boolean field `isold` if the event is older than one second.

* **Query Example**
    ```
    isold := (end() - @timestamp) > 1000
    ```

* **Step-by-Step**
    1.  [cite_start]The query starts with the source repository events. [cite: 687]
    2.  [cite_start]`end()` returns the timestamp for the end of the search interval. [cite: 691]
    3.  [cite_start]`@timestamp` is the timestamp of the individual event. [cite: 691]
    4.  [cite_start]The difference `(end() - @timestamp)` calculates the age of the event relative to the end of the search. [cite: 692]
    5.  [cite_start]This difference is compared to `1000` milliseconds (1 second). [cite: 693] [cite_start]If the age is greater than 1000, the new field `isold` is set to `true`; otherwise, it's `false`. [cite: 693, 694]

* **Summary and Results**
    [cite_start]This query is useful for quickly identifying "old" events within a given time frame. [cite: 697] [cite_start]For example, in a security investigation, knowing if an event occurred at the very beginning or end of a search window can be important for establishing a timeline. [cite: 700]