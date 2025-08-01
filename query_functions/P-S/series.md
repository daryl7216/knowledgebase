# series()

The `series()` function collects a series of values for the selected fields from multiple events into one (or more) events. Combined with `groupBy()`, this can be used to gather data from transactions by some identity field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `collect` | array of strings | required | Names of the fields to keep. |
| `endmatch` | filter | optional | A filter query inside `{}` to match the end of a transaction (applied to the event as a whole), for example, `{/session end:/}`. Even with this parameter specified, "partial" sessions which do not include an end event are output, unlike what happens with the `startmatch` parameter, `endmatch` does not cause any event to be ignored. |
| `maxduration` | relative-time | optional | Maximum duration of a transaction (for example, `5min`), specified as a Relative Time Syntax. |
| `maxpause` | relative-time | optional | Maximum time between events in a transaction (for example, `10s`), specified as a Relative Time Syntax. |
| `memlimit` | string | optional \<br\> default: `1KiB` \<br\> minimum: `1` \<br\> maximum: `1024` | Limit on number of bytes of memory consumed by each series invocation (defaults to 1KiB). When used with the parameters `startmatch`, `endmatch`, `maxpause` and `maxduration` to produce multiple sub-series, this parameter controls the memory usage of the entire sequence of series, not each individual one. When `series` is used inside a `groupBy()`, this parameter only limits the memory consumption per group. So if the `groupBy()` is limited to 50,000 groups each using 1KB, the combined upper limit would be 50MB. |
| `separator` | string | optional \<br\> default: `\n` | String used to separate multiple values. |
| `startmatch` | filter | optional | A filter query inside `{}` to match the start of a transaction (applied to the event as a whole), for example, `{/session start:/}`. With this parameter specified, any event coming before the first start event, or in between an end event and the start event that follows, is not part of any session and is therefore ignored - all sessions include exactly one start event. |

***

## Function Operation

The `series()` function is used to gather values of specified fields from a sequence of events and consolidate them into one or more resultant events. This is particularly useful for reconstructing transaction flows or user sessions by collecting related data points that are spread across multiple individual log entries. It works effectively when combined with `groupBy()` to analyze data based on a common identity (e.g., `userID`, `client_ip`).

The function allows for flexible definition of a "series" or "session" using several optional parameters:

  * **`collect`**: Defines which fields' values to accumulate into the output series.
  * **`startmatch`**: A filter that specifies the event that marks the beginning of a new series. Events occurring before the first `startmatch` or between an `endmatch` and a subsequent `startmatch` are ignored. Each series will contain exactly one start event.
  * **`endmatch`**: A filter that specifies the event that marks the end of a series. Unlike `startmatch`, `endmatch` does not cause partial sessions (those without an end event) to be ignored; they are still outputted.
  * **`maxduration`**: Sets an upper time limit for a series. If a series exceeds this duration, it is terminated.
  * **`maxpause`**: Defines the maximum allowed time gap between consecutive events within a single series. If the pause between events exceeds this limit, the current series is considered ended, and a new one may begin with the next event.
  * **`separator`**: The string used to join the collected values within a field (default is a newline character).
  * **`memlimit`**: Controls the memory consumption for each series invocation, or for the entire sequence of sub-series when `startmatch`, `endmatch`, `maxpause`, and `maxduration` are used. When `series()` is nested within `groupBy()`, `memlimit` applies per group.

The output event(s) from `series()` typically include `@timestamp` (timestamp of the first event in the series), `_duration` (timespan of the series), and the collected fields as time-ordered series of values.

***

## Examples

### In an access log, collect the series of methods used for a given URL

This example shows how to collect a time-ordered series of HTTP methods for events related to a specific URL.

  * **Query Example**:

    ```
    url="/some/url"
    | series([method], separator=";")
    ```

  * **Input Event Data**:
    (Assume events related to `/some/url` with `method` fields, e.g., "GET", "POST", "GET", "DELETE")

  * **Result Event Data**:

    | Field | Example | Description |
    | :--- | :--- | :--- |
    | `@timestamp` | 145245466 | Timestamp of the first event arriving. |
    | `_duration` | 1245 | Timespan (in milliseconds) of the series of events included in this series. If the series contains just one field value, the value of `_duration` will be 0. |
    | `method` | GET;POST;GET; GET; DELETE | Time-ordered series of values for the method field. |

---

### Aggregate series of website visits, each visitor defined as non-active after 1 minute

This example uses `groupBy()` with `series()` to aggregate website visit data for each client IP, defining a new visit session if there's a pause of more than 1 minute between events.

  * **Query Example**:

    ```
    groupBy(client_ip, function=series(maxpause=1m, collect=[url], memlimit=1KB))
    ```

  * **Step-by-Step**:

    1.  `groupBy(client_ip, function=series(...))`: Groups events by `client_ip`.
    2.  `series(maxpause=1m, collect=[url], memlimit=1KB)`: For each `client_ip` group, it collects the `url` field values. A new series (visit) starts if there's a gap of more than 1 minute between events (`maxpause=1m`). The `memlimit=1KB` controls memory usage per series.

  * **Summary and Results**:
    This query effectively segments user activity into distinct "visits" based on inactivity. It can produce multiple events per `client_ip` if their activity has breaks longer than 1 minute, with each event representing a single visit series containing the URLs accessed during that visit.

---

### Aggregate series of auth logs, starting a new series for each login attempt

This example aggregates authentication logs into series, where each series begins with a "Login attempt" message.

  * **Query Example**:

    ```
    groupBy(userID, function=series(collect=[@rawstring], startmatch={ /Login attempt:/ }))
    ```

  * **Step-by-Step**:

    1.  `groupBy(userID, function=series(...))`: Groups events by `userID`.
    2.  `series(collect=[@rawstring], startmatch={ /Login attempt:/ })`: For each `userID`, it collects the `@rawstring` content into a series. A new series is initiated every time an event matching the filter `/Login attempt:/` is encountered. Events before the first `startmatch` or between an `endmatch` (if used) and a subsequent `startmatch` are ignored for session purposes.

  * **Summary and Results**:
    This query allows for tracking user activity starting precisely from a login attempt, providing a view of all subsequent raw log messages within that "session" until implicitly ended (or by `endmatch`, `maxpause`, etc.).

---

### Aggregate series of auth logs, ending each series with a failed login attempt

This example aggregates authentication logs, with each series terminating upon a "Failed Login" message.

  * **Query Example**:

    ```
    groupBy(userID, function=series(collect=[@rawstring], endmatch={ /Failed Login/ }))
    ```

  * **Step-by-Step**:

    1.  `groupBy(userID, function=series(...))`: Groups events by `userID`.
    2.  `series(collect=[@rawstring], endmatch={ /Failed Login/ })`: For each `userID`, it collects the `@rawstring` content into a series. A series is marked as ended upon encountering an event matching the filter `/Failed Login/`. Note that unlike `startmatch`, the `endmatch` parameter still outputs "partial" sessions that do not explicitly end with a matched event.

  * **Summary and Results**:
    This query is useful for analyzing sequences of events leading up to a failed login, providing context for troubleshooting or security investigations.