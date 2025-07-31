# copyEvent()

The `copyEvent()` function creates a duplicate of an event. After this function is used, both the original and the copied event proceed to the next step in the query pipeline. It is most useful within a parsing pipeline.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **type** | string | required | The value to be assigned to the `#type` field for the copied event. The parameter name `type` can be omitted. |

***

## Function Operation

When you call `copyEvent()`, you create a second, identical version of the current event, but with a new `#type` that you specify. This allows you to process the same event in two different ways. For on-premise deployments, if you intend to copy an event to a different repository, the `ALLOW_CHANGE_REPO_ON_EVENTS` environment variable must be set to `true`.

***

## Examples

### Make a Copy of an Event to Track Arrival Time

This example demonstrates how to store an event with both its original timestamp and a separate timestamp based on when it arrived in the system.

* **Query Example**
    ```
    copyEvent("arrivaltime")
    | case { 
        #type=arrivaltime | @timestamp := now(); *;
        * | parseTimestamp(field=ts) 
      }
    ```

* **Step-by-Step**
    1.  `copyEvent("arrivaltime")`: A copy of the current event is created, and its `#type` is set to `arrivaltime`. Now, two events (the original and the copy) are in the pipeline.
    2.  `case`: This statement handles the two events differently based on their `#type`.
    3.  If `#type=arrivaltime`, the function sets the `@timestamp` of the copied event to the current time (`now()`).
    4.  The original event (which does not have `#type=arrivaltime`) has its timestamp parsed from a field named `ts`.

* **Summary and Results**
    The query results in two versions of the event being stored. This allows you to track both when an event originally occurred (original timestamp) and when it was received and processed by the system (arrival time), which is useful for monitoring ingestion delays.

---

### Copy an Event to a Different Repository

This example shows how a single parser can be used to ingest data and then distribute copies of events to different repositories.

* **Query Example**
    ```
    copyEvent("cloned_event")
    | case { 
        #type="cloned_event" | #repo := "target-repo-name"; * }
    ```

* **Step-by-Step**
    1.  `copyEvent("cloned_event")`: A copy of the current event is made, and its `#type` is set to `cloned_event`.
    2.  `case`: The statement checks the `#type` field.
    3.  If the `#type` is `cloned_event`, the function sets the `#repo` field to `"target-repo-name"`, effectively directing this copied event to be stored in the target repository. The original event is not modified and remains in its original repository.

* **Summary and Results**
    This technique is useful for creating centralized parsing logic. You can send logs from a single source to one parser, which can then decide which repositories to send the events to based on their content, allowing for flexible log routing.