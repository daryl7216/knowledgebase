# length()

Computes the number of characters in a string field.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
|---|---|---|---|---|
| `as` | string | optional | length | Name of output field. |
| `field` | string | required | | The name of the input field to length. |

***

## Function Operation
The `length()` function computes the number of characters in a string field. This count includes all visible characters, spaces, and punctuation marks in the log entry.

***

## Examples

### 1. Count Characters in Field

* **Query Example**: `length(@rawstring)`
* **Step-by-step**:
    1.  Starting with the source repository events, the query `length(@rawstring)` counts the number of characters in the `@rawstring` field and outputs the result in a field named `length`.
    2.  This count includes all visible characters, spaces, and punctuation marks in the log entry.
    3.  Using the `as` parameter, it is also possible to define another output field, for example, `rawLength`, by adding `length(@rawstring, as="rawLength")`.
* **Summary**: The query counts all characters (visible characters, spaces, and punctuation marks) in a log entry. This is useful for managing and analyzing security logs, ensuring complete data capture for threat detection and incident response.