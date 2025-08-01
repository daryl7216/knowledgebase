# splitString()

The `splitString()` function splits a string using a regular expression into an array of values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_splitstring` | Emit selected attribute using this name. |
| `by` | string | required | String or regular expression to split by. |
| `field` | string | optional \<br\> default: `@rawstring` | Field that needs splitting. |
| `index` | number | optional | Emit only this index after splitting. Can be negative; -1 designates the last element. |

***

## Function Operation

The `splitString()` function takes a string from a specified field and divides it into an array of substrings based on a provided regular expression or string delimiter.

Key aspects of its operation include:

  * The `by` parameter defines the delimiter, which can be a literal string or a regular expression.
  * The `field` parameter specifies the input string to be split; if omitted, `@rawstring` is used by default.
  * The `index` parameter allows you to extract only a specific element from the resulting array by its numerical position (0-based index). A negative index, like `-1`, refers to elements from the end of the array (e.g., `-1` is the last element).
  * The `as` parameter names the output field that will contain the resulting array or the single extracted element if `index` is specified.
  * When splitting by a multi-character separator using a regular expression, ensure the regular expression group is used as the value for `by`. Special characters (including asterisk `*` and pipe `|`) also need to be escaped.

***

## Examples

### Splitting `2007-01-01 test bar` into `part[0]`, `part[1]`, `part[2]`

This example splits a string by whitespace into multiple indexed fields.

  * **Input Event Data**: `@rawstring="2007-01-01 test bar"`

  * **Query Example**:

    ```
    | part := splitString(field=@rawstring, by=" ")
    ```

  * **Result Event Data**: (Implicitly creates fields `part[0]`, `part[1]`, `part[2]`)

---

### Picking out the date part using `splitString()`

This example extracts a specific indexed element (the date) after splitting a string.

  * **Input Event Data**: `@rawstring: 2007-01-01 test bar`

  * **Query Example**:

    ```
    | date: splitString(field=@rawstring, by=" ", index=0)
    ```

  * **Result Event Data**: (Implicitly creates `date` field with value `2007-01-01`)

---

### Splitting a string using `<` `>` or `;` as delimiters

This example splits a string using a regular expression that defines multiple possible delimiters.

  * **Input Event Data**: `@rawstring: <2007-01-01>test; bar`

  * **Query Example**:

    ```
    | part := splitString(field=@rawstring, by="[<>;]")
    ```

  * **Result Event Data**: (Implicitly creates fields `part[0]`, `part[1]`, `part[2]`)

---

### Splitting an event into multiple events by newlines

This example combines `splitString()` with `split()` to transform a multi-line string within an event into separate events, each representing a line.

  * **Query Example**:

    ```
    | splitString(by="\n", as=@rawstring)
    | split(@rawstring)
    ```

  * **Step-by-Step**:

    1.  `splitString(by="\n", as=@rawstring)`: Splits the original `@rawstring` by newline characters (`\n`) and stores the resulting array back into `@rawstring`.
    2.  `| split(@rawstring)`: Takes the array now in `@rawstring` and creates a new event for each element in that array.

---

### Split the value of a string field into individual characters

This example uses a regex to split a string into an array of its individual characters.

  * **Query Example**:

    ```
    characters := splitString(my_field, by="(?<!^)(?=.)")
    ```

  * **Result Event Data**: (Implicitly creates `characters` array with each element being a single character from `my_field`)

---

### Split the value of a string using case-insensitive regex

This example demonstrates splitting a string using a regular expression that performs case-insensitive matching.

  * **Query Example**:
    ```
    characters := splitString(my_field, by="(?i)(e encoded|enc)")
    ```

---

### Split the string using a multi-character separator

This example shows how to split incoming data by a multi-character separator `*|*` and correctly extract the parts.

  * **Input Event Data**: `image.png*|*PNG*|*0755*|*john`

  * **Query Example**:

    ```
    splitString(by="(\\*\\|\\*)")
    ```

  * **Result Event Data**:

| Field | Value |
| :--- | :--- |
| \_splitstring\[0] | image.png |
| \_splitstring\[1] | PNG |
| \_splitstring\[2] | 0755 |
| \_splitstring\[3] | john |

---

### Deduplicate Compound Field Data With `array:union()`, and `split()`

This example combines `splitString()`, `array:filter()`, `array:union()`, and `split()` to deduplicate and normalize user agent data.

  * **Query Example**:

    ```
    splitString(field=userAgent, by="", as=agents)
    array:filter(array="agents[]", function={bname=/\//}, var="bname")
    array:union(array=agents, as=browsers)
    split(browsers)
    ```

  * **Input Event Data**: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome`

  * **Step-by-Step**:

    1.  Starting with the source repository events.

    2.  `splitString(field=userAgent, by="", as=agents)`: Splits the `userAgent` field into individual array entries within the `agents` array for each event.

          * **Result Event Data (after splitString)**:

        | agents\[0] | agents\[1] | agents\[2] | agents\[3] | agents\[4] |
        | :--- | :--- | :--- | :--- | :--- |
        | Mozilla/5.0 | (Macintosh; | Intel | Mac | OS |

    3.  `array:filter(array="agents[]", function={bname=/\//}, var="bname")`: Filters the `agents` array based on a function checking for `/`.

    4.  `array:union(array=agents, as=browsers)`: Aggregates the list of user agents across all events to create a list of unique entries, eliminating duplicates where the value of the user agent is the same.

          * **Result Event Data (after array:union)**:

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

    5.  `split(browsers)`: Splits the `browsers` array into individual events, turning the single event with an array into multiple events, each representing a browser.

          * **Result Event Data (for split)**: (Implicitly, the event after `array:union` as shown above)

        | browsers\[0] | browsers\[1] | browsers\[2] |
        | :--- | :--- | :--- |
        | Gecko/20100101 | Safari/537.36 | AppleWebKit/605.1.15 |

          * **Result Event Data (after split)**:

        | \_index | browsers |
        | :--- | :--- |
        | 0 | Gecko/20100101 |
        | 1 | Safari/537.36 |
        | 2 | AppleWebKit/605.1.15 |

    6.  Event Result set.

  * **Summary and Results**: The resulting output from the query is a list of events with each event containing a matching index and browser. This can be useful if you want to perform further processing on a list of events rather than an array of values.