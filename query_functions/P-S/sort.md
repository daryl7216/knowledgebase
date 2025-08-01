# sort()

The `sort()` query function is used to sort events based on a given field or fields. Events can be sorted by multiple fields by setting the `field` parameter to an array of field names (for example, between square-brackets in a comma-separated list). Likewise, the `order` and `type` of each field can be specified by setting the `order` and `type` parameters to arrays.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | array of strings | optional | The fields by which to sort events. |
| `limit` | integer | optional \<br\> default: `200` | The argument given to this parameter determines the limit on the number of events included in the result of the function. The default argument is `default`. The maximum is controlled by the `StateRowLimit` dynamic configuration, which is `StateRowLimit` by default. If the argument is `max` (`limit=max`), then the value of `StateRowLimit` is used. |
| `order` | array of strings | optional \<br\> default: `desc` | The order in which to sort. \<br\>**Values:**\<br\>`asc`: Sort ascending. \<br\>`desc`: Sort descending. |
| `reverse` | boolean | optional \<br\> default: `false` | Whether to sort in descending order. This parameter is deprecated: use `order` instead. \<br\>**Values:**\<br\>`false`: Sort ascending. \<br\>`true`: Sort descending. |
| `type` | array of strings | optional \<br\> default: `number` | Set the data type of the fields to influence the sorting order. \<br\>**Values:**\<br\>`any`: Any fields. \<br\>`hex`: Treat the field as a hexadecimal value and sort in numerical order. \<br\>`number`: Treat the field as numerical and sort in numerical order. \<br\>`string`: Treat the field as string values and sort alphabetical. |

***

## Function Operation

The `sort()` function is used to arrange events in a specified order based on the values of one or more fields.

Key aspects of its operation:

  * **Field Specification**: Events can be sorted by a single field or multiple fields by providing an array of field names to the `field` parameter.
  * **Sorting Order**: The `order` parameter dictates the sort direction, either `asc` (ascending) or `desc` (descending, which is the default). When sorting by multiple fields, you can provide a corresponding array to `order` to specify a different direction for each field. The `reverse` parameter is deprecated in favor of `order`.
  * **Data Type for Sorting**: The `type` parameter informs `sort()` how to compare values. Options include `string` (lexicographical order), `number` (numerical magnitude), and `hex` (treating as hexadecimal values for numerical sorting). If `order` or `type` is a single value, it applies to all specified fields.
  * **Limiting Results**: The `limit` parameter controls the maximum number of events returned *after* the sorting is complete. The default limit is 200, but can be set to `max` to use the `StateRowLimit` dynamic configuration value.
  * **Performance Considerations**: Sorting is an in-memory operation and can be resource-intensive for large numbers of events. For optimal performance, it is recommended to:
      * Perform filtering prior to sorting to reduce the number of events.
      * Use aggregation functions to simplify the event list before sorting.
      * Place `sort()` as the last function in a query, especially after an aggregating function, for best performance.

***

## Examples

### Get a list of referring domains and sub-domains, counted and sorted alphabetically in reverse order

This example extracts referring domains from URLs, groups them, counts them, and then sorts them alphabetically in descending order.

  * **Query Example**

    ```
    regex(regex="/.*/(?<ref_url>.+?)(/|$)", field=referrer)
    groupBy(ref_url)
    | sort(ref_url, type=string, order=desc, limit=12)
    ```

  * **Step-by-Step**

    1.  The `regex()` function extracts just the domain and sub-domain portion of the URL from the `referrer` field into a new field called `ref_url`, dropping everything after the domain and first slash.
    2.  `groupBy(ref_url)`: Groups events by each unique `ref_url` and implicitly counts their occurrences.
    3.  `| sort(ref_url, type=string, order=desc, limit=12)`: Sorts the grouped results by `ref_url` as a `string` (alphabetically) in `descending` order, and limits the output to the top 12 entries.

  * **Result Event Data**

| ref\_url | \_count |
| :--- | :--- |
| www.zabotkin.ru | 14 |
| www.tangblog.top | 1 |
| [www.skidn.com](https://www.google.com/search?q=https://www.skidn.com) | 1 |
| www.klfd.net | 1 |
| www.iopt.pro | 42 |
| [www.google.com](https://www.google.com).au | 1 |
| [www.google.com](https://www.google.com) | 11 |
| [www.bing.com](https://www.bing.com) | 3 |

---

### Sort metrics by repository name and then output the top entry

This example shows how to use `sort()` with `groupBy()` to find the single top entry after sorting.

  * **Query Example**

    ```
    name=datasource-count
    groupBy([repo])
    | sort(field=_count, type=number, order=desc, limit=1)
    ```

  * **Step-by-Step**

    1.  `name=datasource-count`: Filters events where the `name` field is `datasource-count`.
    2.  `groupBy([repo])`: Groups the events by repository (`repo`) and implicitly counts them.
    3.  `| sort(field=_count, type=number, order=desc, limit=1)`: Sorts the grouped counts (in the `_count` field) numerically in descending order, and then limits the result to only the top 1 entry.

---

### Sort events by multiple fields with different orders

This example demonstrates sorting by multiple fields where each field has a different sorting order (one descending, one ascending).

  * **Query Example**

    ```
    groupBy([ref_url, method])
    | sort([ref_url, method], type=string, order=[desc, asc], limit=12)
    ```

  * **Step-by-Step**

    1.  `groupBy([ref_url, method])`: Groups events by both `ref_url` and `method`.
    2.  `| sort([ref_url, method], type=string, order=[desc, asc], limit=12)`: Sorts the results. `ref_url` will be sorted as a string in descending order, while `method` will be sorted as a string in ascending order. The output is limited to 12 results.

---

### Analyze User Sessions Based on Click Activity

This example analyzes user sessions, groups them by cookie ID, counts clicks within each session, and then sorts the sessions by the number of clicks.

  * **Query Example**

    ```
    groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))
    sort(clicks)
    ```

  * **Input Event Data**

    | timestamp | cookie\_id | action\_type | page\_url | user\_agent |
    | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-15 05:30:00 | user123 | pageview | /home | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:30:15 | user123 | click | /products | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:30:30 | user123 | click | /product/item1 | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:31:00 | user123 | click | /cart | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:31:30 | user123 | click | /checkout | Mozilla/5.0 (Windows NT 6.1; Win64; x64) |
    | 2025-05-15 05:35:00 | user456 | pageview | /home | Mozilla/5.0 (Macintosh; Mac OS X) |
    | 2025-05-15 05:35:30 | user456 | click | /about | Mozilla/5.0 (Macintosh; Mac OS X) |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))`: Groups events by `cookie_id` and creates sessions with a 15-minute inactivity timeout. It then counts the number of events (clicks) within each session, storing the result in a field named `clicks`.
    3.  `sort(clicks)`: Sorts the results by the `clicks` field (by default, in descending order).
    4.  Event Result set.

  * **Summary and Results**
    The query analyzes user sessions based on click activity, useful for identifying most/least active user sessions, detecting potential automated behavior, or understanding user engagement levels.

  * **Result Event Data**

| cookie\_id | clicks |
| :--- | :--- |
| user123 | 5 |
| user456 | 3 |
| user789 | 2 |

---

### Calculate Query Cost for All Users by Repository

This example searches across multiple repositories to calculate query costs for all users by repository, then sorts the results by live and static costs.

  * **Query Example**

    ```
    #type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"
    groupBy(repo, initiatingUser, totalLiveCost, totalStaticCost)
    | sort([totalLiveCost, totalStaticCost])
    ```

  * **Input Event Data**

    | \#type | \#kind | class | message | timestamp |
    | :--- | :--- | :--- | :--- | :--- |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:30:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:31:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:32:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:33:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:34:00Z |
    | humio | logs | c.h.j.RunningQueriesLoggerJob | Highest Cost query | 2025-03-26T09:35:00Z |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#type=humio #kind=logs class=c.h.j.RunningQueriesLoggerJob message="Highest Cost query"`: Filters for internal LogScale logs containing `c.h.j.RunningQueriesLoggerJob` in the `class` field and where the `message` field is "Highest Cost query".
    3.  `groupBy(repo, initiatingUser, totalLiveCost, totalStaticCost)`: Groups the results by `repo`, `initiatingUser`, `totalLiveCost`, and `totalStaticCost`, and returns a count in a field named `_count`.
    4.  `| sort([totalLiveCost, totalStaticCost])`: Sorts the results by both `totalLiveCost` and `totalStaticCost` fields (by default, in descending order).
    5.  Event Result set.

  * **Summary and Results**
    The query searches across multiple repositories and outputs query costs for all users by repository. The query returns the count in a field named `_count`. Use this query to focus on live and static costs separately.

  * **Result Event Data**

| repo | initiatingUser | totalLiveCost | totalStaticCost | \_count |
| :--- | :--- | :--- | :--- | :--- |
| app-logs | jane.smith | 2000 | 1200 | 1 |
| security-logs | john.doe | 1500 | 800 | 1 |
| infra-logs | bob.wilson | 1000 | 500 | 1 |

---

### Calculate Total Network Bandwidth Per Host

This example calculates total inbound and outbound network traffic per host, then calculates total bandwidth consumption, and finally sorts by total traffic.

  * **Query Example**

    ```
    event_simpleName="NetworkConnectStats"
    groupBy([ComputerName], function=[
        sum(field="BytesReceived", as=InboundTraffic),
        sum(field="BytesSent", as=OutboundTraffic)
    ])
    TotalTraffic := InboundTraffic + OutboundTraffic
    sort(field="TotalTraffic", order="desc")
    ```

  * **Input Event Data**

    | @timestamp | event\_simpleName | ComputerName | BytesReceived | BytesSent |
    | :--- | :--- | :--- | :--- | :--- |
    | 1686837825000 | NetworkConnectStats | DESKTOP-A1 | 15000000 | 8000000 |
    | 1686837825000 | NetworkConnectStats | DESKTOP-A1 | 8900000 | 4600000 |
    | 1686837825000 | NetworkConnectStats | LAPTOP-B2 | 25000000 | 12500000 |
    | 1686837826000 | NetworkConnectStats | SERVER-C3 | 95000000 | 47000000 |
    | 1686837826000 | NetworkConnectStats | DESKTOP-A1 | 12000000 | 6000000 |
    | 1686837826000 | NetworkConnectStats | LAPTOP-B2 | 18000000 | 9000000 |
    | 1686837827000 | NetworkConnectStats | SERVER-C3 | 85000000 | 42000000 |
    | 1686837827000 | NetworkConnectStats | DESKTOP-D4 | 5000000 | 2500000 |
    | 1686837827000 | NetworkConnectStats | LAPTOP-B2 | 22000000 | 11000000 |
    | 1686837828000 | NetworkConnectStats | SERVER-C3 | 105000000 | 50000000 |
    | 1686837828000 | NetworkConnectStats | DESKTOP-D4 | 6500000 | 3500000 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `event_simpleName="NetworkConnectStats"`: Filters events to include only those where `event_simpleName` equals "NetworkConnectStats".
    3.  `groupBy([ComputerName], function=[sum(field="BytesReceived", as=InboundTraffic), sum(field="BytesSent", as=OutboundTraffic)])`: Groups the data by `ComputerName` and calculates two aggregate values: the sum of `BytesReceived` (stored as `InboundTraffic`) and the sum of `BytesSent` (stored as `OutboundTraffic`).
    4.  `TotalTraffic := InboundTraffic + OutboundTraffic`: Creates a new field named `TotalTraffic` by summing the `InboundTraffic` and `OutboundTraffic` values.
    5.  `sort(field="TotalTraffic", order="desc")`: Sorts the results based on the `TotalTraffic` field in descending order, showing hosts with the highest bandwidth consumption first.
    6.  Event Result set.

  * **Summary and Results**
    The query analyzes network bandwidth consumption patterns across different hosts. This is useful for identifying hosts consuming excessive bandwidth, monitoring network usage, or detecting network-intensive applications or anomalies. The traffic values are in bytes, and each row represents aggregated traffic for a unique host.

  * **Result Event Data**

| ComputerName | InboundTraffic | OutboundTraffic | TotalTraffic |
| :--- | :--- | :--- | :--- |
| SERVER-C3 | 285000000 | 139000000 | 424000000 |
| LAPTOP-B2 | 65000000 | 32500000 | 97500000 |
| DESKTOP-A1 | 45700000 | 24600000 | 70300000 |
| DESKTOP-D4 | 11500000 | 6000000 | 17500000 |

---

### Filter Out Fields With No Value

This example demonstrates how to filter out fields with no values from search results.

  * **Query Example**

    ```
    method=GET
    groupBy(field=[method, statuscode], function=count(as=method_total))
    sort([method, statuscode], order=asc)
    FieldName!=""
    ```

  * **Input Event Data**
    (Example output after `groupBy` and `sort` before final filter)

    | method | statuscode | method\_total |
    | :--- | :--- | :--- |
    | GET | \<no value\> | 10 |
    | GET | 200 | 32492 |
    | GET | 301 | 1 |
    | GET | 304 | 113 |
    | GET | 403 | 9 |
    | GET | 404 | 132 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `method=GET`: Filters for all events with methods of the type `GET`.
    3.  `groupBy(field=[method, statuscode], function=count(as=method_total))`: Groups the returned results into `method` and `statuscode` fields and counts the events into a new field named `method_total`.
    4.  `sort([method, statuscode], order=asc)`: Sorts the returned results in ascending order.
    5.  `FieldName!=""`: Excludes all events where one of the fields in the result set (`method` or `statuscode`) does not contain a value. (Note: The `FieldName!=""` syntax typically checks if a *specific* field is not empty. If the intent is to check all output fields, a different approach might be needed or `FieldName` should represent the actual field to check, e.g. `statuscode!=""`).
    6.  Event Result set.

  * **Summary and Results**
    The query filters out fields not containing any values from the returned search result.

  * **Result Event Data**

| method | statuscode | method\_total |
| :--- | :--- | :--- |
| GET | 200 | 32492 |
| GET | 301 | 1 |
| GET | 304 | 113 |
| GET | 403 | 9 |
| GET | 404 | 132 |

---

### Find Least Common Values of a Field

This example finds the least common values of a field by sorting aggregated counts in ascending order. This is the opposite of the `top()` function.

  * **Query Example**

    ```
    groupby([type, kind], limit=max, function=count())
    sort(_count, order=asc)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupby([type, kind], limit=max, function=count())`: Groups the event set by the `type` and `kind` fields. For each unique combination, it counts the number of events and returns the results in a field named `_count`. The `limit=max` parameter is set to return all counts, as the purpose of this example is to find the least common values of a field.
    3.  `sort(_count, order=asc)`: Sorts the returned results in ascending order from the least common combinations to the most common combinations of `type` and `kind`.
    4.  Event Result set.

  * **Summary and Results**
    The query finds the least common values of a field in a set of events, in this example, identifying the least and most common type/kind combinations in an event set. Identifying rare combinations of type and kind could indicate unusual events or potential security issues. This type of query is useful in various scenarios, particularly for data analysis and understanding patterns in event sets.

---

### Sort Timestamps With `groupBy()`

This example demonstrates how to sort fields based on aggregated field values, specifically sorting timestamps within a `groupBy()` aggregation.

  * **Query Example**

    ```
    timestamp: formatTime(format="%H:%M")
    groupBy([thread], function=[{sort("timestamp") | collect("timestamp")}])
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `timestamp: formatTime(format="%H:%M")`: Creates a new field, `timestamp`, formatted as `HH:MM`.
    3.  `groupBy([thread], function=[{sort("timestamp") | collect("timestamp")}])`: Groups the events by the name of the `thread`. As an embedded expression for the function, it first sorts the events on the `timestamp` field (the newly formatted `HH:MM` timestamp) and then uses `collect("timestamp")` to retrieve and include that sorted `timestamp` field's value in the aggregated event set, as it would normally be removed during aggregation.
    4.  Event Result set.

  * **Summary and Results**
    The result set will contain a list of the aggregated thread names, with their associated timestamps sorted.

  * **Result Event Data (Example)**

    ```
    10:09
    LogCollector ManifestUpdate
    TransientChatter event loop
    10:10
    ```