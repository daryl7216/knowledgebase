# eventFieldCount()

[cite_start]The `eventFieldCount()` function computes the number of internal fields an event contains and adds this count to the event in a new field[cite: 3176]. [cite_start]It's important to note that tags are generally not counted as fields[cite: 3176].

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_eventFieldCount` | [cite_start]The name of the output field that will store the field count[cite: 3177]. |

***

## Example

### Search for Events by Number of Fields

[cite_start]This example demonstrates how to find events that have a low field count, which can be useful for identifying sparsely populated or potentially incomplete events[cite: 3182, 3204].

* **Query Example**
    ```
    eventFieldCount()
    | _eventFieldCount < 6
    ```
    [cite_start][cite: 3185, 3186]

* **Step-by-Step**
    1.  `eventFieldCount()`: This function is first called to go through the events. [cite_start]It calculates the number of fields for each event and adds a new field, `_eventFieldCount`, containing this number[cite: 3195].
    2.  `| [cite_start]_eventFieldCount < 6`: The events, now augmented with the field count, are then piped to a filter[cite: 3189]. [cite_start]This filter keeps only the events where the value of `_eventFieldCount` is less than 6[cite: 3200].

* **Summary and Results**
    [cite_start]The query is used to get an overview of events with a low field count[cite: 3204]. [cite_start]The final result set will only include events that contain fewer than six fields[cite: 3200].