# stats()

The `stats()` function is used to compute multiple aggregate functions over the input.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `function` | array of aggregate functions | optional \<br\> default: `count(as=_count)` | Specifies which aggregate functions to perform on each group. |

***

## Function Operation

The `stats()` function is a versatile tool for applying multiple aggregate functions simultaneously over a set of input events. It is implicitly used in various contexts where a list of sub-aggregators is provided (e.g., `groupBy(x, function=[min(y), max(y)])` is equivalent to `groupBy(x, function=stats([min(y), max(y)]))`). This mechanism enables the combination of results from different aggregators.

The output behavior of `stats()` depends on the sub-aggregators:

  * **Single-Row Output**: If all sub-aggregators produce at most one row of output (common for most numerical aggregators like `sum()`, `avg()`, `min()`, `max()`), the result will be a single combined row containing all the aggregated values.
  * **Multi-Row Output (Cartesian Product)**: If one or more sub-aggregators emit more than one result row (e.g., `table()`), the total output will be the Cartesian product of all the sub-aggregators' outputs. This means every row from one sub-aggregator is combined with every row from another. If any sub-aggregator outputs zero rows, it is treated as outputting a single empty row for the purpose of the Cartesian product.
  * **Field Name Collisions**: The output combination checks for field name collisions. It is considered an error if a field is present in multiple outputs with conflicting values.

`stats()` is also available as a shorthand syntax by simply listing aggregators within square brackets (e.g., `| [min(), max()]` is equivalent to `stats(function=[min(), max()])`). A simple `stats(function=count())` is equivalent to just `count()`.

***

## Examples

### Annotate Events With Aggregation - Example 1

This example demonstrates using the `stats()` function with aggregation on field `x` where one sub-aggregator outputs multiple rows.

  * **Query Example**

    ```
    kvParse()
    | stats([
        avg(x),
        table([x])
    ])
    ```

  * **Input Event Data**
    (Events containing `x` field values):
    `x=1`
    `x=2`
    `x=9`
    `x=10`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs (e.g., extracts `x=1` as a field `x` with value `1`).
    3.  `| stats([avg(x), table([x])])`: Computes two aggregate functions over the field `x`:
          * `avg(x)`: Calculates the average of `x` values, returning a single row with a field named `_avg`.
          * `table([x])`: Returns a row for each `x` value, producing multiple rows.
            Since `table([x])` outputs multiple rows, the final output is the Cartesian product of the `_avg` result and the `x` values from `table([x])`.
    4.  Event Result set.

  * **Summary and Results**
    The query computes multiple aggregate functions over an input. The result shows the average `_avg` repeated for each `x` value due to the Cartesian product.

  * **Result Event Data**

| \_avg | x |
| :--- | :--- |
| 5.5 | 1 |
| 5.5 | 2 |
| 5.5 | 9 |
| 5.5 | 10 |

---

### Annotate Events With Aggregation - Example 2

This example illustrates using the `stats()` function with aggregation where one of the sub-aggregators (`avg(y)`) outputs zero rows.

  * **Query Example**

    ```
    kvParse()
    | stats([
        sum(x,as=sumX),
        avg(y, as=avgY),
        table([x,y])
    ])
    ```

  * **Input Event Data**
    `"x=1 y=N/A"`
    `"x=2 y=N/A"`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs.
    3.  `| stats([sum(x, as=sumX), avg(y, as=avgY), table([x, y])])`: Computes aggregate functions over fields `x` and `y`:
          * `sum(x, as=sumX)`: Sums `x` values.
          * `avg(y, as=avgY)`: Attempts to average `y` values. Since `y=N/A` (which is likely not a number), this aggregator will produce zero rows.
          * `table([x, y])`: Returns rows for each combination of `x` and `y`.
            Since `avg(y)` outputs zero rows, the `stats()` function treats it as outputting a single empty row for the Cartesian product.
    4.  Event Result set.

  * **Summary and Results**
    The query computes multiple aggregate functions over an input. Despite `avg(y)` not producing a meaningful numerical result, its presence in `stats()` influences the Cartesian product.

  * **Result Event Data (example shown in document)**
    (Header): `sumX`, `x`, `y`
    (Row 1): `3`, `1`, `N/A`
    (Row 2): `3`, `2`, `N/A`

---

### Annotate Events With Aggregation - Example 3

This example demonstrates using the `stats()` function with aggregation where all of the sub-aggregators output multiple rows, resulting in a Cartesian product.

  * **Query Example**

    ```
    kvParse()
    | stats([
        table([x,y]),
        table([z])
    ])
    ```

  * **Input Event Data**
    `"x=1 y=10 z=100"`
    `"x=2 y=20 z=200"`

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `kvParse()`: Parses the string into key-value pairs.
    3.  `| stats([table([x,y]), table([z])])`: Computes aggregate functions using `table()` over the fields `x`, `y`, and `z`. Since both `table([x,y])` and `table([z])` sub-aggregators output multiple rows, the returned result is the Cartesian product, containing all combinations of the results from the sub-aggregators.
    4.  Event Result set.

  * **Summary and Results**
    The query computes multiple aggregate functions over an input. The result is a combination of all outputs (the Cartesian product) in fields named `x`, `y`, and `z`.

  * **Result Event Data**

| x | y | z |
| :--- | :--- | :--- |
| 1 | 10 | 100 |
| 1 | 10 | 200 |
| 2 | 20 | 100 |
| 2 | 20 | 200 |

---

### Calculate Minimum and Maximum Response Times

This example calculates minimum and maximum response times using multiple aggregate functions in square brackets, which is shorthand for `stats()`.

  * **Query Example**

    ```
    [min_response:= min(responsetime), max_response:= max(responsetime)]
    ```

  * **Input Event Data**

    | @timestamp | endpoint | responsetime | status\_code |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | /api/users | 145 | 200 |
    | 1686837826000 | /api/products | 892 | 200 |
    | 1686837827000 | /api/orders | 167 | 200 |
    | 1686837828000 | /api/payment | 1290 | 500 |
    | 1686837829000 | /api/users | 156 | 200 |
    | 1686837830000 | /api/items | 78 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `[min_response:= min(responsetime), max_response = max(responsetime)]`: In a single operation, calculates the minimum value from the `responsetime` field and returns the result in a field named `min_response`. It also calculates the maximum value from the `responsetime` field and returns the result in a field named `max_response`. Square brackets allow multiple aggregations to be performed in a single operation (shorthand for `stats()`).
    3.  Event Result set.

  * **Summary and Results**
    The query finds the range of response times by calculating both the minimum and maximum values. The results are returned in fields with names specified in the field assignments. This query is useful for monitoring service performance, identifying outliers in response times, or establishing performance baselines. Only one row is returned containing both calculated values.

  * **Result Event Data**

| min\_response | max\_response |
| :--- | :--- |
| 78 | 1290 |

---

### Count Total Events

This example counts the total number of events in the result set using the `stats()` function.

  * **Query Example**

    ```
    stats(function=count())
    ```

  * **Input Event Data**

    | @timestamp | status\_code | endpoint | response\_time |
    | :--- | :--- | :--- | :--- |
    | 1686837825000 | 200 | /api/users | 145 |
    | 1686837826000 | 404 | /api/products | 89 |
    | 1686837827000 | 200 | /api/orders | 167 |
    | 1686837828000 | 500 | /api/payment | 890 |
    | 1686837829000 | 200 | /api/users | 156 |
    | 1686837830000 | 404 | /api/items | 78 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `stats(function=count())`: Counts the total number of events in the result set. The `count()` function is passed as an argument to `stats()` and returns the count in a field named `_count`. This query is actually equivalent to just `count()`.
    3.  Event Result set.

  * **Summary and Results**
    The query gets a simple count of the total number of events matching the query. This is useful for monitoring event volumes, verifying data ingestion, or getting quick counts of specific event types when combined with filters. Only one row is returned containing the total count.

  * **Result Event Data**

| \_count |
| :--- |
| 6 |