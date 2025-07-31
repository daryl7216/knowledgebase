# top()

The `top()` function is an efficient way to find the most common values of a field in a set of events. It is a more powerful and concise alternative to running `groupBy()`, `count()`, and `sort()` together. It can also find the top values based on the `sum` or `max` of another field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_count` or `_sum` | The name of the output field for the aggregated value. |
| **error** | number | optional <br> default: 5 | The error threshold in percentage. A warning is shown if the approximative algorithm's precision is lower than this. |
| **field** | array of strings | required | The field(s) on which to group and find the top values. |
| **limit** | number | optional <br> default: 10 | The number of top results to return. |
| **max** | string | optional | Changes the aggregation function from `count()` to find the `max()` of the specified field. |
| **percent** | boolean | optional <br> default: `false` | If `true`, adds a `_percent` column showing the percentage of the total count. |
| **rest** | string | optional | If specified, adds an extra row that aggregates all other values not included in the top results. The string provided becomes the label for this row. |
| **sum** | string | optional | Changes the aggregation function from `count()` to find the `sum()` of the specified field. |

***

## Function Operation

The `top()` function uses an approximative algorithm to efficiently compute the most frequent items, which is guaranteed to be exact for a certain number of unique items but may be an estimate for very high cardinality fields. It only returns events that are guaranteed to be in the top results (no false positives).

***

## Examples

### Calculate Query Costs by User and Repository

This example shows how to use `top()` with the `sum` parameter to find which users have the highest total query costs, rather than just the highest number of queries.

* **Query Example**
    ```
    #type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"
    | repoUser := format("%s/%s", field=[dataspace, initiatingUser])
    | top(repoUser, sum(deltaTotalCost), as=cost)
    | table([cost, repoUser], sortby=cost)
    ```

* **Step-by-Step**
    1.  The query first filters for internal logs that report on high-cost queries.
    2.  `format()`: The `dataspace` (repository) and `initiatingUser` fields are combined into a single `repoUser` field for easier grouping.
    3.  `top(repoUser, sum(deltaTotalCost), as=cost)`: This is the key step. It groups by the combined `repoUser` field. Instead of counting, it calculates the `sum()` of the `deltaTotalCost` for each user and renames the resulting sum field to `cost`. It then returns the top users with the highest total cost.
    4.  `table()`: The final results are displayed in a clean, sorted table.

* **Result Event Data**
    The output is a table showing the total query cost for each user/repository pair, sorted by the highest cost.

| cost | repoUser |
| :--- | :--- |
| 3200 | development/jane.smith |
| 2300 | production/john.doe |
| 1500 | staging/bob.wilson |

---

### Extract the Top Most Viewed Pages and Group the Rest

This example demonstrates how to extract a value with `regex`, find the top 12 most frequent values, and group all other less frequent values into a single "others" category.

* **Query Example**
    ```
    regex(regex="/.*/(?<url_page>\\S+\\.page)", field=url)
    | top(url_page, limit=12, rest=others)
    ```
* **Step-by-Step**
    1.  `regex()`: A regular expression is used to extract the page name (ending in `.page`) from a `url` field and store it in a new field called `url_page`.
    2.  `top(url_page, limit=12, rest=others)`: The `top()` function then finds the 12 most frequent values in the `url_page` field. The `rest=others` parameter instructs the function to create an additional row labeled "others" that contains the aggregated count of all pages that didn't make it into the top 12.

* **Result Event Data**
    The output is a table of the top 12 most viewed pages and their counts, plus a final row for all other page views combined.

| url_page | _count |
| :--- | :--- |
| home.page | 51 |
| index.page | 21 |
| ... | ... |
| others | 153 |

---

### Create a Frequency Count With Formatted Links

This example shows how `top()` can be used to generate data that is then piped into another function (`format()`) for enhanced presentation, such as creating clickable links in the UI.

* **Query Example**
    ```
    top(repo)
    | format("[Link](https://example.com/%s)", field=repo, as=link)
    ```

* **Input Event Data**
    A series of events with a `repo` field indicating activity in different code repositories.

| @timestamp | repo |
| :--- | :--- |
| 2023-06-15T10:00:00Z | "frontend-app" |
| 2023-06-15T10:05:00Z | "backend-api" |
| 2023-06-15T10:10:00Z | "frontend-app" |
| ... | ... |

* **Step-by-Step**
    1.  `top(repo)`: This function finds the most frequently occurring repository names and provides a count for each, sorted in descending order.
    2.  `format(...)`: The results are then piped to `format()`. For each repository name, it constructs a markdown link string, inserting the `repo` field value into the URL. This new link is stored in a field named `link`.

* **Result Event Data**
    The output is an interactive table showing each repository, its activity count, and a clickable link to that repository.

| repo | _count | link |
| :--- | :--- | :--- |
| frontend-app | 4 | `[Link](https://example.com/frontend-app)` |
| backend-api | 2 | `[Link](https://example.com/backend-api)` |
| ... | ... | ... |