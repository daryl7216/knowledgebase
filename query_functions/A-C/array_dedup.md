# array:dedup()

The `array:dedup()` function removes duplicate values from an array. The order of the first occurrence of each unique element is kept.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | array | required | The array from which to remove duplicate elements. The parameter name `array` can be omitted. |
| **asArray** | string | optional <br> default: input array | Specifies where to output the deduplicated array. |

***

## Function Operation

This function processes an array and removes any duplicate entries, ensuring that every element in the resulting array is unique. The original order is preserved based on the first time each element appeared in the input array.

***

## Example

### Deduplicate Values in Array

This example shows how to remove duplicate email addresses from an array named `emails[]`.

* **Query Example**
    ```
    parseJson()
    | array:dedup("emails[]")
    ```

* **Input Event Data**
    ```json
    {"emails": ["john@mail.com", "admin@mail.com", "jane@mail.com", "admin@mail.com"]}
    ```

* **Step-by-Step**
    1.  `parseJson()`: This function parses the raw event data to identify and create fields from the JSON structure.
    2.  `array:dedup("emails[]")`: This function then takes the `emails[]` array and removes any duplicate values.

* **Result Event Data**
    The output array contains only the unique values from the original array, with the duplicate `"admin@mail.com"` removed.

| emails[0] | emails[1] | emails[2] |
| :--- | :--- | :--- |
| john@mail.com | admin@mail.com | jane@mail.com |
