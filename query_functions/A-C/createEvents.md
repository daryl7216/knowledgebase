# createEvents()

The `createEvents()` function generates temporary events from a list of raw strings provided in the query. It is an aggregator function, which means it discards all incoming events and outputs only the newly generated ones. This is ideal for creating sample data for testing or troubleshooting queries without needing actual data.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **rawstring** | string | required | A specification of the events to create. Each event is provided as a `@rawstring`. The parameter name `rawstring` can be omitted. |

***

## Function Operation

This function takes a list of strings and creates a new temporary event for each string. The content of each string becomes the `@rawstring` for the new event. Initially, these events have no extracted fields, so `createEvents()` is almost always piped to a parsing function like `kvParse()` or `parseJson()` to structure the raw string data into fields.

***

## Example

### Generate a Temporary Event to Test Bit Flag Extraction

This is a complex example that shows how `createEvents()` can be used to generate a test event to validate a more complex parsing and extraction query involving bit flags.

* **Query Example**
    ```
    createEvents(["flags=4"])
    | kvParse()
    | bitfield:extractFlags(
        field=flags,
        output=[
            [1, ErrorFlag],
            [2, WarningFlag]
        ]
      )
    ```

* **Step-by-Step**
    1.  `createEvents(["flags=4"])`: A single temporary event is created with the raw string `"flags=4"`.
    2.  `kvParse()`: This parser processes the `@rawstring` of the new event, identifies the key-value pair, and creates a field named `flags` with the integer value `4`.
    3.  `bitfield:extractFlags()`: This function then operates on the newly created `flags` field. It converts the value `4` to its binary representation (`100`) and extracts the state of the specified bits:
        * It checks bit 1 (the "2s" place) and creates a field `ErrorFlag` set to `false`.
        * It checks bit 2 (the "4s" place) and creates a field `WarningFlag` set to `true`.

* **Result Event Data**
    The final output is a single event with two new boolean fields, `ErrorFlag` and `WarningFlag`, representing the decoded bitmask.

| ErrorFlag | WarningFlag |
| :--- | :--- |
| false | true |