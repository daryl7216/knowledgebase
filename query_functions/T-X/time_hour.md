# time:hour()

The `time:hour()` function gets the hour of the day (0-23) from a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_hour` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Example

### Summarize Data Events by Hour

This example uses `time:hour()` to extract the hour from the ingest timestamp of events and then uses `groupBy()` to count how many events occurred in each hour.

* **Query Example**
    ```
    hr := time:hour(field="@ingesttimestamp")
    | groupBy(hr)
    ```

* **Step-by-Step**
    1.  `hr := time:hour(field="@ingesttimestamp")`: This step extracts the hour (0-23) from the `@ingesttimestamp` of each event and stores it in a new field named `hr`.
    2.  `groupBy(hr)`: The events are then grouped by the `hr` field, and a count of events for each hour is calculated.

* **Summary and Results**
    The query is used to get a count of data events per hour. The final result is a table showing each hour of the day and the total number of events that were ingested during that hour, which can then be plotted on a bar chart.