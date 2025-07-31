# if()

The `if()` function allows you to choose one of two expressions to execute based on a condition, similar to a ternary operator (`? :`) in many programming languages. Unlike a `case` statement, `if()` can be embedded as an expression inside other functions.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_if` | The name of the output field. |
| **condition**| expression | required | The conditional expression to evaluate. The parameter name `condition` can be omitted. |
| **else** | expression | required | The expression to execute if the condition is false. |
| **then** | expression | required | The expression to execute if the condition is true. |

***

## Examples

### Add a Field Based on Values of Another Field

This example uses multiple nested `if()` functions to categorize HTTP status codes into human-readable classes, demonstrating how to build `if-elseif-else` logic.

* **Query Example**
    ```
    statusClass := if(regex("^1", field=statuscode), then="informational", else=
                   if(regex("^2", field=statuscode), then="successful", else=
                   if(regex("^3", field=statuscode), then="redirection", else=
                   if(regex("^4", field=statuscode), then="client error", else=
                   if(regex("^5", field=statuscode), then="server error", else="unknown")))))
    ```
* **Step-by-Step**
    This query creates a new field called `statusClass`. It checks the `statuscode` field in a sequence:
    1.  If the status code starts with a "1", `statusClass` becomes "informational".
    2.  If not, it checks if it starts with a "2", setting `statusClass` to "successful".
    3.  This continues for "3" (redirection), "4" (client error), and "5" (server error).
    4.  If none of the conditions are met, `statusClass` is set to "unknown".

---

### Calculate a Percentage of Successful Status Codes Over Time

This example shows `if()` being used as a pre-processing step to create a binary field, which is then used in a more complex `timeChart()` aggregation to calculate a success rate.

* **Query Example**
    ```
    success := if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
          [sum(success, as=success), count(as=total)]
          | pct_successful := (success / total) * 100
          | drop([success, total])
        }
      ], span=15m, limit=100)
    ```

* **Step-by-Step**
    1.  `if(status >= 500, then=0, else=1)`: The query first creates a new field named `success`. It's set to `0` if the `status` is a server error (500 or greater) and `1` otherwise.
    2.  `timeChart()`: This function then takes the events. Inside its embedded aggregation:
        * `sum(success)` adds up all the `1`s to get a count of successful requests.
        * `count()` gets the total number of requests.
        * These two values are used to calculate the `pct_successful`.
        * The temporary `success` and `total` fields are then dropped from the final output.

* **Summary and Results**
    This query creates a time chart showing the percentage of successful requests per customer. The `if()` function is crucial for transforming the status codes into a numeric value that can be easily aggregated to calculate the percentage.

---

### Determine a Score Based on a Field's Percentile Rank

This example demonstrates using `if()` to assign a categorical score to an event based on where its `filesize` falls in a statistical distribution calculated by the `percentile()` function.

* **Query Example**
    ```
    percentile(filesize, percentiles=[40, 80], as=score)
    | symbol := if(filesize > score_80, then=":+1:", else=if(filesize > score_40, then="so-so", else=":-1:"))
    ```
* **Step-by-Step**
    1.  `percentile()`: This function first calculates the filesize values that correspond to the 40th and 80th percentiles, storing them in fields `score_40` and `score_80`.
    2.  `if()`: A nested `if()` statement then creates a new `symbol` field for each event:
        * If the event's `filesize` is greater than the 80th percentile value (`score_80`), the symbol is set to `:+1:`.
        * If not, it checks if the `filesize` is greater than the 40th percentile value (`score_40`), setting the symbol to `so-so`.
        * If neither is true, the symbol is set to `:-1:`.

* **Summary and Results**
    This query is a powerful way to categorize data based on its relative rank within a dataset, rather than on fixed thresholds. The `if()` function provides the conditional logic to assign these categorical labels.