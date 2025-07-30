# array:length()

The `array:length()` function calculates the number of elements present in an array.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to measure. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_length` | The name of the field that will store the output length. |

***

## Function Operation

This function counts the elements in an array, but with a critical rule: it only counts elements that are in sequential order starting from index 0. The count stops at the first gap or missing index. For example, for an array with elements at `animals[0]`, `animals[1]`, and `animals[3]`, the function will return a length of `2` because it stops at the missing `animals[2]` index.

If the function is applied to a field that is not an array, it will return `0`.

***

## Example

### Count Array Elements After Filtering

This example first filters events to find those where the first function used is `head`, and then it counts the number of functions in the array for those specific events.

* **Query Example**
    ```
    queryParserMetrics.function[0] = "head"
    | array:length("queryParserMetrics.function[]", as="_numberOfFunctions")
    ```

* **Input Event Data**
    An event containing an array of functions.
    ```
    queryParserMetrics.function[0]="head"
    queryParserMetrics.function[1]="bucket"
    queryParserMetrics.function[2]="groupBy"
    ```

* **Step-by-Step**
    1.  The query first filters events, keeping only those where the value of the `queryParserMetrics.function[0]` field is exactly `"head"`.
    2.  For the events that pass the filter, the `array:length()` function is then applied to the `queryParserMetrics.function[]` array.
    3.  It counts the number of elements in that array and places the result in a new field named `_numberOfFunctions`.

* **Result Event Data**
    The output is the total count of functions for an event that started with the `head` function.
    ```
    _numberOfFunctions 3
    ```