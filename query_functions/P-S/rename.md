# rename()

This function renames one or more fields.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional | The new name of the field; it is used when a single field name is given in `field`. |
| `field` | string or array, array of arrays of strings | required | The field to rename, if a new field name is given in `as`. From v1.106.0, multiple fields can be given using an array of old/new field name pairs: `[[oldName1,newName1], [oldName2, newName2]]`. |

***

## Function Operation

The `rename()` function allows you to change the name of one or more fields within your event stream.

When a field is renamed to a name that already exists, the existing field and its content are overwritten by the new field and its content. This also occurs when renaming through field aliasing. Old fields are removed from the event stream after renaming, which can add overhead during processing. For more efficiency, especially if you want to retain the old field, copying to a new field using `newfield := oldfield` is more efficient, but it does retain the old field in the event set.

For single field renames, you can specify the `field` parameter (the original name) and the `as` parameter (the new name). For renaming multiple fields in a single operation, you can provide an array of arrays in the `field` parameter, where each inner array contains an `[oldName, newName]` pair.

***

## Examples

### Rename Fields

This example renames multiple fields to more readable names using the `rename()` function.

  * **Query Example**

    ```
    rename(field=[[src_ip, source_address], [dst_ip, destination_address], [src_port, source_port], [dst_port, destination_port]])
    ```

  * **Input Event Data**

    | timestamp | src\_ip | dst\_ip | src\_port | dst\_port | pro |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-04-01T07:00:00Z | 192.168.1.100 | 10.0.0.50 | 52431 | 443 | TCH |
    | 2025-04-01T07:00:01Z | 172.16.0.25 | 8.8.8.8 | 33221 | 53 | UD |
    | 2025-04-01T07-00-037 | 192.168.1.150 | 172.16.0.100 | 49223 | 80 | TCF |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `rename(field=[[src_ip, source_address], [dst_ip, destination_address], [src_port, source_port], [dst_port, destination_port]])`: Renames the fields `src_ip`, `dst_ip`, `src_port`, and `dst_port` to more readable field names. The original field names are replaced with the new field names. Since `field` is the unnamed parameter, the query could also be written as `rename([[src_ip, source_address], [dst_ip, destination_address], [src_port, source_port], [dst_port, destination_port]])`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to rename multiple fields in one single operation. Renaming of fields is used for standardization, normalization, and readability. Normalizing field names across different data sources is, for example, useful for joins. The `rename()` function is often used with the `table()` function. For renaming existing fields in arrays, refer to the documentation on "Rename Existing Fields in Array".

  * **Result Event Data (only showing renamed fields)**

    | destination\_address | destination\_port | source\_address | source\_port |
    | :--- | :--- | :--- | :--- |
    | 10.0.0.50 | 443 | 192.168.1.100 | 52431 |
    | 8.8.8.8 | 53 | 172.16.0.25 | 33221 |
    | 172.16.0.100 | 80 | 192.168.1.150 | 49223 |
    | 192.168.1.1 | 22 | 10.0.0.75 | 55678 |
    | 1.1.1.1 | 53 | 192.168.1.200 | 44556 |
    | 192.168.1.25 | 3389 | 172.16.0.50 | 51234 |
    | 10.0.0.100 | 445 | 192.168.1.75 | 48751 |
    | 172.16.0.75 | 8080 | 10.0.0.25 | 53992 |
    | 8.8.4.4 | 53 | 192.168.1.125 | 35667 |
    | 192.168.1.50 | 21 | 172.16.0.100 | 47891 |

---

### Rename a Single Field - Example 1

This example renames a single field using the `rename()` function with the `as` parameter to define the new name of the field.

  * **Query Example**

    ```
    rename(field=badName, as=goodName)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `rename(field=badName, as=goodName)`: Renames the field `badName` to `goodName`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to quickly rename single fields.

---

### Rename a Single Field - Example 2

This example renames a single field using the `rename()` function with assignment syntax to define the new name of the field.

  * **Query Example**

    ```
    goodName := rename(badName)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `goodName := rename(badName)`: Renames the `badName` field to `goodName` by assigning the new value (variable name) to the field. The value on the right side of the assignment operator is set equal to the value on the left side of it.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to quickly rename single fields.