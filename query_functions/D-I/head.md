# head()

The `head()` function retrieves a specified number of the oldest events first. It sorts events by timestamp (`@timestamp` or `@ingesttimestamp`) to determine which are the oldest.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **limit** | number | optional <br> default: 200 | The maximum number of events to return. |

***

## Function Operation

This function acts as an aggregator to ensure that events are processed in chronological order. This is a requirement for many sequence functions (like `accumulate()`, `neighbor()`, etc.) that depend on the order of events to calculate their results correctly.

***

## Example

### Calculate a Running Average of Field Values

This example uses `head()` to ensure events are properly ordered by time before calculating a running average with the `accumulate()` function.

* **Query Example**
    ```
    head()
    | accumulate(avg(value))
    ```

* **Input Event Data**

| key | value |
| :-- | :-- |
| a | 5 |
| b | 6 |
| c | 1 |
| d | 2 |

* **Step-by-Step**
    1.  `head()`: This first step retrieves and orders the events by time, starting with the oldest ones.
    2.  `accumulate(avg(value))`: The ordered events are then piped to the `accumulate()` function, which computes a running average of the `value` field for each event in the sequence.

* **Result Event Data**
    The output shows a new `_avg` field where each row represents the cumulative average up to that point.

| _avg | key | value |
| :-- | :-- | :-- |
| 5 | a | 5 |
| 5.5 | b | 6 |
| 4 | c | 1 |
| 3.5 | d | 2 |