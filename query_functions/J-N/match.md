# match()

The `match()` function matches or joins data from query results with a table. The table can be provided either as a lookup CSV file or through a limited form of JSON file, uploaded using Lookup Files or as an ad-hoc table using Ad-hoc Tables.

-----

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `column` | optional string, field, or array | optional | Which column in the file to use for the match. A single column or an array of columns can be specified. |
| `field` | required string or array | required | Which field in the event (log line) must match the given column value. A single field or an array of fields can be specified. Field and column must have the same length, are matched in order and must all match. |
| `file` | required string | required | Specifies the source file (when using Look Up files) or the name of the ad-hoc table. The file name should be specified with `.csv` or `.json` suffix. |
| `glob` | optional boolean | optional \<br\> default: `false` | This parameter is deprecated. Use `mode` parameter with the `glob` option instead. |
| `ignoreCase` | optional boolean | optional \<br\> default: `false` | If true, ignore case when matching against the CSV data. \<br\>**Values:**\<br\>`false`: Perform a case-sensitive match of the event values with the lookup table. `true`: Ignore the case of the event values with the lookup table. |
| `include` | optional string or array | optional | The columns to include. If no argument is given, include all columns from the corresponding row in the output event. |
| `mode` | optional string | optional | The function to use when matching against keys. \<br\>**Values:**\<br\>`cidr`: The key is interpreted as a CIDR subnet and the event is matched if the field contains an IP within the subnet. If multiple subnets match, the most specific one is selected or an arbitrary one if there are multiple equally specific subnets. \<br\>`glob`: The key is interpreted as a globbing pattern and matched accordingly, for example, a CSV key value of `*thisMatch*` would match the field value of `123thisMatch456`. When using `mode=glob`, the underlying CSV is limited to 20,000 rows/lines. For self-hosted customers, the maximum value for glob matches is configurable using `GLOB_MATCH_LIMIT`. \<br\>`string`: The matching is done using exact string matching. |
| `strict` | optional boolean | optional \<br\> default: `true` | If true (the default) selects only the fields that match a key in the file; if false lets all events through (works like the deprecated `lookup()`). |

-----

## Function Operation

The `match()` function is used to join or match data from query results with an external table. This table can be either a CSV file or a JSON file.

For CSV files, whitespace is included in keys and values, and commas in values should be quoted. The first line of a CSV is interpreted as column titles. When using `match()` with a single column, the last matching row in the CSV is used.

For JSON files, two formats are supported: object-based and array-based. In the object-based variant, the lookup values are declared as an object with a key and embedded fields, where the key field does not have a name; the key for each value is used for matching. Nested JSON objects are not supported and will not be matched, though uploading a file with them will not fail. In the array-based variant, lookup values are an array of objects, and the key field is selected using the `column` parameter in `match()`. Any field in the JSON file can be used as the match value.

The default behavior of `match()` when `strict` is set to `true` works like an `INNER JOIN`, selecting only fields that match a key in the file. When `strict` is set to `false`, the function enriches events, allowing all events through but only enriching those with matches.

The `mode` parameter controls the matching behavior:

  * `cidr`: Interprets the key as a CIDR subnet and matches if the event field contains an IP within that subnet. If multiple subnets match, the most specific one is selected.
  * `glob`: Interprets the key as a globbing pattern. When using `mode=glob`, the underlying CSV is limited to 20,000 rows/lines.
  * `string`: Performs exact string matching.

The `match()` function does not report an error if the file format cannot be parsed. Up until LogScale version 1.140, the file is limited to 1,000,000 rows/lines for exact matching using `mode=string`.

-----

## Examples

### Match Multiple Pairs of Event Fields Against Multiple Columns in .CSV Lookup File

This example demonstrates how to compare multiple pairs of event fields against multiple columns in a CSV lookup file simultaneously using the `match()` function.

  * **Query Example**

    ```
    match(test.csv, field=[field1, field2], column=[column1, column2])
    ```

  * **Input Event Data**
    (Example `test.csv` file data)

    | column1 | column2 | column3 |
    | :--- | :--- | :--- |
    | a | b | d |
    | C | e | f |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  For each event, `match(test.csv, field=[field1, field2], column=[column1, column2])` checks if `field1` matches `column1` and `field2` matches `column2`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to match multiple pairs of event fields against multiple columns in the `.CSV` file named `test.csv`. Multiple field matching helps validate and enrich complex event data.

  * **Result Event Data**

    | column3 | field1 | field2 |
    | :--- | :--- | :--- |
    | f | C | e |

---

### Filter For Items Not Part of Data Set Using `match()`

This example shows how to find the set difference using the `match()` function with negation to search and find IP addresses that are not part of a known list (`known_ips.csv`).

  * **Query Example**

    ```
    src_ip=*
    | !match("known_ips.csv", field=src_ip)
    ```

  * **Input Event Data**
    (Example incoming data)

    | timestamp | src\_ip | dst\_ip | src\_port | dst\_port | protc |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-04-01T07:00:00Z | 192.168.1.101 | 10.0.0.50 | 52431 | 443 | TCP |
    | 2025-04-01T07:00:01Z | 172.16.0.24 | 8.8.8.8 | 33221 | 53 | UDP |
    | 2025-04-01T07:00:02Z | 192.168.1.150 | 172.16.0.100 | 49223 | 80 | TCP |
    | 2025-04-01T07:00:03Z | 10.0.0.75 | 192.168.1.1 | 55678 | 22 | TCP |
    | 2025-04-01T07:00:04Z | 192.168.1.200 | 1.1.1.1 | 44556 | 53 | UDP |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `src_ip=*`: Filters for all events that have a `src_ip` field.
    3.  `!match("known_ips.csv", field=src_ip)`: Excludes (filters out) any events where the `src_ip` field matches entries in the file `known_ips.csv`, and returns a list of IP addresses that are not found in the specified file. The negation operator is used to return non-matching results.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to search for unknown or unexpected source IP addresses matched up against a known list. This is useful for detecting potential security threats and monitoring for unauthorized network access.

  * **Result Event Data**

    | timestamp | src\_ip | dst\_ip | src\_port | dst\_port | protc |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-04-01T07:00:00Z | 192.168.1.101 | 10.0.0.50 | 52431 | 443 | TCP |
    | 2025-04-01T07:00:01Z | 172.16.0.24 | 8.8.8.8 | 33221 | 53 | UDP |

---

### Match Event Fields Against Lookup Table Values Adding Specific Columns

This example compares event IP fields with CIDR ranges in a lookup table using the `match()` function with the `mode` parameter set to `cidr`. The query matches IP addresses against CIDR blocks (CIDR subnets) and adds specific network information from the columns `info` and `type` to the output rows.

  * **Query Example**

    ```
    match(file="cidr-file.csv", column="cidr-block", field=ip, mode=cidr, include=["info", "type"])
    ```

  * **Input Event Data**
    (Example incoming event data)

    | @timestamp | ip | action |
    | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | 10.0.1.25 | login |
    | 2024-01-15T09:01:00.000Z | 192.168.1.100 | connect |
    | 2024-01-15T09:02:00.000Z | 172.16.5.12 | access |

    (Example `cidr-file.csv` file data)

    | cidr-block | info | type | location | department |
    | :--- | :--- | :--- | :--- | :--- |
    | 10.0.1.0/24 | Internal Network | corporate | HQ | IT |
    | 192.168.1.0/24 | Development Network | test | Lab | Engineering |
    | 172.16.0.0/16 | Production Network | critical | DC | Operations |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `match(file="cidr-file.csv", column="cidr-block", field=ip, mode=cidr, include=["info", "type"])`: Uses CIDR ranges matching to match the `cidr-block` column of the `cidr-file.csv` lookup table file against the IP addresses (`ip` field) in the events, and adds specific network information to the output rows. It will only add the specified columns of the matching row, and the column names become new field names. When the `mode` parameter is set to `cidr`, the event is matched if the field contains an IP address within the CIDR subnet. If multiple subnets match, the most specific one is selected, or an arbitrary one if there are multiple equally specific subnets.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to match IP addresses against CIDR blocks and add specific network information from the columns `info` and `type` to the output rows. This query helps analyze network traffic and security events by mapping IP addresses to network segments.

  * **Result Event Data**

    | @timestamp | ip | action | info | type |
    | :--- | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | 10.0.1.25 | login | Internal Network | corporate |
    | 2024-01-15T09:01:00.000Z | 192.168.1.100 | connect | Development Network | test |
    | 2024-01-15T09:02:00.000Z | 172.16.5.12 | access | Production Network | critical |

    Notice how only the specified fields from the `cidr-file.csv` file appear in output.

---

### Match Event Fields Against Lookup Table Values Allowing All Events to Pass

This example compares event fields with column values in a lookup table using the `match()` function with the `strict` parameter set to `false` to allow also non-matching events to pass. When `strict` is set to `false`, all events (including non-matching ones) are passed through, but only events with matches will be enriched with all other columns of the matching row.

  * **Query Example**

    ```
    id=~ match(file="users.csv", column=userid, strict=false)
    ```

  * **Input Event Data**
    (Example incoming event data)

    | @timestamp | id | action | source\_ip |
    | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 |
    | 2024-01-15T09:01:00.000Z | unknown-user | login\_attempt | 10.0.0.2 |
    | 2024-01-15T09:02:00.000Z | dev-user-456 | code\_push | 10.0.0.3 |

    (Example `users.csv` file data)

    | userid | department | access\_level | location |
    | :--- | :--- | :--- | :--- |
    | ADMIN-123 | IT | administrator | HQ |
    | dev-user-456 | Engineering | developer | Remote |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `id=~ match(file="users.csv", column=userid, strict=false)`: When an event ID matches the column `userid` in the `users.csv` lookup file, all columns from that first matching row are added to the event. The column names become new field names. As the `strict` parameter is set to `false`, all events (including non-matching events) are passed through, but only events with matches will be enriched with all other columns of the matching row.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to enrich matching events while allowing all events to pass through. Matching events against an authorized users list is useful for filtering out unauthorized access attempts, for validation of user activities, or other monitoring. Showing non-matching events in output as well can be useful for tracking unauthorized access attempts, identifying unknown users or systems, or for monitoring suspicious activities.

  * **Result Event Data**

    | @timestamp | id | action | source\_ip | department | access\_level |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 | IT | administrator |
    | 2024-01-15T09:01:00.000Z | unknown-user | login\_attempt | 10.0.0.2 | \<no value\> | \<no value\> |
    | 2024-01-15T09:02:00.000Z | dev-user-456 | code\_push | 10.0.0.3 | Engineering | developer |

    After matching, the output combines original event fields with matched user details. Notice how also non-matching events (in this example `unknown-user`) appear in output.

---

### Match Event Fields Against Patterns in Lookup Table Values

This example compares event fields with column values containing patterns in a lookup table using the `match()` function with glob pattern matching (`mode=glob`). This query matches event IDs against patterns in a user list and then adds the matching user details (other columns from the row in the `users.csv` file) to the events.

  * **Query Example**

    ```
    id = match(file="users.csv", column=userid, mode=glob, ignoreCase=true)
    ```

  * **Input Event Data**
    (Example incoming event data)

    | @timestamp | id | action | source\_ip |
    | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 |
    | 2024-01-15T09:01:00.000Z | dev-user-456 | code\_push | 10.0.0.2 |
    | 2024-01-15T09:02:00.000Z | TEST\_789 | test\_run | 10.0.0.3 |
    | 2024-01-15T09:03:00.000Z | support-001 | ticket\_update | 10.0.0.4 |
    | 2024-01-15T09:04:00.000Z | unknown-user | login\_attempt | 10.0.0.5 |

    (Example `users.csv` file data)

    | userid | department | access\_level | location | title |
    | :--- | :--- | :--- | :--- | :--- |
    | ADMIN-\* | IT | administrator | HQ | System Administrator |
    | dev-user-\* | Engineering | developer | Remote | Software Engineer |
    | TEST\_\* | QA | tester | Lab | QA Engineer |
    | support-\* | Support | agent | Office | Support Specialist |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `id match(file="users.csv", column=userid, mode=glob, ignoreCase=true)`: Uses glob pattern matching to match the `userid` column of the `users.csv` lookup table file against the IDs in the events. When an event ID matches a pattern in `users.csv`, all (non-pattern) columns from that first matching row are added to the event. The column names become new field names.
        Pattern matching examples based on the input data and lookup file are:
          * `ADMIN-123` matches `ADMIN-*`
          * `dev-user-456` matches `dev-user-*`
          * `TEST_789` matches `TEST_*`
          * `support-001` matches `support-*`
    3.  Event Result set.

  * **Summary and Results**
    The query is used to match event IDs against patterns in a user list, and then add the matching user details (other columns from the row in the `users.csv` file) to the events. Pattern-based matching with field enrichment is useful, for example, if you want to add user context to authentication logs.

  * **Result Event Data**

    | @timestamp | id | action | source\_ip | department | access\_level |
    | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2024-01-15T09:00:00.000Z | ADMIN-123 | login | 10.0.0.1 | IT | administrator |
    | 2024-01-15T09:01:00.000Z | dev-user-456 | code\_push | 10.0.0.2 | Engineering | developer |
    | 2024-01-15T09:02:00.000Z | TEST\_789 | test\_run | 10.0.0.3 | QA | tester |
    | 2024-01-15T09:03:00.000Z | support-001 | ticket\_update | 10.0.0.4 | Support | agent |

    After matching, the output combines original event fields with matched user details. Only events with matching patterns appear in output.