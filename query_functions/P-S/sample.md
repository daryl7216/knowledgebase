# sample()

The `sample()` function samples the event stream. Events that do not have the field being sampled are discarded.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | optional \<br\> default: `@timestamp` | The name of the field to use for sampling events. |
| `percentage` | double | optional \<br\> default: `1` \<br\> maximum: `100` | Keep this percentage of the events. |

***

## Function Operation

The `sample()` function is used to randomly select and retain a specified percentage of events from the event stream. This is particularly useful for analyzing characteristics of large datasets without processing every single event, thereby improving query performance and reducing resource usage.

Key aspects of its operation include:

  * Events that do not possess the field specified for sampling are discarded from the stream.
  * The `percentage` parameter (defaulting to 1%) determines what proportion of events to keep; it accepts values between \>0 and 100.
  * The `field` parameter (defaulting to `@timestamp`) indicates which field to use as the basis for sampling.

When used as part of a query, the randomly selected events are passed to the next stage of the query pipeline. Sampling can be applied before aggregations (like `groupBy()`) to analyze common patterns efficiently without hitting aggregation limits. It is useful for survey analysis (drawing conclusions without surveying all events) and for filtering both frequently and infrequently occurring events.

***

## Examples

### Sample Event Streams - Example 1

This example uses the `sample()` function to keep only a specified percentage of events.

  * **Query Example**

    ```
    sample(percentage=2)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `sample(percentage=2)`: Samples events, keeping only 2% of them. These randomly selected events are passed to the next stage of the query.
    3.  Event Result set.

  * **Summary and Results**
    The query is used to sample events, keeping only a specified percentage of them. Event sampling can be used to determine the characteristics of a large set of data without processing every event. Sampling is useful in, for example, survey analysis, making it possible to draw conclusions without surveying all events. Sampling can also be used to filter on both frequently and infrequently occurring events.

---

### Sample Event Streams - Example 2

This example uses the `sample()` function to keep a specified percentage of events and then sorts them by host using `groupBy()` and `sort()`.

  * **Query Example**

    ```
    sample(percentage=0.1)
    | groupBy(host)
    | sort()
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `sample(percentage=0.1)`: Samples events, keeping only 0.1% of them. These randomly selected events are passed to the next stage of the query.
    3.  `| groupBy(host)`: Groups the sampled events by the `host` field. The advantage of sampling events before grouping them is that it allows for analysis of common patterns without hitting `groupBy()` limits.
    4.  `| sort()`: Sorts the returned results (by default, in descending order of count) to find the most common hosts.
    5.  Event Result set.

  * **Summary and Results**
    The query is used to sample events, keeping only a specified percentage of them, and then find the most common host among the sampled events. Event sampling can be used to determine the characteristics of a large set of data without processing every event. Sampling is useful in, for example, survey analysis, making it possible to draw conclusions without surveying all events. Sampling can also be used to filter on both frequently and infrequently occurring events.