# eventInternals()

The `eventInternals()` function adds a set of fields to an event that describe its storage locations, such as the datasource ID.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **prefix** | string | optional <br> default: `@` | The prefix for the new fields that are created. |

***

## Example

### Count Events From Each Datasource

This example uses `eventInternals()` to add storage location information to events, and then uses that information to group and count the number of events coming from each datasource.

* **Query Example**
    ```
    eventInternals()
    | groupBy([@datasourceId])
    ```

* **Step-by-Step**
    1.  `eventInternals()`: This function first processes the events and adds fields describing their storage location. In this case, it adds the `@datasourceId` field to each event.
    2.  `groupBy([@datasourceId])`: The events, now enriched with the datasource ID, are then grouped by this new field. By default, `groupBy` will also count the number of events in each group and place the result in a `_count` field.

* **Summary and Results**
    The query is used to find the number of events per datasource. The final output is a table listing each unique datasource ID and the total count of events that originated from it.