# array:filter()

The `array:filter()` function filters an array's elements using a provided function, creating a new array containing only the elements that meet the specified condition. The original order of the elements is maintained in the output.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The input array to be filtered. The parameter name `array` can be omitted. |
| **asArray** | string | optional <br> default: input array | The name of the output array where the filtered results will be stored. |
| **function**| non-aggregate function | required | The function to use for filtering the elements in the array. |
| **var** | string | optional <br> default: input array name | The name of the variable representing an array element to be used within the `function`. |

***

## Examples

### Deduplicate Compound Field Data

This complex example uses `array:filter()` as part of a larger pipeline to parse, filter, and deduplicate browser user agent strings.

* **Query Example**
    ```
    splitString(field=userAgent, by=" ", as=agents)
    | array:filter(array="agents[]", function={bname=/\//}, var="bname")
    | array:union(array=agents, as=browsers)
    | split(browsers)
    ```

* **Input Event Data**
    A single field `userAgent` contains a long string of information.
    ```
    Mozilla/5.0 (Macintosh Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko)
    ```

* **Step-by-Step**
    1.  `splitString()`: The `userAgent` string is split by spaces into a new array called `agents`.
    2.  `array:filter()`: This function filters the `agents` array. The condition `{bname=/\//}` keeps only the elements that contain a forward slash (`/`), which are typically the browser name/version pairs.
    3.  `array:union()`: This aggregates the filtered `agents` arrays from all events into a single array named `browsers`, removing any duplicate entries.
    4.  `split()`: The final `browsers` array is split into individual events, with each event containing one unique browser entry.

* **Result Event Data**
    The final output is a list of individual events, each containing a unique browser/toolkit string.

| _index | row[1] |
| :--- | :--- |
| 0 | Gecko/20100101 |
| 1 | Safari/537.36 |
| 2 | AppleWebKit/605.1.15|

---

### Filter an Array on a Given Condition

This example demonstrates the core use of `array:filter()` to select elements from an array that match a specific pattern.

* **Query Example**
    ```
    array:filter(array="mailto[]", var="addr", function={addr="ba*@example.com"}, asArray="out[]")
    ```

* **Input Event Data**
    An event containing an array of email addresses.
    ```
    mailto[0]=foo@example.com
    mailto[1]=bar@example.com
    mailto[2]=baz@example.com
    ```

* **Step-by-Step**
    1.  The query starts with an event that has a `mailto[]` array.
    2.  `array:filter()` iterates through the `mailto[]` array. Each element is assigned to the variable `addr`.
    3.  The `function` `{addr="ba*@example.com"}` checks if the `addr` variable starts with "ba".
    4.  Elements that match the condition are placed into a new array called `out[]`.

* **Result Event Data**
    The new `out[]` array contains only the emails that matched the filter.
    ```
    out[0]=bar@example.com
    out[1]=baz@example.com
    ```