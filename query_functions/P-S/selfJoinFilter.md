# selfJoinFilter()

This function is a filter query that runs in two phases:

1.  Runs a query to determine a set of IDs (specified using the `field` parameter), for which there exists an event with that field ID which satisfy all the `where` clauses. Each `where` clause can be satisfied by distinct events (but they must all have the same ID).
2.  Runs as a filter function that lets all events that have one of the determined IDs pass through. In the secondary run, the events need only match the ID, not any of the `where` clauses; unless `prefilter=true` is set.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | array of strings | required | Specifies which field in the event (log line) is the join key identifier. |
| `prefilter` | boolean | optional \<br\> default: `false` | Defines whether to include events that match the `where` clauses. \<br\>**Values:**\<br\>`false`: Passes all events with matching IDs and includes events that do not match the `where` clauses. \<br\>`true`: Only passes events that directly match `where` clauses. |
| `where` | `[filter]` | required | The subquery to execute producing the values to join with. |

***

## Function Operation

The `selfJoinFilter()` function operates as a two-phase filter query, primarily designed to efficiently narrow down a set of events that share a common key, especially when dealing with a large volume of data where other aggregate functions might hit key limits.

**Phase 1: ID Determination**
In the first phase, `selfJoinFilter()` executes a subquery defined by the `where` parameter. This subquery identifies a set of unique IDs (specified by the `field` parameter, which acts as the join key). An ID is included in this set if there exists at least one event with that ID that satisfies *all* of the conditions specified in the `where` clauses. Importantly, each `where` clause can be satisfied by distinct events, as long as those events share the same common ID. This process uses a compact and fast, but imprecise, summary of relevant keys, often employing a probabilistic bloom filter.

**Phase 2: Event Filtering**
In the second phase, `selfJoinFilter()` acts as a filter on the main event stream. It allows all events that possess one of the IDs determined in Phase 1 to pass through. By default, with `prefilter=false`, these events are passed regardless of whether they themselves directly satisfy the original `where` clauses. However, if `prefilter=true` is set, only events that *directly* match one of the `where` clauses are passed through.

**Important Considerations:**

  * **Probabilistic Nature**: `selfJoinFilter()` is probabilistic, meaning the result can contain false positives. The documentation provides a table illustrating the false positive rate based on the number of matches. For example, with 25,000 matching IDs, there could be an average of 1.1 false positives.
  * **Two-Pass Operation**: This function performs two passes over the data, which means it cannot be used in a truly live query unless combined with `beta:repeating()`.
  * **Multiple Fields in `field`**: If multiple fields are specified in the `field` parameter, all of them must exist in an event for that event to be considered valid for `selfJoinFilter()`.

A typical workflow when using this function involves:

1.  Filtering the initial event set to define a base set of events.
2.  Using `selfJoinFilter()` to identify events sharing common keys based on specified conditions.
3.  Correlating the content of the filtered events, often by using `groupBy()` for aggregation.
4.  (Optionally) Filtering the results further to exclude any correlated data not required in the final output.

***

## Examples

### Compare and Filter Values in Same Table

This example retrieves all emails with attachments sent from one given person to another given person using the `selfJoinFilter()` function, matching only by the email ID.

  * **Query Example**

    ```
    selfJoinFilter(field=email_id, where=[{from=peter}, {to=paul}])
    attachment=*
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selfJoinFilter(field=email_id, where=[{from=peter}, {to=paul}])`: This is the first phase. It finds all `email_id` values for which there exists an event where the `from` field is "peter" AND an event where the `to` field is "paul" (both sharing the same `email_id`). The subsequent events will then be filtered based on these `email_id`s (Phase 2).
    3.  `| attachment=*`: This is applied to the events passed from `selfJoinFilter()`. It further filters these events, retaining only those that also have an `attachment` field present (i.e., `attachment` exists and has any value).
    4.  Event Result set.

  * **Summary and Results**
    The query is used to find all emails with attachments sent from one given person to another person. In general, the `selfJoinFilter()` function is useful for narrowing down a set of events in a fairly efficient manner, especially in cases where the total set of events is too voluminous. Note that this query performs two passes over the data and, therefore, cannot be used in a live query.