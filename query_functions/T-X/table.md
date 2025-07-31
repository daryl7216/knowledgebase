# table()

The `table()` function is an aggregate function that displays query results in a table, allowing you to specify which fields to include and how to sort them.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **fields** | array of strings | required | The names of the fields to select for the table columns. The parameter name `fields` can be omitted. |
| **limit** | number | optional <br> default: 200 | The maximum number of rows to include in the result. |
| **order** | string | optional <br> default: `desc` | The order to sort in. Can be `asc` (ascending) or `desc` (descending). |
| **reverse** | boolean | optional (deprecated) | Deprecated parameter. Use `order` instead. |
| **sortby** | array of strings | optional <br> default: `@timestamp`| The names of the fields to sort the results by. |
| **type** | array of strings | optional <br> default: `number` | The data type of the fields to sort by. Can be `any`, `hex`, `number`, or `string`. |

***

## Function Operation

The `table()` function aggregates events to create a tabular view. It performs three main actions:
1.  **Sorts Columns**: Arranges the columns in the table in the order specified in the `fields` parameter.
2.  **Limits Rows**: Restricts the number of events (rows) returned based on the `limit` parameter.
3.  **Sorts Results**: Sorts the rows of the table according to the `sortby` and `order` parameters.

For large data exports, the `select()` function is recommended as an alternative, as it provides similar output without the row limits and sorting constraints of `table()`.

***

## Example

### Calculate and Display Query Costs by User and Repository

This complex example demonstrates how `table()` can be used as the final step in a data analysis pipeline to present aggregated and sorted results in a clean, readable format.

* **Query Example**
    ```
    #type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"
    | repoUser := format("%s/%s", field=[dataspace, initiatingUser])
    | top(repoUser, sum(deltaTotalCost), as=cost)
    | table([cost, repoUser], sortby=cost)
    ```

* **Input Event Data**
    Internal logs from Humio that contain information about query costs.

| #type | #kind | class | message |
| :--- | :--- | :--- | :--- |
| humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query |
| humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query |
| ... | ... | ... | ... |

* **Step-by-Step**
    1.  The query first filters for specific internal logs related to high-cost queries.
    2.  `format()`: It then combines the `dataspace` (repository) and `initiatingUser` fields into a single, new field called `repoUser` (e.g., `"development/jane.smith"`).
    3.  `top()`: This function finds the most common `repoUser` values and, for each one, calculates the `sum()` of their query costs, storing it in a new field named `cost`.
    4.  `table()`: Finally, `table()` takes this aggregated data and displays it. It creates a table with two columns, `cost` and `repoUser`, and sorts the rows by the `cost` field in descending order (the default).

* **Result Event Data**
    The output is a sorted table that clearly shows the total query cost for each user and repository combination, with the highest costs listed first.

| cost | repoUser |
| :--- | :--- |
| 3200 | development/jane.smith |
| 2300 | production/john.doe |
| 1500 | staging/bob.wilson |