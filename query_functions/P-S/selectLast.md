# selectLast()

The `selectLast()` function specifies a set of fields to select from events; for each field it will keep the field value of the most recent event with that field. This can be used to collect field values across a range of events, where each event contributes one or more fields to the output event. It is usually most useful in combination with `groupBy()`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `fields` | array of strings | required | The names of the fields to keep. |

***

## Function Operation

The `selectLast()` function identifies and extracts the values of specified fields from the most recent event (based on timestamp) within a group or across the entire event stream. Its primary purpose is to consolidate information by retaining only the latest state or value for each unique combination of grouping keys.

When used, `selectLast()` processes events and, for each field listed in its `fields` parameter, it takes the value from the event with the latest timestamp where that field is present. If multiple events share the same latest timestamp, one is chosen non-deterministically. This function is particularly effective when combined with `groupBy()`, as it allows you to retrieve the most up-to-date attributes for each distinct group (e.g., the latest status for each server ID).

It's important to note the equivalences:

  * `selectLast([@timestamp])` is equivalent to `max(@timestamp)`.
  * The logical opposite (selecting the first matching event) for `@timestamp` can be achieved using `min(@timestamp)`. For arbitrary arrays of values, achieving the opposite of `selectLast([from, to])` would require a more complex query involving `head(1)` for each field.

***

## Examples

### Find the first value of a field `x` (and when that value was from)

This example demonstrates how `selectLast()` works with `@timestamp` to find the most recent value and its timestamp.

  * **Query Example**

    ```
    selectLast(@timestamp, include=[x, @timestamp])
    ```

  * **Step-by-Step**

    1.  This query uses `@timestamp` as the field to find the maximum value, effectively sorting by time to get the most recent event.
    2.  It then returns the `x` field and the `@timestamp` of that most recent event.

  * **Summary and Results**
    This selects the event with the minimum value of `@timestamp` that also contains the specified field `x`, and returns an event with fields `@timestamp` and `x` only.

---

### List All EC2 Hosts With FirstSeen Data Within 14 Days

This example retrieves the latest information about AWS EC2 instances running CrowdStrike sensors, showing their platform, hostname, agent version, and when they were first seen, with a 14-day reference point for age comparison.

  * **Query Example**

    ```
    #repo=sensor_metadata #data_source_name=aidmaster cloud.provider = "AWS_EC2_V2"
    | groupBy([aid], function=(selectLast([event_platform, aid, ComputerName, AgentVersion, FirstSeen])), limit=max)
    | FirstSeen := formatTime("%FT%T%z", field=FirstSeen)
    | TimeDelta := now() - duration("14d")
    ```

  * **Input Event Data**

    | @timestamp | aid | cloud.provider | event\_platform | ComputerName | AgentVersion | FirstSeen |
    | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
    | 2025-05-20T10:00:00Z | 1234abcd | AWS\_EC2\_V2 | Windows | ec2-web-01 | 6.45.15679 | 2025-05-20T10:00:00Z |
    | 2025-05-21T11:00:00Z | 1234abcd | AWS\_EC2\_V2 | Windows | ec2-web-01 | 6.45.15679 | 2025-05-20T10:00:00Z |
    | 2025-05-22T12:00:00Z | 5678efgh | AWS\_EC2\_V2 | Linux | ec2-app-02 | 6.45.15679 | 2025-05-22T12:00:00Z |
    | 2025-05-23T13:00:00Z | 5678efgh | AWS\_EC2\_V2 | Linux | ec2-app-02 | 6.45.15679 | 2025-05-22T12:00:00Z |
    | 2025-05-24T14:00:00Z | 90123ijk | AWS\_EC2\_V2 | Windows | ec2-db-03 | 6.45.15679 | 2025-05-24T14:00:00Z |
    | 2025-05-25T15:00:00Z | 90123ijk | AWS\_EC2\_V2 | Windows | ec2-db-03 | 6.45.15679 | 2025-05-24T14:00:00Z |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `#repo=sensor_metadata #data_source_name=aidmaster cloud.provider = "AWS_EC2_V2"`: Searches in the `SENSOR_METADATA` repository and filters for `#data_source_name` fields containing the value `aidmaster`, looking for `cloud.provider` of the type `AWS_EC2_V2` only.
    3.  `| groupBy([aid], function=(selectLast([event_platform, aid, ComputerName, AgentVersion, FirstSeen])), limit=max)`: Groups results by the field `aid` (Agent ID). Then, for each unique group, it selects the most recent values for the fields: `event_platform`, `aid`, `ComputerName`, `AgentVersion`, `FirstSeen`. Using `selectLast()` within `groupBy()` is what actually selects the most recent record for each group.
    4.  `| FirstSeen := formatTime("%FT%T%z", field=FirstSeen)`: Formats the timestamp in the `FirstSeen` field into ISO 8601 format. The result is stored back in the `FirstSeen` field.
    5.  `| TimeDelta := now() - duration("14d")`: Calculates a timestamp from 14 days ago and returns the results into a new field named `TimeDelta`. The calculation is done by subtracting a 14-day duration from the current time using `duration()`. This new `TimeDelta` field represents a timestamp from 14 days ago and can be used for filtering or comparing against the `FirstSeen` timestamps.
    6.  Event Result set.

  * **Summary and Results**
    The query is used to retrieve the latest information about AWS EC2 instances running CrowdStrike sensors, showing their platform, hostname, agent version, and when they were first seen, with a 14-day reference point for age comparison. The query is useful, for example, for auditing EC2 instance coverage, identifying newly added EC2 instances within the last two weeks, monitoring sensor versions, or identifying aging or outdated installations. Each `aid` appears only once with its most recent values.

  * **Result Event Data**

    | aid | event\_platform | ComputerName | AgentVersion | FirstSeen |
    | :--- | :--- | :--- | :--- | :--- |
    | 1234abcd | Windows | ec2-web-01 | 6.45.15679 | 2025-05-21T11:00:00Z |
    | 5678efgh | Linux | ec2-app-02 | 6.45.15679 | 2025-05-23T13:00:00Z |
    | 90123ijk | Windows | ec2-db-03 | 6.45.15679 | 2025-05-25T15:00:00Z |