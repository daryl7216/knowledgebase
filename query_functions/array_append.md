# array:append()

The `array:append()` function adds one or more values to an existing array. If the specified array does not exist, it will create a new one.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to which values will be appended. The parameter name `array` can be omitted. |
| **values** | array of expressions | required | A list of expressions to be added to the array. |

***

## Function Operation

The function appends values to an array. If any of the expressions provided in the `values` parameter don't evaluate to a value, that expression is simply skipped.

A critical requirement is that the target array must have **continuous, sequential indexes**. If there are gaps or missing indexes in the array, `array:append()` will start inserting new values at the first gap. This can lead to existing elements being overwritten.

For example, given an array with elements at indexes `0`, `1`, and `3`, the index `2` is a gap. A query like `array:append("array[]", values=["x", "y", "z"])` would place `"x"` at index `2`, `"y"` at index `3` (overwriting the original value), and `"z"` at index `4`.

***

## Examples

### Create New Array by Appending Expressions

This example demonstrates how to create a new array by appending expressions.

* **Query Example**
    ```
    array:append(array="related.user[]", values=[lower(source.user.name), lower(destination.user.name)])
    ```

* **Input Event Data**
    ```
    source.user.name="user_1" destination.user.name="USER_2"
    ```

* **Step-by-Step**
    1.  The query takes the values from the `source.user.name` and `destination.user.name` fields.
    2.  It applies the `lower()` function to convert both values to lowercase.
    3.  A new array named `related.user[]` is created containing the two lowercase usernames.

* **Result Event Data**

| source.user.name | destination.user.name | related.user[0] | related.user[1] |
| :--- | :--- | :--- | :--- |
| user_1 | USER_2 | user_1 | user_2 |

---

### Split Comma-Separated Strings into a New Array

This example shows how to split a string into an array and then append a new value to it.

* **Query Example**
    ```
    splitString(field=numbers, by=", ", as=numbarr)
    | array:append(array="numbarr[]", values=[4])
    ```

* **Input Event Data**
    ```
    numbers | "1,2,3"
    ```

* **Step-by-Step**
    1.  The `splitString()` function first converts the comma-separated string in the `numbers` field into a new array named `numbarr`.
    2.  The `array:append()` function then adds the number `4` to the end of the `numbarr` array.

* **Result Event Data**

| numbarr[0] | numbarr[1] | numbarr[2] | numbarr[3] |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 3 | 4 |