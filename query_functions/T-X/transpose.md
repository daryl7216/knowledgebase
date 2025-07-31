# transpose()

The `transpose()` function transforms table-like data by converting columns into rows, effectively switching the orientation of the data. This is a great way to reorganize data into a more readable format or prepare it for specific visualizations.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **column** | string | optional <br> default: `"column"` | The name of the field that will contain the original column headers. |
| **header** | string | optional | The name of a field whose values should be used as the new column headers for the transposed table. |
| **limit** | integer | optional <br> default: 5 | The maximum number of rows to transpose (from 1 to 1,000). |
| **pivot** | string | optional | A shorthand to use the same field for both the `header` and `column` values. |

***

## Examples

### Create a Pivot Table

This complex example shows how to create a pivot table to summarize the last time each user performed a specific action, using a nested `groupBy` to prepare the data and `transpose` to format it.

* **Query Example**
    ```
    groupBy([type, actor.user.id], function={groupBy(actor.user.id, function=max(@timestamp))})
    | transpose(header=type)
    | drop(column)
    ```

* **Step-by-Step**
    1.  **Nested `groupBy()`**: The query first performs a nested aggregation. The inner `groupBy` finds the maximum (i.e., latest) timestamp for each `actor.user.id`. The outer `groupBy` then groups those results by `type` (the action performed). This creates a flat table with columns: `type`, `actor.user.id`, and `_max` (the timestamp).
    2.  **`transpose(header=type)`**: The `transpose()` function then pivots this table. It uses the unique values from the `type` column (e.g., "alert.create", "dashboard.create") as the new column headers. The values in the table become the `actor.user.id` and `_max` timestamp.
    3.  **`drop(column)`**: Transposing creates a leftover metadata field named `column` by default. This final step removes that field to clean up the output.

* **Result Event Data**
    The final output is a pivot table where the columns are the event types, and the rows show the last user to perform that action and the timestamp of when it occurred.

| alert.clear-error | alert.create | alert.update |
| :--- | :--- | :--- |
| 1700546666592 | 1699004139419 | 1700546666676 |
| 007WGPBX9YbvZbKOrBMd5fgH| 007WGPBX9YbvZbKOrBMd5fgH| 007WGPBX9YbvZbKOrBMd5fgH|

---

### Search Across Multiple Structured Fields

This example demonstrates an advanced technique where `transpose()` is used to enable searching for a value across all fields of an event simultaneously.

* **Query Example**
    ```
    groupBy(@id, function=transpose())
    | row[1] = /httpd/
    | groupBy(column)
    ```

* **Input Event Data**
    Standard log events with multiple fields.

| host | @rawstring |
| :--- | :--- |
| MAIL01 | `... MAIL01 httpd[61789]: ...` |
| LON-SRV01| `... LON-SRV01 httpd[60123]: ...` |

* **Step-by-Step**
    1.  `groupBy(@id, function=transpose())`: This step transposes each event individually. For each event, it creates a set of new events where each new event has two fields: `column` (containing the original field name, e.g., "host", "app") and `row[1]` (containing the original field's value, e.g., "MAIL01", "httpd[61789]:").
    2.  `row[1] = /httpd/`: Now that all values are in a single field (`row[1]`), a simple filter can search across all of them. This line keeps only the transposed rows where the value contains "httpd".
    3.  `groupBy(column)`: Finally, the query groups by the `column` field. This produces a final count of which original fields contained the search term "httpd".

* **Result Event Data**
    The final output is a table listing the original field names that contained the value "httpd" and a count of how many times it appeared in each.

| column | _count |
| :--- | :--- |
| @rawstring | 5 |
| app | 5 |