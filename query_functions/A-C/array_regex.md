# array:regex()

The `array:regex()` function filters events by checking if a regular expression pattern matches any of the values within a specified array. If no value in the array matches the pattern for a given event, that event is excluded from the search results.

> **Note**: It is recommended to always test regular expressions directly within LogScale to ensure compatibility, rather than relying on third-party tools.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to search within. The parameter name `array` can be omitted. |
| **flags** | string | optional | The regex modifier flags to use. Possible values are:<br>- `F`: Use LogScale Regex Engine v2.<br>- `d`: Period (`.`) also includes newline characters.<br>- `i`: Ignore case for matched values.<br>- `m`: Multi-line parsing of regular expressions. |
| **regex** | regex | required | The regular expression pattern to search for within the array values. |

***

## Example

### Find Matches in Array and Group Results

This example uses a regular expression to find events where an array element starts with "Cozy Bear" and then groups the results by host.

* **Query Example**
    ```
    array:regex("incidents[]", regex="^Cozy Bear.*")
    | groupBy(host)
    ```

* **Input Event Data**
    A set of events each containing a `host` field and an `incidents` array.

| host | incidents[0] | incidents[1] | incidents[2] |
| :--- | :--- | :--- | :--- |
| v1 | Evil Bear | Cozy Bear | |
| v15 | Fancy Fly | Tiny Cat | Cozy Bears |
| v22 | Fancy Fly | Tiny Cat | Cold Bears |
| v4 | Fancy Fly | Tiny Cat | Cozy Bearskins|
| v1 | Evil Bear | Cozy Bears | |

* **Step-by-Step**
    1.  The query starts with the source events.
    2.  `array:regex("incidents[]", regex="^Cozy Bear.*")`: This filters the events, keeping only those where at least one element in the `incidents` array starts with the string "Cozy Bear".
    3.  `groupBy(host)`: The events that pass the filter are then grouped by their `host` value, and a count is generated for each host.

* **Result Event Data**
    The output is an aggregated count of matching events for each host.

| host | _count |
| :--- | :--- |
| v1 | 2 |
| v15 | 1 |
| v4 | 1 |