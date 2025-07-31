# coalesce()

[cite_start]The `coalesce()` function accepts a list of fields or expressions and returns the first value that is not null or empty. [cite: 458] [cite_start]It is useful for normalizing data where the same type of information might exist in fields with different names. [cite: 459, 473]

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_coalesce` | [cite_start]The name of the field that will contain the selected value. [cite: 462] |
| **expressions** | list of expressions | required | The prioritized list of expressions to select from. [cite_start]The first non-null result from left to right is used. [cite: 462] |
| **ignoreEmpty** | boolean | optional <br> default: `true` | [cite_start]If set to `true`, empty strings are ignored and not selected as a value. [cite: 462] |

***

## Function Operation

[cite_start]The function evaluates a list of expressions in order from left to right and returns the value of the first expression that is not null. [cite: 458] [cite_start]By default, empty strings are also treated as undefined values and are skipped. [cite: 462, 461] [cite_start]This behavior can be changed by setting `ignoreEmpty=false`. [cite: 461] [cite_start]If a field name contains unsupported characters (like a space or a hyphen), it must be wrapped in the `getField()` function to be used correctly within `coalesce()`. [cite: 486, 487]

***

## Example

### Find the First Value in a List of Fields

This example uses `coalesce()` to find a hostname from several possible fields (`host`, `server`, `host[0].name`) and provides a default value (`"example.com"`) if none of them exist. [cite_start]This is a common way to normalize data from different sources. [cite: 473, 482]

* **Query Example**
    ```
    coalesce([host, server, host[0].name, "example.com"])
    ```

* **Input Event Data**

| host | server | host[0].name | machine |
| :--- | :--- | :--- | :--- |
| | 'crowdstrike.com' | | |
| | | 'crowdstrike.com' | |
| | | | 'clienta' |

* **Step-by-Step**
    1.  [cite_start]The query starts with events that may contain different fields for the hostname. [cite: 478]
    2.  [cite_start]`coalesce()` evaluates the fields in order for each event: `host`, `server`, and then `host[0].name`. [cite: 481]
    3.  [cite_start]It returns the value of the first field in that list that is not null or empty. [cite: 458]
    4.  [cite_start]If none of the fields have a value, it returns the string literal `"example.com"` as a default. [cite: 482]
    5.  [cite_start]The result is placed in a new field named `_coalesce`. [cite: 481]

* **Result Event Data**
    The output shows the `_coalesce` field populated with the first available hostname value.

| _coalesce | host | server | host[0].name | machine |
| :--- | :--- | :--- | :--- | :--- |
| crowdstrike.com | <no value> | crowdstrike.com | | |