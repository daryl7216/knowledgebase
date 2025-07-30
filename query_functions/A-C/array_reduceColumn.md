# array:reduceColumn()

The `array:reduceColumn()` function computes an aggregate value for each array element that shares the same index across multiple events.

> **Important**
> This function is experimental and should not be used in production environments. It must also be enabled using the `ArrayFunctions` feature flag.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The prefix of the array to process. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_reduceColumn` | The name for the output array. |
| **function**| array of aggregate functions | required | The aggregate function(s) to apply to each column (e.g., `max()`, `avg()`). |
| **var** | string | required | A placeholder name for array elements to be used in the aggregate function. |

***

## Function Operation

This function works by grouping together all array elements that have the same index from all input events. It then applies a specified aggregate function to each of these groups (or "columns"). For instance, it would take all elements at index `[0]` from all events and calculate their average, then do the same for all elements at index `[1]`, and so on. The results are returned in a new array where each index corresponds to the result of the calculation for that column.

***

## Example

### Compute Average Value for Each Array Element With Same Index

This example calculates the average value for each column across multiple events containing an `ages[]` array.

* **Query Example**
    ```
    array:reduceColumn("ages[]", var=x, function=avg(x))
    ```

* **Input Event Data**
    Four separate events, each with an `ages[]` array of the same length.

| ages[0] | ages[1] | ages[2] |
| :--- | :--- | :--- |
| 16 | 32 | 64 |
| 15 | 30 | 45 |
| 1 | 2 | 4 |
| 89 | 57 | 67 |

* **Step-by-Step**
    1.  The query starts with a set of events, each containing an `ages[]` array.
    2.  `array:reduceColumn()` groups the values by their index:
        * Column 0: `{16, 15, 1, 89}`
        * Column 1: `{32, 30, 2, 57}`
        * Column 2: `{64, 45, 4, 67}`
    3.  The `avg()` function is applied to each of these columns independently.
    4.  The results are stored in a new array named `_reduceColumn` by default.

* **Result Event Data**
    The output is a single event with one array containing the calculated average for each column.

| _reduceColumn[0] | _reduceColumn[1] | _reduceColumn[2] |
| :--- | :--- | :--- |
| 40.3 | 40.3 | 45.0 |