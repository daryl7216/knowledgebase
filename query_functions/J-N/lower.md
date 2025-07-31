# lower()

The lower() function converts text to lower case letters. It can process text from event fields or other sources. By default, it uses the system locale, but it is possible to specify a different language and locale if needed.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
|---|---|---|---|---|
| `as` | string | optional | lower | The name of the output field. |
| `field` | string | required | | The name of the input field with the value to convert to lower case. |
| `locale` | string | optional | system locale | Locale to use, as ISO-639 language and an optional ISO-3166 country (for example, en or en\_US). |
| `type` | string | optional | | The name of the locale to use as ISO 639 language and an ISO 3166 country. When not specified, it uses the system locale. |

***

## Function Operation
The `lower()` function converts text to lowercase letters. It can process text from event fields or other sources. By default, it uses the system locale, but it is possible to specify a different language and locale if needed. In addition to providing the field of events to change to all lowercase letters, as well as optionally assigning a name to the resulting field, you can specify the country and language so that conversion is done correctly and without odd characters. For the value of `type`, you can specify just the language, or you can refine that choice by including the country (e.g., "en" for English, "en\_UK" for UK English, "en\_US" for US English). Specifying the correct locale is particularly important for languages with non-Latin alphabets, such as Russian with Cyrillic letters.

***

## Examples

### Create New Array by Appending Expressions
This example demonstrates creating a new flat array by appending new expressions using the `array::append()` function in conjunction with `lower()`.

  * **Query**:

    ```
    array::append(array="related.user[]", values=[lower(source.user.name), lower(destination.user.name)])
    ```

  * **Input Event Data**:
    `source.user.name="user_1" destination.user.name="USER_2"`

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  The query `array::append(array="related.user[]", values=[lower(source.user.name), lower(destination.user.name)])` creates a new array `related.user[]` containing information about all user names seen on the event. Notice that the `lower()` function formats the results into lowercase before appending them to the array.
    3.  Event Result set.

  * **Summary and Results**: This query is used to create a new flat array based on values from an array of expressions.

  * **Result Event Data**:

| source.user.name | destination.user.name | related.user[0] | related.user[1] |
|---|---|---|---|
| user\_1 | USER\_2 | user\_1 | user\_2 |

---

### Format a String to Upper Case and Lower Case
This example uses `upper()` and `lower()` functions with `concat()` to concatenate two fields containing error messages, where one field's result is all lowercase letters and the other field's results are all uppercase letters.

  * **Query**:

    ```
    lower(@error_msg[0], as=msg1)
    | upper(@error_msg[1], as=msg2)
    | concat([msg1, msg2], as=test)
    ```

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `lower(@error_msg[0], as=msg1)`: Formats the first element (index 0) of the `@error_msg` array to lowercase and returns the results in a field named `msg1`.
    3.  `| upper(@error_msg[1], as=msg2)`: Formats the second element (index 1) of the `@error_msg` array to uppercase and returns the results in a field named `msg2`.
    4.  `| concat([msg1, msg2], as=test)`: Concatenates (combines) the values in field `msg1` and field `msg2`, and returns the concatenated string in a new field named `test`.
    5.  Event Result set.

  * **Summary and Results**: The query is used to either convert strings to lowercase or uppercase and return the new concatenated strings/results in a new field, in this example, concatenating error messages. The specific labeling of `msg1` and `msg2` is particularly useful when you have more than one field that uses the same query function. By converting fields to consistent cases, it helps standardize data for easier analysis and comparison. The concatenation allows you to combine multiple fields into a single field, which can be useful for creating unique identifiers or grouping related information.

---

### Standardize Values And Combine Into Single Field
This example standardizes values using the `upper()` and `lower()` functions and combines them into a single field with `concat()`.

  * **Query**:

    ```
    lower(#severity, as severity)
    | upper (#category, as category)
    | concat([severity, category], as=test)
    | top(test)
    ```

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `lower(#severity, as severity)`: Converts the values in the `#severity` field to lowercase and returns the results in a field named `severity`.
    3.  `| upper (#category, as category)`: Converts the values in the `#category` field to uppercase and returns the results in a field named `category`.
    4.  `concat([severity, category], as=test)`: Concatenates (combines) the values in field `category` and field `severity`, and returns the concatenated string in a new field named `test`.
    5.  `| top(test)`: Finds the most common values of the field `test`—the top of an ordered list of results—along with their count. The result of the count of their occurrences is displayed in a field named `_count`.
    6.  Event Result set.

  * **Summary and Results**: The query is used to standardize the format of the values in the fields `#category` and `severity` and concatenate the values into a single field, showing which combinations of severity and category are most common in the data. The specific labeling of `category` and `severity` is particularly useful when you have more than one field that uses the same query function. By converting fields to consistent cases, it helps standardize data for easier analysis and comparison. The concatenation allows you to combine multiple fields into a single field, which can be useful for creating unique identifiers or grouping related information. It provides a quick overview of the distribution of events across different severity-category combinations.

  * **Result Event Data**:

| test | \_count |
|---|---|
| infoALERT | 90005 |
| infoFILTERALERT | 36640 |
| errorALERT | 17256 |
| warningGRAPHQL | 14240 |
| warningALERT | 13617 |
| warningSCHEDULEDSEARCH | 11483 |
| infoSCHEDULEDSEARCH | 5917 |
| warningFILTERALERT | 1646 |
| errorFILTERALERT | 1487 |
| infoACTION | 3 |

```
Notice how the value of `#severity` is in lowercase letters, and the value of `#category` is in uppercase letters.
```