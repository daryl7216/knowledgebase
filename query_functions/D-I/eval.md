# eval()

[cite_start]The `eval()` function creates or modifies a field by evaluating a given expression. [cite: 3] [cite_start]The result of the expression is stored in a new or existing field. [cite: 3, 4]

***

## Parameters

[cite_start]This function does not take any formal parameters. [cite: 9] [cite_start]Instead, its arguments consist of one or more comma-separated assignment expressions (e.g., `eval(field1 = expression1, field2 = expression2)`). [cite: 7]

***

## Function Operation

[cite_start]The `eval()` function processes expressions from left to right. [cite: 21] [cite_start]It supports basic arithmetic (`+`, `-`, `*`, `/`) and comparison (`==`, `!=`) operators. [cite: 6] [cite_start]An assignment only occurs if the expression can be successfully evaluated. [cite: 25] [cite_start]If a field used in the expression is missing from an event or is not a number, the evaluation fails, and no new field is created. [cite: 26, 27] [cite_start]If the target field already existed, its value will remain unchanged in case of a failed evaluation. [cite: 28]

***

## Example

### Match Field to Timespan

This example demonstrates how to use `eval()` after a `timeChart()` function to normalize aggregated data. It converts a 5-minute event count into an average per-minute count.

* **Query Example**
    ```
    timechart(method, span=5min)
    | eval(_count = _count / 5)
    ```

* **Step-by-Step**
    1.  [cite_start]`timechart(method, span=5min)`: This function first groups events into 5-minute buckets. [cite: 60] [cite_start]By default, it counts the events in each bucket and outputs the result to a field named `_count`. [cite: 61]
    2.  `eval(_count = _count / 5)`: The results from the time chart are then piped to `eval()`. [cite_start]This expression takes the `_count` value for each 5-minute bucket, divides it by 5, and then overwrites the `_count` field with this new per-minute average. [cite: 64]

* **Summary and Results**
    [cite_start]This query is useful for displaying per-minute rates while still benefiting from the improved performance of aggregating data into larger time buckets. [cite: 65] [cite_start]The final chart will show a value that represents the average count per minute, even though the data is bucketed in 5-minute intervals. [cite: 64, 70]