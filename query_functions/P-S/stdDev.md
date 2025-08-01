# stdDev()

The `stdDev()` function calculates the standard deviation for a field over a set of events. The result is returned in a field named `stddev`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `stddev` | Name of output field. |
| `field` | string | required | Field to extract a number from and calculate standard deviation over. |

***

## Function Operation

The `stdDev()` function computes the standard deviation of numerical values within a specified field across a set of events. Standard deviation is a measure of how widely values are dispersed from their average value. A lower standard deviation indicates that the data points tend to be close to the mean (average) of the set, while a higher standard deviation indicates that the data points are spread out over a wider range of values.

The function returns a single numerical value representing the calculated standard deviation. This result is outputted into a new field, which by default is named `stddev`, but a custom name can be assigned using the `as` parameter. The unit of measurement for the standard deviation will be the same as the input field. This function is particularly useful in statistical analysis for understanding the variability and distribution of data, such as identifying unusual patterns or establishing normal ranges for metrics.

***

## Examples

### Find the standard deviation of bytes sent in http responses

This is a basic example of using `stdDev()` to calculate the standard deviation of `bytes_sent`.

  * **Query Example**
    ```
    stdDevBytes := stdDev(field=bytes_sent)
    ```

### Calculate Standard Deviation of Bytes Sent

This example uses the `stdDev()` function to calculate how much the number of bytes sent varies from the mean value.

  * **Query Example**

    ```
    stdDevBytes := stdDev(field=bytes_sent)
    ```

  * **Input Event Data**

| @timestamp | endpoint | bytes\_sent | status\_code |
| :--- | :--- | :--- | :--- |
| 1686837825000 | /api/users | 1450 | 200 |
| 1686837826000 | /api/products | 8920 | 200 |
| 1686837827000 | /api/orders | 1670 | 200 |
| 1686837828000 | /api/payment | 12900 | 500 |
| 1686837829000 | /api/users | 1560 | 200 |
| 1686837830000 | /api/items | 780 | 200 |
| 1686837831000 | /api/orders | 9340 | 200 |
| 1686837832000 | /api/checkout | 9230 | 200 |
| 1686837833000 | /api/products | 1340 | 200 |
| 1686837834000 | /api/users | 4450 | 200 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `stdDevBytes := stdDev(field=bytes_sent)`: Calculates the standard deviation of values in the `bytes_sent` field and assigns the result to a new field named `stdDevBytes`. The `stdDev()` function measures how widely the values are dispersed from their average value.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to understand the variability in the size of data being transferred. This is useful for identifying unusual patterns in data transfer sizes, establishing normal ranges for network traffic, or detecting anomalies in data transmission. The result is a single value representing the standard deviation. A higher value indicates greater variation in the data. The unit of measurement is the same as the input field (bytes in this case).

  * **Result Event Data**

| stdDevBytes |
| :--- |
| 4289.32 |