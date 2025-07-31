# wildcard()

The `wildcard()` function performs a wildcard (glob) pattern search on event fields, making it easier to find matches without using complex regular expressions.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | array of strings | optional | The field or fields to search within. If not specified, all fields of the original event are searched. |
| **ignoreCase**| boolean | optional <br> default: `false` | If `true`, the search will be case-insensitive. |
| **pattern** | string | required | The wildcard pattern to search for. The parameter name `pattern` can be omitted. |

***

## Function Operation

The behavior of `wildcard()` changes based on its parameters:
* **Case-sensitive, specific field**: `wildcard(field=log, pattern="*error*")` is equivalent to a standard wildcard search: `log="*error*"`.
* **Case-sensitive, all fields**: `wildcard(pattern="*error*")` performs a free-text search for the pattern across the entire event.
* **Case-insensitive**: When `ignoreCase=true`, the search is equivalent to a case-insensitive regular expression. For performance reasons, only set `ignoreCase=true` when necessary.

The function can be negated using `!wildcard(...)` to find all events that do *not* match the pattern.

***

## Examples

### Drop Events Based on Specific Field Values or Patterns

This example demonstrates using `wildcard()` as one of several conditions within a `case` statement to filter events based on complex logic.

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
    1.  A `case` statement is used to evaluate multiple conditions.
    2.  The first two lines drop events if `fielda` or `fieldb` contain an exact "badresult".
    3.  The third condition uses `wildcard("badip*", field=[fieldc, fieldd])`. This checks if the value in *either* `fieldc` or `fieldd` starts with the string "badip". If a match is found in either field, the event is dropped.
    4.  The final `*` ensures all other events are kept.

* **Summary and Results**
    This query uses `wildcard()` to apply a pattern match across multiple fields as part of a larger filtering strategy, allowing for flexible and powerful event processing.

---

### Find and Group Fields With a Specific Pattern

This example shows a common workflow: using `wildcard()` to find all relevant events and then piping the results to `groupBy()` for aggregation.

* **Query Example**
    ```
    wildcard(field=class, pattern="*S3Bucket*", ignoreCase=true)
    | groupBy(class)
    ```

* **Step-by-Step**
    1.  `wildcard(...)`: The query first searches the `class` field for any value that contains the string "S3Bucket", ignoring case.
    2.  `groupBy(class)`: The events that match are then piped to `groupBy()`, which groups them by the unique `class` names and provides a count for each.

* **Result Event Data**
    The output is an aggregated count of all events that have "S3Bucket" anywhere in their `class` field, regardless of capitalization.

| class | _count |
| :--- | :--- |
| c.h.b.s.S3BucketStorageCleaningJob| 197 |
| c.h.b.s.S3BucketStorageFileUploader| 2329 |
| c.h.b.s.S3BucketStorageUploadJob | 3869 |

---

### Search Multiple Fields for a Specific Pattern

This example shows how to use the `field` parameter to search for a pattern in a list of specified fields.

* **Query Example**
    ```
    wildcard(field=[plant, animal], pattern="*horse*", ignoreCase=false)
    ```
* **Input Event Data**
    Events that contain either a `plant` or an `animal` field.

| field | value |
| :--- | :--- |
| animal | "horse" |
| animal | "dancing with horses" |
| plant | "daisy" |
| plant | "horseflower" |

* **Step-by-Step**
    1.  The query starts with the source events.
    2.  `wildcard()` searches for the pattern `*horse*` (the string "horse" anywhere in the value).
    3.  The `field=[plant, animal]` parameter tells it to only check in the `plant` and `animal` fields.
    4.  The search is case-sensitive because `ignoreCase` is false (the default).

* **Summary and Results**
    The query returns a list of events where either the `animal` field or the `plant` field contains the string "horse". This is equivalent to running `animal="*horse*" OR plant="*horse*"`.