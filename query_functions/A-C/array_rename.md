# array:rename()

The `array:rename()` function takes an array and renames all of its fields.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **array** | string | required | The name of the array to rename. The parameter name `array` can be omitted. |
| **asArray** | string | required | The new name for the array. |

***

## Function Operation

This function requires that the input array has continuous, sequential indexes starting from `[0]`. If the function encounters a gap in the indexes (e.g., elements exist at `[0]`, `[1]`, and `[3]` but not `[2]`), it will only rename the elements up to that first gap.

If the target array doesn't exist, or if the new name is the same as the old name, the function will do nothing. If an array with the new name already exists, it will be overwritten.

***

## Example

### Rename Existing Fields in an Array

This example renames an array from `mail[]` to `user.email[]`, which is a common task when normalizing data to a standard schema like the Elastic Common Schema (ECS).

* **Query Example**
    ```
    array:rename(array="mail[]", asArray="user.email[]")
    ```

* **Input Event Data**
    An event with an array named `mail[]`.
    ```
    mail[0]='user0@example.com'
    mail[1]='user1@example.com'
    mail[2]='user2@example.com'
    ```

* **Step-by-Step**
    1.  The query starts with events containing the `mail[]` array.
    2.  `array:rename()` takes all fields belonging to the `mail[]` array and renames them to `user.email[]`. The original `mail[]` fields are removed.

* **Result Event Data**
    The event now has the `user.email[]` array instead of the `mail[]` array.
    ```
    user.email[0]=user0@example.com
    user.email[1]=user1@example.com
    user.email[2]=user2@example.com
    ```