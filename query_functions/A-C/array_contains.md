# array:contains()

The `array:contains()` function checks if a specified value exists within an array. If the value is not found in the array for a given event, that event is excluded from the search results.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to search within. The parameter name `array` can be omitted. |
| **value** | string | required | The exact value to search for within the array. |

***

## Function Operation

This function acts as a filter for events. It evaluates each event's specified array field to see if it contains an exact match for the provided value. If a match is found, the entire event is included in the output; otherwise, it is dropped.

***

## Examples

### Aggregate Array Content

This example finds all events where the `incidents` array contains "Cozy Bear" and then groups the results by the `host` field.

* **Query Example**
    ```
    array:contains("incidents[]", value="Cozy Bear")
    | groupBy(host)
    ```

* **Input Event Data**
    The query runs on events that contain an `incidents` array and a `host` field. For instance:
    * **Event 1**: `host` is `v1`, `incidents` array contains "Evil Bear" and "Cozy Bear".
    * **Event 2**: `host` is `v15`, `incidents` array contains "Fancy Fly", "Tiny Cat", and "Cozy Bears".

* **Step-by-Step**
    1.  `array:contains("incidents[]", value="Cozy Bear")`: This part of the query filters the events, keeping only those where the `incidents` array contains the exact value "Cozy Bear".
    2.  `groupBy(host)`: The remaining events are then grouped by the `host` field, and a count is generated.

* **Result Event Data**
    The final output is an aggregated count of events that matched, grouped by host.

| field | value |
| :--- | :--- |
| host | v1 |
| _count | 1 |

---

### Check for Values in Array

This example uses the function as a simple filter to check for a value in an array.

* **Query Example**
    ```
    array:contains("incidents[]", value="Cozy Bear")
    ```

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  `array:contains("incidents[]", value="Cozy Bear")`: It checks if the value "Cozy Bear" exists within the `incidents` array field of each event.
    3.  If the value is found, the entire event is kept in the search results. If not, the event is excluded.

* **Summary and Results**
    The query is used as a filter to see if a specific value is present in an array within a set of events. If the array contains the searched value, the event is returned in the results, including all other original fields and values in that event.
