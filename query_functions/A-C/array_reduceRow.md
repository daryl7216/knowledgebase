# array:reduceRow()

The `array:reduceRow()` function computes an aggregated value for the array within each event individually.

> **Important**
> This function is experimental, under active development, and should not be used in production. It must be enabled using the `ArrayFunctions` feature flag.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to process. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_reduceRow` | The name for the output field. |
| **function**| function | required | The aggregate function to use (e.g., `avg()`, `max()`), which must output a single event with a single field. |
| **var** | string | required | A placeholder name for the array element to be used in the aggregate function. |

***

## Function Operation

This function operates on each event (or row) separately. For each event, it applies an aggregate function to all the elements within that event's array. It is essentially a shorthand for using `array:eval()` to process each event's array. The result is a new field added to each event, containing the calculated value for that event's array.

***

## Example

### Calculate Average of Field Values in an Array

This example calculates the average of values in an `ages[]` array for each individual event.

* **Query Example**
    ```
    array:reduceRow("ages[]", var=x, function=avg(x))
    ```

* **Input Event Data**
    A series of events, each containing its own `ages[]` array.

| ages[0] | ages[1] | ages[2] |
| :--- | :--- | :--- |
| 16 | 32 | 64 |
| 15 | 30 | 45 |
| 1 | 2 | 4 |
| 89 | 57 | 67 |

* **Step-by-Step**
    1.  The query starts with a set of events, each with an `ages[]` array.
    2.  `array:reduceRow()` processes each event one by one.
    3.  For the first event, it calculates the average of `{16, 32, 64}`.
    4.  For the second event, it calculates the average of `{15, 30, 45}`, and so on for all subsequent events.
    5.  The calculated average for each event is placed in a new `_avg` field for that same event.

* **Result Event Data**
    The original events are returned with a new field, `_avg`, showing the calculated average for each row.

| ages[0] | ages[1] | ages[2] | _avg |
| :--- | :--- | :--- | :--- |
| 16 | 32 | 64 | 37.333 |
| 15 | 30 | 45 | 30 |
| 1 | 2 | 4 | 2.67 |
| 89 | 57 | 67 | 71 |