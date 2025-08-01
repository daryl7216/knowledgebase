# stripAnsiCodes()

[cite\_start]The `stripAnsiCodes()` function removes ANSI color codes and movement commands. [cite: 2401]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional | [cite\_start]Name of output field, default is to replace contents of input field. [cite: 2402] |
| `field` | string | required \<br\> default: `@rawstring` | [cite\_start]Specifies the field where to remove ANSI escape codes. [cite: 2402] |

***

## Function Operation

[cite\_start]The `stripAnsiCodes()` function is used to clean up log data by removing ANSI escape codes, which are often used for text formatting (like colors) and cursor movement in terminal outputs. [cite: 2401] [cite\_start]By default, this function operates on the `@rawstring` field, but you can specify a different input `field` to clean. [cite: 2402] [cite\_start]The cleaned string can either replace the contents of the input field or be stored in a new output field if the `as` parameter is used. [cite: 2402] This ensures that the extracted data is plain text, making it easier to read and analyze without visual clutter from control characters.

***

## Examples

### Remove the ANSI escape codes from the message field

This example demonstrates how to remove ANSI escape codes from a specific field named `message`.

  * **Query Example**

    ```
    message := "\x1b[93;41mColor"
    stripAnsiCodes(message)
    | @display:= message
    ```

  * **Step-by-Step**

    1.  `message := "\x1b[93;41mColor"`: A field named `message` is created and assigned a string containing an ANSI color code for "Color".
    2.  `stripAnsiCodes(message)`: This function is applied to the `message` field, removing the ANSI escape code (`\x1b[93;41m`).
    3.  `| @display:= message`: The cleaned `message` field is then aliased to `@display` for visualization purposes.

  * **Result Event Data**
    (Implicitly, the `message` field will now contain only "Color" without the ANSI codes, and `@display` will reflect this cleaned value.)

---

### Remove all ANSI escape codes from @rawstring

This example shows how to remove ANSI escape codes from the raw event string (`@rawstring`).

  * **Query Example**

    ```
    stripAnsiCodes()
    ```

  * **Step-by-Step**

    1.  `stripAnsiCodes()`: Since no `field` parameter is specified, the function defaults to processing `@rawstring`. It removes all ANSI escape codes and movement commands found within the `@rawstring` of each event.

  * **Result Event Data**
    (The `@rawstring` field in each event will be updated to contain the content without ANSI codes.)