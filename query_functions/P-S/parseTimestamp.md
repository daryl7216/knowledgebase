# parseTimestamp()

The `parseTimestamp()` function parses a string into a timestamp. This function is important for creating parsers, as it is used to parse the timestamp for an incoming event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `addErrors` | boolean | optional \<br\> default: `true` | Whether to add an error field `_tc_error` to the event if it was not possible to parse the timestamp. |
| `as` | string | optional \<br\> default: `@timestamp` | Name of output field that will contain the timestamp. The timestamp is represented in milliseconds since January 1, 1970 in UTC. LogScale expects to find the timestamp in the field `@timestamp` so do not change when creating parsers. |
| `caseSensitive` | boolean | optional \<br\> default: `true` | Whether the timestamp format pattern is case sensitive. For example, the format `LLL` will accept `Feb` but not `feb` in case sensitive mode, while `feb` will be accepted in case insensitive mode. \<br\>**Values:**\<br\>`false`: Pattern is not case sensitive. \<br\>`true`: Pattern is case sensitive. |
| `field` | string | required | The field holding the timestamp to be parsed. |
| `format` | string | optional \<br\> default: `yyyy-MM-dd'T'HH:mm:ss[.SSSSSSSSS]XXXXX` | Pattern used to parse the timestamp. If a format string is not specified, it is inferred using `java.time.format.DateTimeFormatter` or one of the special format specifiers. The special format specifiers are case-insensitive. \<br\>**Values:**\<br\>`millis` or `milliseconds`: Epoch time in milliseconds. \<br\>`nanos`: Epoch time in nanoseconds. \<br\>`seconds` or `unixTimeSeconds` or `unixtime`: Epoch time in seconds (UTC). |
| `timezone` | string | optional | If the timestamp does not contain a timezone, it can be specified using this parameter. Examples are `Europe/London`, `America/New_York`, and `UTC`. See the full list of timezones supported by LogScale at Supported Time Zones. If the timestamp does not contain a timezone, and no timezone is specified here, an error is generated. If the timezone specified here and one also exists in the timestamp, then this parameter overrides the timezone in the event. |
| `timezoneAs` | string | optional \<br\> default: `@timezone` | Name of output field that will contain the timezone. LogScale expects to find the timezone in the field `@timezone` so do not change when creating parsers. |

***

## Function Operation

The `parseTimestamp()` function is crucial for converting string representations of time into standardized timestamps within LogScale. Before parsing, the timestamp part of the log should ideally be captured in a field, usually during parsing, but it can also be extracted during queries using functions like `regex()` and `parseJson()` before `parseTimestamp()`.

This function formats times using a subset of Java's `DateTimeFormatter`. It also supports special format strings such as `seconds`, `milliseconds`, `nanos`, `unixtime`, `unixtimeMillis`, and `unixTimeSeconds` for epoch time conversions. For formats specifying seconds, it can also handle milliseconds using floating-point numbers (e.g., `1690480444.589` for 2023-07-27 19:54:04 and 589 milliseconds). Nanosecond precision is also supported, and the nanosecond component will be extracted.

After parsing, a field named `@timestamp` is created, containing the parsed timestamp in UTC milliseconds, and a `@timezone` field containing the original timezone.

`parseTimestamp()` can also parse time formats that omit the year designator, often seen in Syslog. For instance, `Mar 15 07:48:13` can be parsed using the format `MMM dd HH:mm:ss`. In such cases, LogScale infers the year: if the date is less than 8 days into the future or in the past, the current year is used; otherwise, if it's more than 8 days into the future, the previous year is used.

If the input timestamp string does not contain timezone information, the `timezone` parameter **must** be specified, otherwise an error will be generated. If a timezone is specified both in the parameter and the timestamp, the parameter's value overrides the timestamp's timezone. The `addErrors` parameter controls whether an error field `_tc_error` is added if parsing fails. The `caseSensitive` parameter determines if the format pattern matching is case-sensitive.

***

## Examples

### Extract a timestamp that is using millisecond precision embedded in a JSON value

This example demonstrates how to extract and parse a timestamp with millisecond precision from a JSON field.

  * **Query Example**

    ```
    parseJson()
    parseTimestamp("millis", field=timestamp)
    ```

  * **Step-by-Step**

    1.  The `parseJson()` function is used first to parse the raw event string, assuming it contains JSON data with a `timestamp` field.
    2.  `parseTimestamp("millis", field=timestamp)` then takes the `timestamp` field (extracted by `parseJson()`) and parses it as an epoch time in milliseconds.

---

### Parse timestamps in an accesslog where the timestamp includes an explicit timezone offset

This example shows parsing a timestamp from an access log that includes an explicit timezone offset.

  * **Query Example**

    ```
    /(?<client>\S+) \[(?<@timestamp>.+)\] (?<method>\S+) (?<url>\S+)/
    parseTimestamp("dd/MMM/yyyy:HH:mm:ss Z", field=@timestamp)
    ```

  * **Input Event Data**
    `192.168.1.19 [02/Apr/2014:16:29:32 +0200] GET /hello/test/123`

  * **Step-by-Step**

    1.  `/(?<client>\S+) \[(?<@timestamp>.+)\] (?<method>\S+) (?<url>\S+)/`: This regex extracts various fields from the log line, including the timestamp into `@timestamp`.
    2.  `parseTimestamp("dd/MMM/yyyy:HH:mm:ss Z", field=@timestamp)`: Parses the extracted `@timestamp` field using the specified format string, which includes a timezone offset (`Z`).

---

### Parse an event with a timestamp not containing year

This example demonstrates parsing a timestamp that does not include the year, commonly found in Syslog entries, by specifying a timezone.

  * **Query Example**

    ```
    /(?<timestamp>\S+\s+\d+\s+\d+:\d+:\d+)/
    parseTimestamp("MMM dd HH:mm:ss", field=timestamp, timezone="Europe/London")
    ```

  * **Input Event Data**
    (Assume an event with a timestamp like): `Feb 9 12:22:44 hello world`

  * **Step-by-Step**

    1.  `/(?<timestamp>\S+\s+\d+\s+\d+:\d+:\d+)/`: This regex captures the timestamp part (e.g., `Feb 9 12:22:44`) into a field named `timestamp`.
    2.  `parseTimestamp("MMM dd HH:mm:ss", field=timestamp, timezone="Europe/London")`: Parses the `timestamp` field. Since the year is not present in the timestamp string, the `timezone` parameter is used to provide the necessary context for correct parsing. LogScale will infer the year based on the current date and the 8-day rule.

---

### Bucket Events Into Groups

This example uses `bucket()` and `parseTimestamp()` to group events into 24 buckets and then parse the generated bucket timestamps.

  * **Query Example**

    ```
    bucket(buckets=24, function=sum("count"))
    parseTimestamp(field=_bucket, format=millis)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `bucket(buckets=24, function=sum("count"))`: Buckets the events into 24 groups spanning over a period of one day, using the `sum()` function on the `count` field. The `bucket` function outputs the timestamp as an epoch value in the `_bucket` field.
    3.  `parseTimestamp(field=_bucket, format=millis)`: Extracts the timestamp from the generated `_bucket` field and converts this epoch timestamp (in milliseconds) to a date-time value.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to optimize data storage and query performance by making it easier to manage and locate data subsets when performing analytics tasks. The resulting output shows 25 buckets; the original requested 24 buckets and in addition the bucket for the extracted timestamp.

  * **Result Event Data**

| \_bucket | sum | @timestamp |
| :--- | :--- | :--- |
| 1681290000000 | 1322658945428 | 1681290000000 |

---

### Parse Timestamp Without Timezone Information

This example converts local time strings to timestamps with timezone specification using the `parseTimestamp()` function with the `timezone` parameter.

  * **Query Example**

    ```
    parseTimestamp("yyyy-MM-dd'T'HH:mm:ss", field=event_time, timezone="America/New_York")
    ```

  * **Input Event Data**

    | event\_time | action | user |
    | :--- | :--- | :--- |
    | 2023-05-02T10:30:00 | login | jsmith |
    | 2023-05-02T10:35:00 | logout | jsmith |
    | 2023-05-02T10:40:00 | login | awhite |
    | 2023-05-02T10:45:00 | update | awhite |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `parseTimestamp("yyyy-MM-dd'T'HH:mm:ss", field=event_time, timezone="America/New_York")`: Parses the timestamp string in `event_time` using the specified format pattern. The `timezone` parameter is set to `America/New_York` to properly interpret the local time. The result is stored in a new field named `@timestamp` (and the timezone information in `@timezone`). Note that if the timestamp string does not contain a timezone, then one must be specified using the `timezone` parameter, otherwise an error is generated.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to convert local timestamp strings into properly formatted timestamps with timezone information. This query is useful, for example, to standardize timestamp fields in logs that contain local time information without explicit timezone data.

  * **Result Event Data**

| @timezone | action | event\_time | user |
| :--- | :--- | :--- | :--- |
| America/New\_York | login | 2023-05-02T10:30:00 | jsmith |
| America/New\_York | logout | 2023-05-02T10:35:00 | jsmith |
| America/New\_York | login | 2023-05-02T10:40:00 | awhite |
| America/New\_York | update | 2023-05-02T10:45:00 | awhite |