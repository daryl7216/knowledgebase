# drop()

The `drop()` function is a filter that removes specified fields or columns from a result set.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **fields** | array of strings | required | The name of the field or a list of field names to be discarded. The parameter name `fields` can be omitted. |

***

## Example

### Calculate a Percentage and Drop Intermediate Fields

This advanced example shows `drop()` being used to remove temporary fields that were created for an intermediate calculation, ensuring the final output is clean. The query calculates the percentage of successful HTTP status codes over time.

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
    1.  A temporary field `success` is created, assigned `1` for successful status codes and `0` for server errors.
    2.  `timeChart()` begins creating a time series chart.
    3.  Inside an embedded aggregation `{...}`:
        * The `sum()` of the `success` field and a `count()` of all events are calculated, creating two new temporary fields: `success` and `total`.
        * These fields are used to calculate the `pct_successful`.
        * `drop([success, total])` is then used to remove the temporary `success` and `total` fields, as they are no longer needed in the final result.

* **Summary and Results**
    The query generates a time chart that shows only the final `pct_successful` value for each customer. The `drop()` function is essential here for cleaning up the intermediate data used in the calculation, making the final output focused and relevant.