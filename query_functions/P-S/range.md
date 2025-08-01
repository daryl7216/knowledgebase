# range()

The `range()` function finds the numeric range between the smallest and largest numbers for the specified field over a set of events. The result is returned in a field named `_range`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `range` | Name of output field. |
| `field` | string | required | Field to extract a number from. |

***

## Function Operation

The `range()` function is an aggregation function that calculates the difference between the maximum and minimum numerical values of a specified field across a set of events. It always returns a single number representing this difference. The result is outputted into a new field, which by default is named `_range`, but can be customized using the `as` parameter. This function is particularly useful for performance analysis and identifying inconsistencies, as a small range indicates consistent performance, while a large range suggests reliability issues. It is commonly used in conjunction with `groupBy()` for comparative analysis across different categories or entities.

***

## Examples

### Find Range Between Smallest And Largest Numbers in Field

This example uses the `range()` function to find the range of values in the `responsetime` field.

  * **Query Example**

    ```
    range(responsetime)
    ```

  * **Input Event Data**

| timestamp | endpoint | responsetime |
| :--- | :--- | :--- |
| 2025-04-30T07:00:00Z | /api/users | 0.125 |
| 2025-04-30T07:00:01Z | /api/login | 2.543 |
| 2025-04-30T07:00:02Z | /api/data | 0.891 |
| 2025-04-30T07:00:03Z | /api/users | 1.234 |
| 2025-04-30T07:00:04Z | /api/search | 3.456 |
| 2025-04-30T07:00:05Z | /api/login | 0.567 |
| 2025-04-30T07:00:06Z | /api/data | 1.789 |
| 2025-04-30T07:00:07Z | /api/users | 0.234 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `range(responsetime)`: Finds the range of the values in the `responsetime` field and returns the result in a field named `_range`. The `range()` function always returns a single number (the difference between maximum and minimum).
    3.  Event Result set.

  * **Summary and Results**
    The query is used to calculate the difference between the highest and lowest values in the field `responsetime` across a set of events. Finding the range of `responsetime` in LogScale is particularly useful for performance analysis to identify performance inconsistencies. A small range indicates consistent performance, while a large range suggests reliability issues.

  * **Result Event Data**
    `_range`
    `3.331`

---

### Find Range of CPU Usage by Host

This example uses the `range()` function with `groupBy()` to find the CPU usage range for each host.

  * **Query Example**

    ```
    groupBy([host], function=range(cpu_usage))
    ```

  * **Input Event Data**

| timestamp | host | cpu\_usage |
| :--- | :--- | :--- |
| 2025-04-30T07:00:00Z | host1.com | 50 |
| 2025-04-30T07:01:00Z | host1.com | 75 |
| 2025-04-30T07:02:00Z | host1.com | 95 |
| 2025-04-30T07:03:00Z | host1.com | 65 |
| 2025-04-30T07:00:00Z | host2.com | 50 |
| 2025-04-30T07:01:00Z | host2.com | 70 |
| 2025-04-30T07:02:00Z | host2.com | 55 |
| 2025-04-30T07:03:00Z | host2.com | 65 |
| 2025-04-30T07:00:00Z | host3.com | 25 |
| 2025-04-30T07:01:00Z | host3.com | 100 |
| 2025-04-30T07:02:00Z | host3.com | 45 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy([host], function=range(cpu_usage))`: Groups events by host name (`host`), then calculates the range (the difference between the highest CPU usage value and the lowest CPU usage value) for each host, returning the results in a new field named `_range`. The `range()` function always returns a single number.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find the CPU usage range by host. The smaller the range (e.g., 0-20), the more stable the system.

  * **Result Event Data**

| host | \_range |
| :--- | :--- |
| host1.com | 45 |
| host2.com | 20 |
| host3.com | 75 |