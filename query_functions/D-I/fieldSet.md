# fieldset()

The `fieldset()` function retrieves and displays a list of all available fields within the current event set.

***

## Parameters

The `fieldset()` function does not take any parameters.

***

## Function Operation

This function is useful for quickly getting an overview of the data schema, especially when you are new to a dataset. It returns a list of all field names present in the events at that stage of the query. The list is context-specific; if you run `fieldset()` after an aggregation function like `groupBy()`, it will show the fields present in the aggregated results, not the original events.

***

## Example

### Request a List of Fields After Aggregation

This example demonstrates how the output of `fieldset()` changes when used after an aggregation. It first groups events and then lists the fields available in the aggregated result.

* **Query Example**
    ```
    groupBy([#type, @host])
    | fieldset()
    ```

* **Step-by-Step**
    1.  `groupBy([#type, @host])`: This function first groups all events by the `#type` and `@host` fields. By default, it also creates a `_count` field that contains the number of events in each group. The original fields are discarded, and only `#type`, `@host`, and `_count` remain.
    2.  `fieldset()`: This function is then called on the aggregated results. It lists the fields that are now present: `_count`, `@host`, and `#type`.

* **Result Event Data**
    The output is a list of the fields that exist after the `groupBy` operation, not the full list of fields from the original events.
    ```
    _count
    @host
    #type
    ```