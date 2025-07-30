# array:eval()

The `array:eval()` function evaluates a given function on all values within a specified array. It can overwrite the original array or save the results into a new array if the `asArray` parameter is used. The output array is always compacted, meaning its indices are continuous (0, 1, 2, ...).

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the input array. The parameter name `array` can be omitted. |
| **asArray** | string | optional <br> default: overwrites input array | The name of the output array where the results will be stored. |
| **function** | non-aggregate function | required | The function to apply to each element of the array. |
| **var** | string | optional <br> default: input array name | The name of the variable to be used within the `function`. |

***

## Examples

### Evaluate Function Argument on Values in Array

This example squares the value of each item in an array and stores the result in a new array named `squared[]`.

* **Query Example**
    ```
    array:eval("values[]", asArray="squared[]", var="x", function={squared := x*x})
    ```

* **Input Event Data**

| values[0] | values[1] | values[2] |
| :--- | :--- | :--- |
| 2 | 3 | 4 |

* **Step-by-Step**
    1.  The query starts with events containing the `values[]` array.
    2.  `array:eval()` iterates through each element of `values[]`, assigning the element to the variable `x`.
    3.  The `function` `squared := x*x` is executed for each element, and the result is stored in the new `squared[]` array.

* **Result Event Data**

| field | value |
| :--- | :--- |
| squared[0] | 4 |
| squared[1] | 9 |
| squared[2] | 16 |

---

### Perform Formatting on All Values in an Array

This example converts all string values in the `devices[]` array to uppercase and saves them in a new array called `upperDevices[]`.

* **Query Example**
    ```
    array:eval("devices[]", asArray="upperDevices[]", var=d, function={upperDevices := upper(d)})
    ```

* **Input Event Data**
    The input is a series of JSON events, each with a `devices` array.
    ```json
    {"devices":["Thermostat", "Smart Plug"],"room":"Kitchen"}
    {"devices":["Smart Light", "Thermostat", "Smart Plug"],"room":"Living Room"}
    {"devices":["Smart Light", "Smart Plug"],"room":"Bedroom"}
    ```

* **Step-by-Step**
    1.  The query processes events containing the `devices[]` array.
    2.  `array:eval()` iterates through the array, assigning each device name to the variable `d`.
    3.  The `function` `upperDevices := upper(d)` converts the device name to uppercase.
    4.  The results are stored in the new `upperDevices[]` array, leaving the original `devices[]` array unchanged.

* **Result Event Data**

| devices[] | upperDevices[] |
| :--- | :--- |
| Thermostat | THERMOSTAT |
| Smart Plug | SMART PLUG |
| Smart Light| SMART LIGHT |