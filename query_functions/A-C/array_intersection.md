# array:intersection()

This function determines the set intersection of array values over all input events, computing the values that occur in all events supplied to it.

> **Important**
> This function must be enabled using the `ArrayFunctions` feature flag.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The prefix of the array to process. The parameter name `array` can be omitted. |
| **as** | string | optional <br> default: `_intersection` | The name of the output array that will contain the intersection. |

***

## Function Operation

This function calculates the common values found in the specified array across multiple events. Key behaviors include:
* The order of values in the output array is not defined.
* Empty arrays are ignored by the function.
* If no arrays are found in the input, the output will be empty.

***

## Example

### Find Set Intersection Within an Array

This example finds the unique email addresses within a single array by performing an intersection on its own values. Note that this example seems to misuse the function's primary purpose of finding intersections *across events* and instead uses it to deduplicate a single array.

* **Query Example**
    ```
    array:intersection("mailto[]", as=unique_mails)
    ```

* **Input Event Data**
    An event containing an array of email addresses with duplicates.
    ```
    mailto[0]=foo@example.com
    mailto[1]=bar@example.com
    mailto[2]=bar@example.com
    ```

* **Step-by-Step**
    1.  The query begins with events containing the `mailto[]` array.
    2.  `array:intersection()` processes the `mailto[]` array, finds the common or unique set of element values, and stores them in a new array called `unique_mails`.

* **Result Event Data**
    The output is a new array, `unique_mails`, containing only the unique values from the input array.
    ```
    unique_mails[0]=foo@example.com
    unique_mails[1]=bar@example.com
    ```