# beta:repeating()

The `beta:repeating()` function marks a live query as "repeating," which means it is implemented by making a series of repeated historical queries.

> **Important**
> This is a beta feature that may not work as expected and could be removed without warning. The `RepeatingQueries` feature must be enabled for this function to be available. If the feature is disabled, any query, alert, or dashboard using this function will fail.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | The name of the output field. |
| **interval** | long | optional <br> default: system-selected | The time interval between successive historical queries, defined using Relative Time Syntax. The parameter name `interval` can be omitted. |

***

## Function Operation

This function allows for the use of functions not normally supported in live queries (such as `selfJoin()` or certain uses of `join()`) by executing the query as a series of historical queries. When the query snapshot cache is enabled, these repeated queries can reuse previous results for better performance. This function has no effect when used in a historical query.

***

## Example

### Repeat a Query Every 5 Minutes

This example demonstrates how to set a live query to repeat every five minutes.

* **Query Example**
    ```
    beta:repeating(5m)
    ```