# array:drop()

The `array:drop()` function takes the name of an array and removes all of its fields from an event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to be dropped. The parameter name `array` can be omitted. |

***

## Function Operation

This function requires the input array to have continuous, sequential indexes that start at 0 (e.g., `field[0]`, `field[1]`, `field[2]`). If the array has any gaps in its indexes (e.g., `field[0]`, `field[2]`), the function will only drop the elements up to the first gap. If an array with the given name doesn't exist, the function does nothing.

***

## Example

### Drop Fields From an Input Array

This example demonstrates how to use `array:drop()` to remove all fields belonging to a specific array.

* **Query Example**
    ```
    array:drop("a[]")
    ```

* **Input Event Data**
    An event contains multiple arrays (`a[]`, `b[]`, `c[]`) and other fields.

| field | value |
| :--- | :--- |
| a[0] | Dog |
| a[1] | Cat |
| a[42] | Horse |
| b[0] | Cat |
| c[0] | Horse |
| animal| cow |

* **Step-by-Step**
    1.  The query starts with the source repository events.
    2.  `array:drop("a[]")` takes the array named `a[]` and drops all of its fields.
    3.  The other arrays (`b[]`, `c[]`) and the `animal` field are not affected.

* **Result Event Data**
    The resulting event no longer contains any fields from the `a[]` array.

| b[0] | c[0] | animal |
| :--- | :--- | :--- |
| Cat | Horse | cow |