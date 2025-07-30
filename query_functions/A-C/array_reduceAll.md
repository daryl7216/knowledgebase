# array:reduceAll()

The `array:reduceAll()` function computes a single aggregate value from all elements of a specified array across all input events.

***

## Parameters

| Parameter | Type | Required/Default |
| :--- | :--- | :--- |
| **array** | string | required |
| **function**| array of aggregate functions | required |
| **var** | string | required |

***

## Function Operation

This function applies an aggregate function (like `avg()`, `count()`, `sum()`, or `max()`) to every element within a specified array, across all events in the search result. It effectively treats all the values from all the arrays as one large collection and then performs the calculation. This is a more efficient method than using `split()` to create individual events for each array element before aggregating.

For example, if you have three events each with an array `a[]`:
* Event 1: `[1, 4, 2]`
* Event 2: `[3, 5, 2]`
* Event 3: `[5, 2, 3]`

The function would operate on the combined set of values: `{1, 4, 2, 3, 5, 2, 5, 2, 3}`.

***

## Example

### Compute an Aggregated Value of an Array on All Events

This example uses the `max()` aggregate function to find the highest value from all elements across all `values[]` arrays in a set of events.

* **Query Example**
    ```
    array:reduceAll("values[]", var=x, function=max(x))
    ```

* **Input Event Data**
    Consider multiple events each containing a `values[]` array.
    * Event 1: `values[] = [10, 25, 15]`
    * Event 2: `values[] = [5, 30, 20]`
    * Event 3: `values[] = [40, 12, 8]`

* **Step-by-Step**
    1.  The query begins with a set of events, each containing a `values[]` array.
    2.  `array:reduceAll()` gathers all elements from every `values[]` array across all events into a single conceptual list: `{10, 25, 15, 5, 30, 20, 40, 12, 8}`.
    3.  The `max()` function is applied to this combined list to find the highest value.

* **Result Event Data**
    The output is a single event with a single field containing the maximum value found.
    ```
    _max=40
    ```