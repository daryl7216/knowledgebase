# concatArray()

The `concatArray()` function concatenates the values of all fields that share the same base name and have a sequential array suffix (e.g., `field[0]`, `field[1]`, `field[2]`) into a single string in a new field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_concatArray` | The name of the output field. |
| **field** | string | required | The base name for the array fields to concatenate. The parameter name `field` can be omitted. |
| **from** | number | optional <br> default: 0 | The first array index to include in the concatenation. |
| **prefix** | string | optional | A prefix to add to the beginning of the generated string. |
| **separator**| string | optional | The separator to place between the values. |
| **suffix** | string | optional | A suffix to append to the end of the generated string. |
| **to** | number | optional | The last array index to include. If omitted, it includes all sequential fields from the `from` index. |

***

## Function Operation

This function selects all array fields beginning with the specified `field` name and a sequential index. It's crucial to note that the concatenation will **stop** at the first missing index. For example, if fields `email[0]`, `email[1]`, and `email[3]` exist, only the values from `email[0]` and `email[1]` will be concatenated. If the starting index (e.g., `email[0]`) is missing, no output field is created.

***

## Examples

### Concatenate Object Arrays into a Single String

This example first uses `objectArray:eval()` to pull a specific value (`bar`) from each object in an array of objects into a new, simple array. Then, it uses `concatArray()` to join the elements of that new array into a single string.

* **Query Example**
    ```
    "a[0].foo" := "a"
    | "a[0].bar" := "b"
    | "a[1].foo" := "c"
    | "a[1].bar" := "d"
    | objectArray:eval(array="a[]", asArray="output[]", var="x", function={output := x.bar})
    | concatArray("output")
    ```

* **Input Event Data**
    ```
    a[0].foo: a
    a[0].bar: b
    a[1].foo: c
    a[1].bar: d
    ```

* **Step-by-Step**
    1.  The query first establishes an array of objects, `a[]`.
    2.  `objectArray:eval()` iterates over `a[]` and extracts the value of the `bar` key from each object, creating a new flat array called `output[]` with the values `[b, d]`.
    3.  `concatArray("output")` then takes the `output[]` array and concatenates its elements (`b` and `d`) into a single string.

* **Result Event Data**
    The final output includes the new `_concatArray` field with the value `"bd"`.

| a[0].foo | a[0].bar | a[1].foo | a[1].bar | output[0] | output[1] | _concatArray |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| a | b | c | d | b | d | bd |

---

### Concatenate Values with a Prefix, Suffix, and Separator

This example demonstrates how to format the concatenated string by adding a prefix `[` and a suffix `]` and using a comma as the separator, effectively creating a string that looks like an array.

* **Query Example**
    ```
    concatArray(server, prefix="[", separator=",", suffix="]")
    ```

* **Input Event Data**
    ```
    server[0] := "dopey"
    server[1] := "sleepy"
    server[2] := "doc"
    server[3] := "happy"
    server[4] := "sneezy"
    ```

* **Step-by-Step**
    1.  The query starts with an event containing the `server[]` array fields.
    2.  `concatArray()` takes the values from `server[0]` through `server[4]`.
    3.  It prepends the `prefix` `[`, joins the elements with the `,` `separator`, and appends the `suffix`]`.

* **Result Event Data**
    The output is a single field `_concatArray` containing the formatted string.
    ```
    _concatArray="[dopey,sleepy,doc,happy,sneezy]"
    ```