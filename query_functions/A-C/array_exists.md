# array:exists()

The `array:exists()` function filters events by checking if an array contains at least one element that satisfies a specific condition. It is used for more complex scenarios where `array:contains()` is insufficient, such as when comparing array elements to other fields or using query functions in the condition.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to search. The parameter name `array` can be omitted. |
| **condition** | non-aggregate pipeline | required | A pipeline that acts as a condition. If an element passes through the pipeline, the event is included. |
| **var** | string | optional <br> default: input array name | The name of the variable representing an array element within the `condition`. |

***

## Function Operation

This function iterates through each element of the specified `array` and applies the `condition` to it. If any element in the array causes the `condition` pipeline to return true (i.e., the event passes through the filter), the entire event is included in the results. For structured arrays (arrays of objects), the `objectArray:exists()` function must be used instead.

***

## Examples

### Check For Existence of an Element Using Complex Conditions

This example uses a nested `if()` and `in()` function to create a more complex, OR-like filtering condition.

* **Query Example**
    ```
    kvParse()
    | array:exists(
        array="a[]",
        condition=if(in(a, values=[2, 5]), then=true, else=in(a, values=[3, 6]))
      )
    ```

* **Input Event Data**

| a[0] | a[1] |
| :--- | :--- |
| 1 | 2 |
| 1 | 3 |
| 1 | 4 |

* **Step-by-Step**
    1.  `kvParse()`: Parses the raw string data into key-value pairs.
    2.  `array:exists()`: Iterates through the `a[]` array of each event.
    3.  `condition`: For each element in `a[]`, it first checks if the element is `2` or `5`. If it is, the condition is met. If not, it then checks if the element is `3` or `6`.
    4.  The query keeps events where the `a[]` array contains `2`, `3`, `5`, or `6`.

* **Result Event Data**

| a[0] | a[1] |
| :--- | :--- |
| 1 | 2 |
| 1 | 3 |

---

### Check For Existence of an Element Using a Filtering Pipeline

This example demonstrates using a full pipeline within the condition to compare an array element against both a fixed value and another field in the event.

* **Query Example**
    ```
    kvParse()
    | array:exists(
        array="a[]",
        var=x,
        condition={ x=3 OR x=4 | test(x>=b) }
      )
    ```

* **Input Event Data**

| a[0] | a[1] | b |
| :--- | :--- | :-|
| 1 | 2 | 4 |
| 1 | 3 | 4 |
| 1 | 4 | 3 |

* **Step-by-Step**
    1.  `kvParse()`: Parses the raw string data into key-value pairs.
    2.  `array:exists()`: Iterates through the `a[]` array, assigning each element to the variable `x`.
    3.  `condition`: It applies a two-stage filter to `x`:
        * First, it checks if `x` is `3` or `4`.
        * If that passes, it then pipes the event to `test(x>=b)`, which checks if the array element `x` is greater than or equal to the value of the `b` field in the same event.
    4.  The query only keeps events that satisfy the entire pipeline.

* **Result Event Data**

| a[0] | a[1] | b |
| :--- | :--- | :-|
| 1 | 4 | 3 |