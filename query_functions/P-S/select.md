# select()

The `select()` function specifies a set of fields to select from each event and include in the resulting event set. It is possible that an aggregate function, such as `table()` or `groupBy()` may be more suitable for summarizing and selecting the fields that you want to be displayed. A use-case for `select()` is when you want to export a few fields from a large number of events into a CSV file without aggregating the values. Because an implicit `tail(200)` function is appended in non-aggregating queries, only 200 events might be shown in those cases; however, when exporting the result, you get all matching events.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `fields` | array of strings | required | The names of the fields to keep. |

***

## Function Operation

The `select()` function is used to narrow down the fields included in each event within the result set. It allows you to specify exactly which fields you want to retain, discarding all others. While aggregate functions like `table()` or `groupBy()` might be more appropriate for summarizing and displaying specific fields, `select()` is particularly useful for scenarios where you need to export a subset of fields from a large number of events to a CSV file without performing any aggregation.

When using `select()` in non-aggregating queries within the LogScale UI, an implicit `tail(200)` function is appended, meaning only the last 200 events might be displayed. However, when exporting the results, all matching events with the selected fields will be included in the CSV file, regardless of the UI's display limit. The `fields` parameter, which can be omitted if the context clearly defines the fields (e.g., in a pipeline where fields are directly assigned), is required to specify the names of the fields to keep.

***

## Examples

### Reduce Large Event Sets to Essential Fields

This example reduces a large dataset to essential fields using the `select()` function, focusing on `statuscode` and `responsetime` for `GET` requests.

  * **Query Example**

    ```
    method=GET
    select([statuscode, responsetime])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `method=GET`: Filters for all HTTP request methods of the type `GET`.
    3.  `select([statuscode, responsetime])`: Creates an unsorted table showing only the `statuscode` field and the `responsetime` field for the filtered events.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to filter specific fields from an event set and create a table showing these focused fields. In this example, the table shows the HTTP response status and the time taken to respond to the request, which is useful for analyzing HTTP performance, monitoring response codes, and identifying slow requests. The `select()` function is useful when you want to export a few fields from a large number of events into a CSV file without aggregating the values. Note that while the LogScale UI may only show up to 200 events, the exported CSV file will contain all results.

---

### Select Fields to Export

This example demonstrates how to select specific fields (`@timestamp` and `@rawstring`) for export as a CSV file.

  * **Query Example**

    ```
    select([@timestamp, @rawstring])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `select([@timestamp, @rawstring])`: Creates an unsorted table showing the `@timestamp` field and the `@rawstring` field for each event.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to filter specific fields from an event set and create a table showing these fields (focused event set). In this example, the table shows the timestamp of the events and the complete raw log entry, which is useful for full log analysis and data backup. The `select()` function is useful when you want to export a few fields from a large number of events into a CSV file without aggregating the values. Note that while the LogScale UI can only show up to 200 events, an exported CSV file contains all results.