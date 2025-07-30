# array:sort()

The `array:sort()` function sorts the elements of an array. By default, it sorts elements as case-insensitive strings in ascending order and replaces the input array with the result. You can also sort elements as numbers and specify a different output array.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | array name | required | The array you want to sort. |
| **asArray** | string | optional <br> default: input array | The name of the array where the sorted output will be stored. |
| **order** | string | optional <br> default: `asc` | The sorting order. Values can be `ascending` (`asc`) or `descending` (`desc`). |
| **type** | string | optional <br> default: `string` | The data type to use for sorting. Values can be `string` for alphabetical sorting or `number` for numerical sorting. |

***

## Function Operation

The `array:sort()` function has several key behaviors:
* **Default Sort**: Without specifying `type` or `order`, it performs a case-insensitive, ascending (A-Z) string sort.
* **Numeric Sort with Non-Numbers**: When `type=number`, any elements that are not numbers are sorted separately as strings and placed *after* the sorted numbers, regardless of whether the order is ascending or descending.
* **Overwriting Arrays**: If you use `asArray` to specify an output array that already exists, the function will overwrite its elements. However, if the target array has more elements than the source array, the extra elements at the end of the target array are **not** removed.

***

## Examples

### Sorting Numbers with Non-Numbers Present

This example shows how `array:sort()` handles an array containing both numbers and strings when sorting numerically. The numbers are sorted first, and the strings are sorted alphabetically after the numbers.

* **Query Example**
    ```
    parseJson()
    | array:sort("array[]", type=number)
    ```

* **Input Event Data**
    ```json
    {"array": ["banana", 5, 2, "apple", 10, 41]}
    ```

* **Result Event Data**
    The numbers are sorted in ascending order, followed by the alphabetically sorted strings.

| array[0] | array[1] | array[2] | array[3] | array[4] | array[5] |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 2 | 5 | 10 | 41 | "apple" | "banana" |

---

### Overwriting a Larger Existing Array

This example demonstrates the behavior when sorting an array and outputting the result to a pre-existing, larger array. The sorted elements overwrite the beginning of the target array, but the extra elements in the target array remain untouched.

* **Query Example**
    ```
    parseJson()
    | array:sort("a[]", asArray="b[]")
    ```

* **Input Event Data**
    An event with two arrays, where `b[]` is larger than `a[]`.
    ```json
    {"a": [3, 1, 2], "b": [5, 5, 5, 5]}
    ```

* **Result Event Data**
    The `a[]` array is sorted numerically `[1, 2, 3]` and its elements overwrite the first three elements of `b[]`. The fourth element of `b[]` is not removed.

| a[0] | a[1] | a[2] | b[0] | b[1] | b[2] | b[3] |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 3 | 1 | 2 | 1 | 2 | 3 | 5 |