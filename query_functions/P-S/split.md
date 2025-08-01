# split()

The `split()` function splits an event structure that includes an array into multiple distinct events with each array element.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | optional \<br\> default: `events` | Field to split by. |
| `strip` | boolean | optional \<br\> default: `false` | Strip the field prefix when splitting (default is `false`). |

***

## Function Operation

When LogScale ingests data into arrays, each array entry is turned into separate attributes named like `[0]`, `[1]`, etc. The `split()` function takes such an event and divides it into multiple distinct events, where each new event corresponds to an individual element of the original array. This is done based on the prefix of these `[N]` attributes, enabling aggregate functions to be applied across array values.

When `split()` is called, each generated split event is assigned a unique index ID in the `_index` field, which can be used to identify the individual event. If the original event data includes an `@id` field, then the `@id` field is also modified for each new event by appending `_` and the index number (e.g., `1_0`, `1_1`, `1_2`).

**Important Note on Efficiency and Memory Usage:**
The `split()` function is not very efficient and should only be used after aggressive filtering. When `split()` creates new events from an array, it copies *all* fields from the original event into each new split event by default, with the exception of the array that is being split. This includes large fields like `@rawstring`. Splitting an event containing a large `@rawstring` can consume a significant amount of memory (e.g., a 1MB `@rawstring` split into 100 events would consume 100MB of memory). This memory impact is particularly problematic when `split()` is used in parsers where events are stored for longer periods.

**Best practice:** Drop any unnecessary large fields, especially `@rawstring`, before using `split()`. For example:

```
drop(@rawstring)
| split(array_to_split)
```

***

## Examples

### Splitting an event containing an array

Given an input record: `@id=1, a=[1,2,3]`

  * **Query Example**

    ```
    split(a)
    ```

  * **Result Event Data**

| @id | \_index | a |
| :--- | :--- | :--- |
| 1\_0 | 0 | 1 |
| 1\_1 | 1 | 2 |
| 1\_2 | 2 | 3 |

---

### Expanding GitHub PushEvent commits

In GitHub events, a `PushEvent` contains an array of commits, and each commit gets expanded into sub-attributes of `payload.commit0`, `payload.commit1`, etc. LogScale cannot sum/count across such attributes directly. This example shows how `split()` expands each `PushEvent` into one `PushEvent` for each commit so they can be counted.

  * **Query Example**

    ```
    type=PushEvent
    split(payload.commits)
    groupBy(payload.commits.author.email)
    | sort()
    ```

  * **Step-by-Step**

    1.  `type=PushEvent`: Filters for events of type `PushEvent`.
    2.  `split(payload.commits)`: Splits each `PushEvent` event into multiple events, one for each commit within the `payload.commits` array. Each new event will contain the data for a single commit.
    3.  `groupBy(payload.commits.author.email)`: Groups the new events by the email of the commit author.
    4.  `| sort()`: Sorts the results (likely by count, as it's a grouped aggregation).

---

### Splitting events from a JSON array in `@rawstring`

If your parser receives JSON events in a JSON array, and each record in the array is an event in itself, you would like to split them out. First, you need to call `parseJson()`. When `@rawstring` contains an array, the `parseJson()` function doesn't assign names to the fields automatically; it only assigns indexes (e.g., `[0].exampleField`, `[1].exampleField`). Since `split()` needs a field name before it reads indexes, we can tell `split()` to look for the empty field name by calling `split(field="")`.

  * **Input Event Data**

    ```json
    [
      {"exampleField": "value"},
      {"exampleField": "value2"}
    ]
    ```

  * **Input Event Data (after `parseJson()`):**

| @rawstring | @timestamp | @timestamp.nanos | \[0].exampleField | \[1].exampleField |
| :--- | :--- | :--- | :--- | :--- |
| `[ {"exampleField": "value"}, {"exampleField": "value2"} ]` | 2023-10-27 10:33:42.748 | 0 | value | value2 |

  * **Query Example**

    ```
    parseJson()
    | split(field="")
    ```

  * **Result Event Data**

| .exampleField | @rawstring | @timestamp | @timestamp.nanos | \_index |
| :--- | :--- | :--- | :--- | :--- |
| value | `{"exampleField": "value"}, {"exampleField": "value2"}` | 2023-10-27 | 10:33:18.944 | 0 |
| value2 | `{"exampleField": "value"}, {"exampleField": "value2"}` | 2023-10-27 | 10:33:18.944 | 1 |

---

### Alternative for splitting events from a JSON array with `prefix`

Alternatively, you can tell `parseJson()` to add a prefix to all the fields, which can then be used as the field name to split on.

  * **Query Example**

    ```
    parseJson(prefix="example")
    | split(field="example")
    ```

  * **Summary and Results**
    This method adds the `example` prefix to all fields on the new event, so `split(field="")` might be preferred to avoid that.

---

### Deduplicate Compound Field Data With `array:union()` and `split()`

This example demonstrates deduplicating fields of information where there are multiple occurrences of a value in a single field, possibly separated by a single character. This solution uses `array:union()` and `split()` to create a unique array and then split the content out to a unique list.

  * **Query Example**

    ```
    splitString(field=userAgent, by="", as=agents)
    array:filter(array="agents[]", function={bname=/\//}, var="bname")
    array:union(array=agents, as=browsers)
    split(browsers)
    ```

  * **Input Event Data**
    (Example `userAgent` raw event data):
    `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome`
    (The actual names are the Name/Version pairs showing compatibility with different browser standards. Resolving this into a simplified list requires splitting up the list, simplifying (to remove duplicates), filtering, and then summarizing the final list.)

  * **Step-by-Step**

    1.  Starting with the source repository events.

    2.  `splitString(field=userAgent, by="", as=agents)`: Splits the `userAgent` field into individual array entries within the `agents` array for each event.

          * **Result Event Data (after splitString)**:

        | agents\[0] | agents\[1] | agents\[2] | agents\[3] | agents\[4] |
        | :--- | :--- | :--- | :--- | :--- |
        | Mozilla/5.0 | (Macintosh; | Intel | Mac | OS |

    3.  `array:filter(array="agents[]", function={bname=/\//}, var="bname")`: Filters the `agents` array. (The provided example for this step is incomplete in the source).

    4.  `array:union(array=agents, as=browsers)`: Aggregates the list of user agents across all events to create a list of unique entries, eliminating duplicates where the value of the user agent is the same.

          * **Result Event Data (after array:union)**:

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

    5.  `split(browsers)`: Splits the `browsers` array into individual events, turning the single event with an array into multiple events, each representing a browser.

          * **Input Event Data (for split)**: (Implicitly, the event after `array:union` as shown above)

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

          * **Result Event Data (after split)**:

        | \_index | browsers |
        | :--- | :--- |
        | 0 | Gecko/20100101 |
        | 1 | Safari/537.36 |
        | 2 | AppleWebKit/605.1.15 |

    6.  Event Result set.

  * **Summary and Results**
    The resulting output from the query is a list of events with each event containing a matching index and browser. This can be useful if you want to perform further processing on a list of events rather than an array of values.