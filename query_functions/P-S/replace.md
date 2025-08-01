# replace()

The `replace()` function replaces each substring of the specified field's value that matches the given regular expression with the given replacement. LogScale uses JitRex, which closely follows the syntax of re2j regular expressions, which has a syntax very close to Java's regular expressions.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: input field | Specifies the field to store the replaced string as. |
| `field` | string | optional \<br\> default: `@rawstring` | Specifies the field to run the replacement on. |
| `flags` | string | optional \<br\> default: `m` | Specifies other regex flags. \<br\>**Values:**\<br\>`d`: Period (`.`) includes newline characters. \<br\>`i`: Ignore case for matched values. \<br\>`m`: Multi-line parsing of regular expressions. |
| `regex` | string | required | The regular expression to match. |
| `replacement` | string | optional | The string to substitute for each match (same as `with`). |
| `with` | string | optional | The string to substitute for each match. |

***

## Function Operation

The `replace()` function is used to find and substitute substrings within a specified field's value. It operates by matching parts of the string against a provided regular expression (`regex`) and then replacing those matched parts with a `replacement` string (or `with` string, which is an alias). The function uses JitRex regular expressions, which are similar to Java's regex syntax.

Key aspects of its operation include:

  * The `field` parameter specifies which field to apply the replacement to; by default, it operates on `@rawstring`.
  * The `as` parameter dictates the name of the output field where the modified string will be stored. If not specified, the original `field` will be overwritten.
  * The `flags` parameter allows for modifying the regex behavior, such as `d` (dotall mode, where `.` matches newlines), `i` (case-insensitive matching), and `m` (multi-line mode).

This function is highly versatile for string manipulation and transformation tasks, such as correcting spelling mistakes, truncating strings, or extracting specific patterns and reformatting them.

***

## Examples

### Get Integer Part of Number

This example uses the `regex()` function (mentioned as an alternative in the `replace()` documentation) and regex capturing groups to extract the integer part of a number. The documentation also provides an alternative using `replace()` which is more relevant to this function's scope.

  * **Query Example (using `replace()` as suggested alternative)**

    ```
    replace("(\\d+)\\..*", with="$1", field=a, as=b)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace("(\\d+)\\..*", with="$1", field=a, as=b)`: This query uses `replace()` to capture digits before a decimal point in an unnamed group (`(\d+)`) and replaces the entire matched string with the content of this captured group (`$1`). The result is stored in a new field named `b`, leaving the original field `a` unchanged.

  * **Summary and Results**
    This query, using `replace()`, is particularly suited for string manipulation and transformation. It captures the integer part of a number from a field and stores it in a new field.

---

### Replace Word or Substring With Another

This example uses the `replace()` function with a regular expression to correct a spelling mistake.

  * **Query Example**

    ```
    replace(regex=propperties, with properties)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace(regex=propperties, with properties)`: Replaces all occurrences of the word "propperties" with "properties" in the target field.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to correct spelling mistakes in an event set. Changing words or other substrings like this with a regular expression is useful in situations where quick modifications of field values are needed.

---

### Truncate a String or Message

This example uses the `replace()` function together with regex capturing groups to truncate a string or message to exactly 100 characters.

  * **Query Example**

    ```
    replace("^(.{100}).*", with="$1", field=message, as="truncated_message")
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `replace("^(.{100}).*", with="$1", field=message, as="truncated_message")`: This captures a group that matches exactly 100 characters of any type starting from the beginning of the line (`^(.{100})`). The `.*` matches any remaining characters. `with="$1"` means that the entire matched string (the whole original message) is replaced with the content of the first capturing group (the first 100 characters). The truncated version is returned in a new field named `truncated_message`. The original `message` field remains unchanged.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to truncate strings. Truncation can be beneficial for speeding up download times and completing searches faster. In file systems, truncation reduces file size by removing data from the end, useful for reclaiming storage space or managing log files. Additionally, it can broaden search results by allowing searches for variations of words (e.g., via wildcards on truncated terms) and can shorten numerical digits past a certain point.