# unit:convert()

The `unit:convert()` function converts numerical values between different units, such as bytes to megabytes or milliseconds to seconds.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | The name of the output field. Defaults to overwriting the input field. |
| **binary** | boolean | optional <br> default: `false` | If `true`, standard prefixes (K, M, G, etc.) are treated as powers of 1024 (binary) instead of 1000 (decimal). |
| **field** | string | required | The name of the input field to convert. The parameter name `field` can be omitted. |
| **from** | string | optional | If the field contains a unitless number, this specifies the unit to assume for the conversion. |
| **keepUnit**| boolean | optional <br> default: `false` | If `true`, the new unit will be appended to the output value. |
| **to** | string | optional | The unit to convert the value to. If omitted, the function parses the value into its base unit (e.g., "5 MB" becomes `5000000`). |
| **unit** | string | optional | If specified, the field's value must end with this exact unit. This helps resolve ambiguity if a unit name starts with a reserved prefix (e.g., 'h' for hecto- vs. 'hits'). |

***

## Function Operation

This function can parse values that have units attached (e.g., `"5 MB"`) or unitless numbers if a `from` unit is specified. It supports a wide range of SI (decimal, powers of 1000) and binary (powers of 1024) prefixes.
* **Decimal prefixes**: `k`, `M`, `G`, `T`, etc.
* **Binary prefixes**: `Ki`, `Mi`, `Gi`, `Ti`, etc.

The `binary=true` flag is useful when data is labeled with decimal prefixes but represents binary quantities (e.g., a field has the value `"5 kB"` but it should be treated as 5 Kibibytes). The function can also normalize rates (like `hits/min`) to a per-second basis.

***

## Example

### Convert File Size and Transfer Time Units

This example demonstrates how to convert multiple fields to more readable units for analysis. It converts file sizes from bytes to megabytes and transfer times from milliseconds to seconds.

* **Query Example**
    ```
    unit:convert(field=file_size, from="B", to="MB")
    | unit:convert(field=transfer_time, from="ms", to="s")
    | table([file_name, file_size, transfer_time])
    ```

* **Input Event Data**

| file_name | file_size | transfer_time |
| :--- | :--- | :--- |
| doc1.pdf | 1048576 | 3500 |
| img1.jpg | 2097152 | 4200 |
| video1.mp4| 5242880 | 12000 |

* **Step-by-Step**
    1.  The query starts with events containing file size in bytes and transfer time in milliseconds.
    2.  The first `unit:convert()` call reads the `file_size` field, treats the number as Bytes (`from="B"`), and converts it to Megabytes (`to="MB"`).
    3.  The second `unit:convert()` call reads the `transfer_time` field, treats it as milliseconds (`from="ms"`), and converts it to seconds (`to="s"`).
    4.  Finally, `table()` displays the newly converted, human-readable values.

* **Result Event Data**
    The output is a table with standardized units, making it easy to compare the values.

| file_name | file_size | transfer_time |
| :--- | :--- | :--- |
| doc1.pdf | 1.0 | 3.5 |
| img1.jpg | 2.0 | 4.2 |
| video1.mp4| 5.0 | 12.0 |