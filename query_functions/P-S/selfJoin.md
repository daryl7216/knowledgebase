# selfJoin()

This function is used to collate data from events that share a key. Often the `groupBy()` function can be used for this, but if there are too many keys (defaulting to 100,000) then the result is imprecise since some random subset of keys is left out of the result once the limit is reached.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `collect` | array of strings | optional | Specifies columns to include. |
| `field` | array of strings | required | Specifies which field in the event (log line) that must match the given column value. |
| `limit` | number | optional \<br\> default: `20000` \<br\> minimum: `1` \<br\> maximum: `MAX_STATE_LIMIT` | Specifies the maximum number of rows in the subquery. |
| `postfilter` | boolean | optional \<br\> default: `false` | Re-run the AND of the WHERE clauses after collating results. If all fields needed for satisfying the WHERE clauses are provided as values for `collect`, this will eliminate false positives in the output. |
| `prefilter` | boolean | optional \<br\> default: `false` | Only pass values matching at least one of the WHERE clauses into the embedded `groupBy()`. |
| `select` | array of strings | optional | Specifies columns to include. |
| `where` | `[filter]` | required | The subquery to execute producing the values to join with. |

***

## Function Operation

The `selfJoin()` function is designed to correlate data from events that share a common key, especially in scenarios where the number of unique keys exceeds the `groupBy()` function's default limit (100,000), which can lead to imprecise results by omitting a random subset of keys.

`selfJoin()` operates by taking a `field` (the join key) and a series of tests defined in `where` clauses (as a subquery). It performs a two-phase operation to ensure that only log lines with a join key satisfying all the collective requirements of the `where` clauses are passed on for further processing. This is achieved in a probabilistic fashion using a bloom filter.

The `limit` parameter (default 20,000, max `MAX_STATE_LIMIT`) controls the maximum number of rows in the internal subquery and the number of matching join keys that satisfy the `where` clauses' requirements. The `GroupMaxLimit` dynamic configuration also affects the number of matching join keys, defaulting to 1,000,000.

An important note is that `selfJoin()` is **probabilistic**, and the result can contain false positives, particularly as the number of matches increases. For example, for 25,000 matching IDs, there might be an average of 1.1 false positives.

`selfJoin()` performs two passes over the data and, therefore, cannot run as a truly live query.

The embedded aggregation within `selfJoin()` uses either `collect()` or `selectLast()` on the specified fields, depending on the value of the `select` or `collect` parameters. The `postfilter` parameter, if true, re-runs the AND of the `where` clauses after results are collated, which can eliminate false positives if all necessary fields are included in `collect`. The `prefilter` parameter ensures only values matching at least one `where` clause are passed into the embedded `groupBy()`.

***

## Examples

### Filter and Collect Values in Same Table

This example retrieves all emails sent from one given person to another given person using the `selfJoin()` function.

  * **Query Example**

    ```
    selfJoin(email_id, where=[{from=*peter*}, {to=*anders*}], collect=[from, to])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `selfJoin(email_id, where=[{from=*peter*}, {to=*anders*}], collect=[from, to])`: Finds and collects all the values in the `email_id` field that correspond to mails from "Peter" to "Anders". The `where` clause specifies that `from` field should contain "peter" and `to` field should contain "anders". The `collect` parameter ensures that `from` and `to` fields are included in the output.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to find and collect all emails sent from one given person to another person. In general, the `selfJoin()` function is useful for narrowing down a set of events in a fairly efficient manner, in cases where the total set of events is too voluminous. Note that this query does two passes over the data and, therefore, cannot be used in a live query.