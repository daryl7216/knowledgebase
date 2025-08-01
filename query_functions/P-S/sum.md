# sum()

The `sum()` function calculates the sum for a field over a set of events. The result is returned in a field named `sum`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `sum` | Name of output field. |
| `field` | string | required | Field to extract a number from and sum over. |

***

## Function Operation

The `sum()` function is an aggregate function that computes the total sum of all numerical values within a specified field across a given set of events. The result of this calculation is a single numerical value, which is then returned in a new output field. By default, this output field is named `_sum`, but you can customize its name using the `as` parameter. This function is frequently used in conjunction with other aggregate functions like `bucket()` or `groupBy()` to sum values over time intervals or within specific groups, respectively.

***

## Examples

### How many bytes did our webserver send per minute

This is a basic example of using `sum()` with `bucket()` to calculate the total bytes sent per minute.

  * **Query Example**
    ```
    bucket(function=sum(bytes_sent))
    ```

### Bucket Events Into Groups

This example uses the `bucket()` function to request 24 buckets over a period of one day and then sums the "count" field within each bucket.

  * **Query Example**

    ```
    bucket(buckets=24, function=sum("count"))
    parseTimestamp(field=_bucket, format=millis)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `bucket(buckets=24, function=sum("count"))`: Buckets the events into 24 groups spanning over a period of one day, using the `sum()` function on the `count` field.
    3.  `parseTimestamp(field=_bucket, format=millis)`: Extracts the timestamp from the generated `_bucket` field and converts it to a date-time value. The `bucket` function outputs the timestamp as an epoch value in the `_bucket` field. This results in an additional bucket containing all the data after the requested timespan for the requested number of buckets.
    4.  Event Result set.

  * **Summary and Results**
    The query is used to optimize data storage and query performance by making it easier to manage and locate data subsets when performing analytics tasks. The resulting outputs show 25 buckets; the original requested 24 buckets and in addition the bucket for the extracted timestamp.

  * **Result Event Data**

| bucket | \_sum | @timestamp |
| :--- | :--- | :--- |
| 1681290000000 | 1322658945428 | 1681290000000 |
| 1681293600000 | 1879891517753 | 1681293600000 |
| 1681297200000 | 1967566541025 | 1681297200000 |
| 1681300800000 | 2058848152111 | 1681300800000 |
| 1681304400000 | 2163576682259 | 1681304400000 |
| 1681308000000 | 2255771347658 | 1681308000000 |
| 1681311600000 | 2342791941872 | 1681311600000 |
| 1681315200000 | 2429639369980 | 1681315200000 |
| 1681318800000 | 2516589869179 | 1681318800000 |
| 1681322400000 | 2603409167993 | 1681322400000 |
| 1681326000000 | 2690189000694 | 1681326000000 |
| 1681329600000 | 2776920777654 | 1681329600000 |
| 1681333200000 | 2873523432202 | 1681333200000 |

---

### Calculate Total Network Bandwidth Per Host

This example calculates total inbound and outbound network traffic per host using nested `sum()` functions with `groupBy()`, then calculates the total bandwidth consumption, and finally sorts by total traffic.

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
    | 1686837828000 | NetworkConnectStats | DESKTOP-A1 | 9800000 | 5000000 |

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `event_simpleName="NetworkConnectStats"`: Filters events to include only those where `event_simpleName` equals `NetworkConnectStats`.
    3.  `groupBy([ComputerName], function=[sum(field="BytesReceived", as=InboundTraffic), sum(field="BytesSent", as=OutboundTraffic)])`: Groups the data by the `ComputerName` field and calculates two aggregate values: the sum of `BytesReceived` (stored in a field named `InboundTraffic`), and the sum of `BytesSent` (stored in a field named `OutboundTraffic`).
    4.  `TotalTraffic := InboundTraffic + OutboundTraffic`: Creates a new field named `TotalTraffic` containing the sum of `InboundTraffic` and `OutboundTraffic` values.
    5.  `sort(field="TotalTraffic", order="desc")`: Sorts the results based on the `TotalTraffic` field in descending order, showing hosts with the highest bandwidth consumption first.
    6.  Event Result set.

  * **Summary and Results**
    The query is used to analyze network bandwidth consumption patterns across different hosts in the network. This is useful for identifying hosts consuming excessive bandwidth, monitoring network usage patterns, or detecting potential network-intensive applications or anomalies. The traffic values are in bytes and each row represents the aggregated traffic for a unique host.

  * **Result Event Data**

| ComputerName | InboundTraffic | OutboundTraffic | TotalTraffic |
| :--- | :--- | :--- | :--- |
| SERVER-C3 | 285000000 | 139000000 | 424000000 |
| LAPTOP-B2 | 65000000 | 32500000 | 97500000 |
| DESKTOP-A1 | 45700000 | 24600000 | 70300000 |
| DESKTOP-D4 | 11500000 | 6000000 | 17500000 |

---

### Calculate a Percentage of Successful Status Codes Over Time

This example calculates a percentage of successful status codes over time within the `timeChart()` function, using `sum()` and `count()` within an embedded aggregate.

  * **Query Example**

    ```
    success: if(status >= 500, then=0, else=1)
    | timeChart(series=customer, function=[
        {
            [sum(success, as=success), count(as=total)]
            pct_successful := (success/total)*100
            | drop([success, total])
        }
    ], span=15m, limit=100)
    ```

  * **Step-by-Step**

    1.  Starting with the source repository events.
    2.  `success: if(status >= 500, then=0, else=1)`: Adds a `success` field: if the `status` field's value is greater than or equal to 500, `success` is set to 0 (indicating failure), otherwise it's set to 1 (indicating success).
    3.  `timeChart(series=customer, function=[{...}], span=15m, limit=100)`: Creates a new timechart, generating a new series named `customer` that uses a compound function.
          * `[sum(success, as=success), count(as=total)]`: This embedded aggregate (defined using the square bracket syntax, shorthand for `stats()`) calculates the `sum()` of the `success` field (effectively counting successful events) and the `count()` of all events (total events) within each time bucket, assigning results to fields named `success` and `total` respectively.
          * `pct_successful := (success/total)*100`: Calculates the percentage of successful events within each time bucket. At this point, the process is still within the aggregate context.
          * `| drop([success, total])`: Drops the temporary `success` and `total` fields, which were used only for calculation, from the array generated by the aggregate.
    4.  The `span=15m` sets the time bucket interval to 15 minutes, and `limit=100` limits the overall number of results from the timechart.
    5.  Event Result set.

  * **Summary and Results**
    This query shows how an embedded aggregate can be used to generate a sequence of values that can be formatted (in this case to calculate percentages) and generate a new event series for the aggregate values. It's useful for monitoring performance over time, specifically the success rate of operations based on status codes.