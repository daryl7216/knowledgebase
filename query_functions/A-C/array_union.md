# array:union()

[cite_start]The `array:union()` function determines the set union of array values over all input events[cite: 6]. [cite_start]It gathers all unique values from a specified array across all events into a single output array[cite: 7].

> **Important**
> [cite_start]This function is considered experimental and should not be used in production[cite: 4]. [cite_start]It must be enabled using the `ArrayFunctions` feature flag[cite: 5].

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | [cite_start]The prefix of the array to process[cite: 9]. |
| **as** | string | optional <br> default: `_union` | [cite_start]The name of the output array[cite: 9]. |

***

## Function Operation

[cite_start]This function combines all values from the specified array across multiple events and removes duplicates to produce a single array containing only unique values[cite: 6, 70]. [cite_start]The order of the values in the final output array is not defined[cite: 8]. [cite_start]If no arrays are found in the input events, the output will be an empty array[cite: 8].

***

## Example

### Deduplicate Compound Field Data

This complex example demonstrates how `array:union()` can be used within a larger pipeline to parse, filter, and deduplicate information from a compound field like a `userAgent` string.

* **Query Example**
    ```
    splitString(field=userAgent, by=" ", as=agents)
    | array:filter(array="agents[]", function={bname=/\//}, var="bname")
    | array:union(array=agents, as=browsers)
    | split(browsers)
    ```

* **Input Event Data**
    An event with a single `userAgent` field containing a long string.
    ```
    Mozilla/5.0 (Macintosh Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko)
    ```

* **Step-by-Step**
    1.  [cite_start]`splitString()`: The `userAgent` string is first split by spaces into a new array called `agents`[cite: 34].
    2.  [cite_start]`array:filter()`: This function filters the `agents` array to keep only the elements that contain a forward slash (`/`), which are typically the browser name/version pairs[cite: 41].
    3.  [cite_start]`array:union()`: This aggregates the filtered `agents` arrays from all events into a single array named `browsers`, which eliminates any duplicate entries[cite: 44, 45].
    4.  [cite_start]`split()`: Finally, the `browsers` array is split into individual events, with each event containing one unique browser entry[cite: 52].

* **Result Event Data**
    The final output is a list of individual events, each containing a unique browser or toolkit string.

| _index | row[1] |
| :--- | :--- |
| 0 | Gecko/20100101 |
| 1 | Safari/537.36 |
| 2 | AppleWebKit/605.1.15 |