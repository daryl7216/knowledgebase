# dropEvent()

The `dropEvent()` function is used to discard events. It has different behaviors depending on whether it's used during the parsing/ingestion phase or in a normal search query.

***

## Parameters

The `dropEvent()` function does not accept any parameters.

***

## Function Operation

The behavior of `dropEvent()` changes based on its context:
* **During Parsing**: When used in a parser, the function permanently drops the event. The event data will not be stored in Falcon LogScale.
* **During a Search**: When used in a normal search query, the function acts as an alias for `false`. It filters the event from the search results but does not delete it from storage.

***

## Examples

### Drop an Event During Parsing

This example shows how to use `dropEvent()` within a parser to prevent events that match a specific condition from being ingested and stored.

* **Query Example**
    ```
    parseJson()
    | case { 
        someField = "some_value" | dropEvent(); 
        * }
    | parseTimestamp(field=@timestamp)
    ```

* **Step-by-Step**
    1.  `parseJson()`: The query first parses incoming JSON data to extract fields.
    2.  `case`: A `case` statement checks the value of `someField`.
    3.  If `someField` equals `"some_value"`, the `dropEvent()` function is called, and the parsing for that event is terminated, preventing it from being stored.
    4.  All other events (matched by `*`) are passed on to the next step, `parseTimestamp()`.

* **Summary and Results**
    This query is used to filter out unwanted data at the point of ingestion. By dropping events with a specific field value, you can reduce storage costs and noise in your repository.

---

### Drop Events Based on Multiple Conditions in a Search

This more complex example demonstrates using `dropEvent()` in a regular search query to filter results based on several different conditions.

* **Query Example**
    ```
    case {
      fielda = "badresult" | dropEvent();
      fieldb = "badresult" | dropEvent();
      wildcard("badip*", field=[fieldc, fieldd]) | dropEvent();
      *
    }
    ```

* **Step-by-Step**
    1.  The query uses a `case` statement to evaluate multiple conditions on the events in the search results.
    2.  The first two conditions check if `fielda` or `fieldb` have the exact value `"badresult"`. If either is true, the event is dropped from the results.
    3.  The third condition uses the `wildcard()` function to check if either `fieldc` or `fieldd` contains a value starting with `"badip"`. If so, the event is dropped.
    4.  The final `*` ensures that any events not matching the drop conditions are kept in the results.

* **Summary and Results**
    This query is used to filter out events from a search result based on multiple, specific criteria. In this context, `dropEvent()` functions as a simple filter (`false`), removing unwanted events from the final view to help narrow down an investigation.