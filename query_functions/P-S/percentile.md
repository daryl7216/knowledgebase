# percentile()

`percentile()` is an estimation function that estimates percentiles over a given collection of numbers.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `accuracy` | double | optional \<br\> default: `0.01` | Provided as a relative error threshold. Can be between \> 0 and \<1: values closer to 1 means lower accuracy, values closer to 0 means higher accuracy. |
| `as` | string | optional | Prefix of output fields. |
| `field` | string | required | Specifies the field for which to calculate percentiles. The field must contain numbers. |
| `percentiles` | array of numbers | optional \<br\> default: `[50, 75, 99]` | Specifies which percentiles to calculate. |

***

## Function Operation

A percentile is a comparison value between a particular value and the values of the rest of a group. This enables the identification of scores that a particular score surpassed. For example, with a value of 75 ranked in the 85th percentile, it means that the score 75 is higher than 85% of the values of the entire group. This can be used to determine thresholds and limits for triggering events or scoring probabilities and threats.

For example, given the values 12, 25, 50, and 99, the 50th percentile would be any value between 25 and 50; in this case, the `percentile()` function will return 25.79. LogScale's `percentile()` function returns any valid value in order to reduce resource usage and not the mean of valid values, as percentile algorithms in general often return.

LogScale uses an approximate algorithm for percentiles to achieve a good balance of speed, memory usage, and accuracy. The function returns one event with a field for each of the percentiles specified in the `percentiles` parameter. Fields are named by prepending `_` to the values specified in the `percentiles` parameter. For example, the event could contain the fields `_50`, `_75`, and `_99`.

The following conditions apply when using this function:

  * The function only works on non-negative input values.
  * The `accuracy` argument specifies the accuracy of the percentile relative to the number estimated and is intended as a relative error tolerance (lower values imply better accuracy).
      * An `accuracy` of `0.001` specifies the accuracy of the percentile relative to the number estimated (note that specifying `accuracy=0.001` actually implies that the accuracy is `0.999`). The number estimated depends on the `accuracy` argument and the amount of data available. A larger amount of data returns better estimations. For example, with an original value of 1000, the value would be between 999 and 1001 (1000 - 1000/1000 and 1000 + 1000/1000).
      * An `accuracy` of `0.01` means accuracy to 1/100 of the original value. For example, with an original value of 1000, the value would be between 990 and 1010 ((1000 - 1000/100 and 1000 + 1000/100)). With an original value of 500, the value would be between 495 and 505 ((500 - 500/100 and 500 + 500/100)).
  * Any query that does not explicitly set the `accuracy` parameter may see a change in the reported percentile. Specifically, the `percentile()` function may deviate by up to one 100th of the true percentile, meaning that if a given percentile has a true value of 1000, the function may report a percentile in the range of `[990; 1010]`.

Higher accuracy implies higher memory usage. Be careful to choose the accuracy for the kind of precision needed from the expected output value. Lower percentiles are discarded if memory usage becomes too high. If your percentiles seem off, try reducing the accuracy.

***

## Examples

### Calculate the 50th, 75th, 99th and 99.9th percentiles for events with the field responsetime

This is a basic example of calculating multiple percentiles for a given field.

  * **Query Example**
    ```
    percentile(field=responsetime, percentiles=[50,75,99,99.9])
    ```

### In a timechart, calculate percentiles for both of the fields r1 and r2

This example shows how to calculate percentiles for multiple fields within a timechart.

  * **Query Example**
    ```
    timeChart(function=[percentile(field=r1, as=r1), percentile(field=r2, as=r2)])
    ```

### To calculate the median for a given value, use percentile() with percentiles set to 50

This example demonstrates how to use `percentile()` to calculate the median by setting the percentile to 50.

  * **Query Example**

    ```
    percentile(field=allocBytes, percentiles=[50], as=median)
    ```

  * **Step-by-Step**

    1.  `percentile(field=allocBytes, percentiles=[50], as=median)`: Calculates the 50th percentile of the `allocBytes` field and outputs the result in a field named `median_50`.

  * **Summary and Results**
    This query creates the field `median_50` with the 50th percentile value.

---

### Determine a Score Based on Field Value

This example calculates percentiles for `filesize` and then uses conditional logic to assign a symbolic score based on which percentile range the `filesize` falls into.

  * **Query Example**

    ```
    percentile(filesize, percentiles=[40,80], as=score)
    | symbol := if(filesize > score_80, then=":+1:", else if(filesize > score_40, then="so-so", else=":-1:"))
    ```

  * **Step-by-Step**

    1.  `percentile(filesize, percentiles=[40,80], as=score)`: Calculates the 40th and 80th percentiles for the `filesize` field, outputting them as `score_40` and `score_80`.
    2.  `| symbol := if(filesize > score_80, then=":+1:", else if(filesize > score_40, then="so-so", else=":-1:"))`: Compares whether the `filesize` is greater than `score_80`, setting `symbol` to `:+1:`. If not, it checks if `filesize` is greater than `score_40`, setting `symbol` to `so-so`. Otherwise, `symbol` is set to `:-1:`.
    3.  Event Result set.

  * **Summary and Results**
    Using `if()` is the best way to make conditional choices about values. The function has the benefit of being able to be embedded into other statements, unlike `case`.

---

### Show Percentiles Across Multiple Buckets

This example shows how to calculate response time percentiles over time by bucketing events into 1-minute intervals.

  * **Query Example**

    ```
    bucket(span=60sec, function=percentile(field=responsetime, percentiles=[50,75,99,99.9]))
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `bucket(span=60sec, function=percentile(field=responsetime, percentiles=[50, 75, 99, 99.9]))`: Uses a 60-second timespan for each bucket and calculates the specified percentiles for the `responsetime` field within each bucket.
    3.  Event Result set.

  * **Summary and Results**
    The `percentile()` function quantifies values by determining whether the value is larger than a percentage of the overall values. The output provides a powerful view of the relative significance of a value. Combined in this example with `bucket()`, the query will generate buckets of data showing the comparative response time for every 60 seconds.