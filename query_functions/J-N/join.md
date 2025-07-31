# join()

The `join()` function in LogScale is used to combine two LogScale searches by matching results based on specified keys or fields.

***

## Parameters

| Parameter | Type | Required | Default Value | Description |
| :-------- | :------------- | :-------- | :------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `end` | string | optional | End of main query | Specifies either the timestamp relative to the main query's end (for example, end 2h will be two hours before the end of the main query) or an absolute timestamp in milliseconds since UTC. |
| `field` | array of strings | required | (none) | Specifies which field in the event (log line) must match the given column value. |
| `include` | array of strings | optional | none | Specifies columns to include from the subquery. For events in the subquery that do not have one or more of the named include fields, the output will be the empty string. |
| `key` | array of strings | optional | Value of the field parameter | Specifies which fields of the subquery to join on. Defaults to the value of the `field` parameter. |
| `limit` | number | optional | 100000 | Specifies the maximum number of rows in the subquery. Minimum: 1, Maximum: 200000. |
| `live` | boolean | optional | Same as main query | Controls if the subquery runs as live or static query. |
| `max` | integer | optional | 1 | Maximum number of events found in subquery if several share join key. |
| `mode` | string | optional | inner | Specifies the mode (inner or left) of the join. `inner`: Perform an inner join; return only results that match in both queries. `left`: Perform a left join; all values from the parent query are included, matched to any corresponding events in subquery. |
| `query` | function | required | (none) | The subquery to execute producing the values to join with. The parameter name `query` can be omitted. |
| `repo` | string | optional | Repo of main query | Specify which view/repo in which to perform the subquery. |
| `start` | string | optional | Start of main query | Specifies either the timestamp relative to the main query's end (for example, start 2h will be two hours before the end of the main query) or an absolute timestamp in milliseconds since UTC. |
| `view` | string | optional | View of main query | Specify which view/repo in which to perform the subquery. |

***

## Function Operation

The `join()` function performs two passes over the data: one for the primary query and one for the subquery. By default, in `mode=inner`, it acts as a filter, allowing events through only if they match on the join keys. If `mode=left` is specified, events that do not match the join keys will also be included. The `include` parameter extracts specified fields from the subquery's result and adds them to matching events. The `max=N` parameter controls how many rows/events are picked up in the subquery if multiple events share the same join key. Parameters like `start`, `end`, `view`, and `live` can be used to control the subquery's time interval, repository/view, and live query status, inheriting defaults from the primary query. There is also a `limit` parameter to specify the maximum size of the inner query's result set.

* **Warning:** The `join()` function should not be used as part of a live query because the two passes over the data (primary query and subquery) can be executed on different data sets, potentially leading to inconsistent results. When used in a live query, the query will be run in a repeated mode, in which the server chooses the repetition interval based on the resources used by the function. This can impact the liveness of the query, in that long-running repeated queries can be throttled, and thus be less live than expected. LogScale recommends using `defineTable()` with `match()` instead of negated join for filtering items not part of a data set.

***

## Examples

### 1\. Filter For Items Not Part of Data Set Using `join()`

This example uses `join()` with a negation to find all session IDs from data set A that are not present in data set B.

* **Query Example:**

```
#repo=A session_id=*
| !join(query={#repo=B session_id=*}, field=session_id, key=session_id)
```

* **Input Event Data:**

* **Repository A:**

| timestamp | session\_id | user\_name | action | status |
| :------------------ | :---------- | :---------- | :------- | :------ |
| 2025-04-01T07:00:00Z | 123456 | john.doe | login | success |
| 2025-04-01T07:05:00Z | 123457 | jane.smith | download | success |
| 2025-04-01T07:10:00Z | 123458 | mike.jones | upload | failed |
| 2025-04-01T07:15:00Z | 123459 | sara.wilson | login | success |
| 2025-04-01T07:20:00Z | 123460 | bob.brown | logout | success |

* **Repository B:**

| timestamp | session\_id | user\_name | action | status |
| :------------------ | :---------- | :--------- | :------- | :------ |
| 2025-04-01T07:00:00Z | 123456 | john.doe | login | success |
| 2025-04-01T07:05:00Z | 123457 | jane.smith | download | success |
| 2025-04-01T07:20:00Z | 123460 | bob.brown | logout | success |

* **Step-by-Step:**

1.  **Starting with the source repository events.**
2.  **`#repo=A session_id=*`**: This part filters for all events from repository A that have a `session_id` field.
3.  **`| !join(query={#repo=B session_id=*}, field=session_id, key=session_id)`**: This performs a negated join with repository B. It returns sessions that exist in repository A but *not* in repository B. The negation operator `!` makes this an anti-join operation.

* **Result Event Data:**

| timestamp | session\_id | user\_name | action | status |
| :------------------ | :---------- | :---------- | :----- | :------ |
| 2025-04-01T07:10:00Z | 123458 | mike.jones | upload | failed |
| 2025-04-01T07:15:00Z | 123459 | sara.wilson | login | success |
---
#### 2\. Preview Content in a Lookup File With `readFile()` and Filter With `join()`

This example uses `readFile()` to look up a `host_names.csv` file and then filters for host names that are not sending any logs.

* **Query Example:**

```
readFile("host_names.csv")
| !join(query={groupBy(host_name)}, field=host_name, key=host_name, include=[host_name, id])
```

* **Input Event Data:**

`host_names.csv` content:

```
host_name, host_id
DESKTOP-VSKPBK8, 1
FINANCE, 2
homer-xubuntu, 3
logger, 4
DESKTOP-1, 5
DESKTOP-2, 6
DESKTOP-3, 7
```

* **Step-by-Step:**

1.  **Starting with the source repository events.**
2.  **`readFile("host_names.csv")`**: This part displays the content of the `.csv` file.
3.  **`!join(query={groupBy(host_name)}, field=host_name, key=host_name, include=[host_name, id])`**: This filters for host names that do not send any logs.

* **Result Event Data:**

| host\_id | host\_name |
| :------- | :---------- |
| 5 | DESKTOP-1 |
| 6 | DESKTOP-2 |
| 7 | DESKTOP-3 |