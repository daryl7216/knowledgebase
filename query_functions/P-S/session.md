# session()

The `session()` function collects events into sessions, which are series of events that are no further than `maxpause` apart (defaults to 15m), and then performs an aggregate operation across the events that make up the session.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `function` | array of aggregate functions | optional \<br\> default: `count(as=_count)` | Specifies which aggregate functions to perform on each session. If several aggregators are listed for the `function` parameter, then their outputs are combined using the rules described for `stats()`. |
| `maxpause` | string | optional \<br\> default: `15m` | Defines the maximum pause between sessions; for example, events more than this far apart will become separate sessions. |

***

## Function Operation

The `session()` function groups related events into "sessions" based on a maximum inactivity period defined by the `maxpause` parameter. By default, events are grouped into the same session if they occur no more than 15 minutes apart. If the time between two consecutive events exceeds this `maxpause`, a new session is started for the subsequent event.

Once events are grouped into sessions, `session()` applies one or more aggregate functions (specified by the `function` parameter) across all events within each session. If multiple aggregate functions are provided, their outputs are combined following the rules similar to `stats()`. By default, `session()` performs a `count(as=_count)` on each session.

This function is typically used in conjunction with `groupBy()` to analyze sessions for unique identifiers (e.g., `cookie_id`, `client_ip`). It is valuable for understanding user engagement, identifying activity patterns, and detecting anomalies by providing a consolidated view of actions within a defined period of continuous activity.

***

## Examples

### Analyze User Sessions Based on Click Activity

This example uses the `session()` function to analyze user sessions based on their click activity, grouping events by `cookie_id` and counting clicks within sessions.

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
    2.  `groupBy(cookie_id, function=session(maxpause=15m, count(as=clicks)))`: Groups events by the `cookie_id` field (unique user identifier). It then creates sessions with a 15-minute inactivity timeout (the default value of the `maxpause` parameter). For each session, it counts the number of events (clicks) and returns the result in a new field named `clicks`.
    3.  `sort(clicks)`: Sorts the results by the number of clicks (by default, in descending order).
    4.  Event Result set.

  * **Summary and Results**
    The query is used to analyze user sessions based on their click activity. This is useful, for example, to identify the most/least active user sessions, detect potential automated behavior, or simply to understand user engagement levels.

  * **Result Event Data**

    | cookie\_id | clicks |
    | :--- | :--- |
    | user123 | 5 |
    | user456 | 3 |
    | user789 | 2 |

---

### Count Unique Visitors Based on Client IP Addresses

This example uses the `session()` function to count unique visitors (each visitor defined as non-active for 15 minutes) of a site based on client IP addresses.

  * **Query Example**

    ```
    groupBy(client_ip, function=session(maxpause=15m))
    | count()
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(client_ip, function=session(maxpause=15m))`: Groups events by the `client_ip` field into sessions, with events more than 15 minutes apart becoming separate sessions. For example, if the same user returns to a site within 15 minutes, it will be considered the same session. By default, `session()` performs `count(as=_count)` on each session, so this step implicitly counts sessions per IP.
    3.  `| count()`: Makes a total count of all the unique sessions generated across all IP addresses.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to group events by client IP addresses into 15-minute sessions, and then makes a count of the total number of unique sessions (returns the total count of sessions across all IP addresses). This query is useful, for example, for measuring unique website/application visitors and understanding real user engagement patterns. It is also useful for security monitoring and detection of unusual spikes in unique visitors.
    If you instead use `count(client_ip)` after the `groupBy` for `client_ip` and `session`, the query will return a more detailed result showing the session count per IP address. For example, if an IP address has activity that spans beyond the 15-minute session timeout, it would create two distinct sessions.

  * **Result Event Data (Total count)**
    `_count`
    `4`

  * **Result Event Data (Count per IP if `count(client_ip)` used)**

    | client\_ip | \_count |
    | :--- | :--- |
    | 192.168.1.100 | 2 |
    | 192.168.1.101 | 1 |
    | 192.168.1.102 | 1 |

---

### Find Minimum And Maximum Values of any Numerical Field in Session

This example uses the `session()` function to find the minimum and maximum values of the `bet` field within each session.

  * **Query Example**

    ```
    groupBy(cookie_id, function=session([max(bet), min(bet)]))
    ```

  * **Input Event Data**

    | timestamp | cookie\_id | bet | action\_type | category |
    | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-15 05:30:00 | user123 | 25.99 | purchase | electronics |
    | 2025-05-15 05:32:00 | user123 | 49.99 | purchase | electronics |
    | 2025-05-15 05:34:00 | user123 | 15.99 | purchase | accessories |
    | 2025-05-15 05:48:00 | user123 | 99.99 | purchase | appliances |
    | 2025-05-15 05:49:00 | user123 | 150.00 | purchase | furniture |
    | 2025-05-15 05:35:00 | user456 | 75.50 | purchase | clothing |
    | 2025-05-15 05:37:00 | user456 | 199.99 | purchase | appliances |
    | 2025-05-15 05:40:00 | user456 | 89.99 | purchase | electronics |
    | 2025-05-15 05:30:00 | user789 | 10.99 | purchase | books |
    | 2025-05-15 05:55:00 | user789 | 20.99 | purchase | books |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `groupBy(cookie_id, function=session([max(bet), min(bet)]))`: Groups events by the `cookie_id` field (unique user identifier) and creates sessions with a 15-minute timeout (the default value of the `maxpause` parameter). Then, for each session, it calculates the maximum and minimum values of the `bet` field, returning the results in new fields named `_max` and `_min`.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to analyze the likelihood (the "bet") of behavior within user sessions. This query is useful, for example, for identifying if an event was an attempt to hack the system. Each session shows its own min/max values, highlighting that `user123` and `user789` had two sessions each due to a pause exceeding 15 minutes.

  * **Result Event Data**

    | cookie\_id | \_max | \_min |
    | :--- | :--- | :--- |
    | user123 | 49.99 | 15.99 |
    | user123 | 150.00 | 99.99 |
    | user456 | 199.99 | 75.50 |
    | user789 | 10.99 | 10.99 |
    | user789 | 20.99 | 20.99 |