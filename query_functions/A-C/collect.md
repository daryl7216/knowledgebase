# collect()

The `collect()` function gathers values from one or more fields across a set of events. The collected values can be output either as a single string with a separator or as multiple rows, each with an individual value.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **fields** | array of strings | required | The name(s) of the field(s) to collect values from. The parameter name `fields` can be omitted. |
| **limit** | integer | optional <br> default: 2000 | The maximum number of distinct values to collect. |
| **multival** | boolean | optional <br> default: `true` | If `true`, outputs a single concatenated string. If `false`, outputs multiple rows with individual values. |
| **separator** | string | optional <br> default: `\n` | The separator to use between values when `multival=true`. |

***

## Function Operation

The `collect()` function has both a value limit and a memory limit. The value limit is controlled by the `limit` parameter. The memory limit is 10 MiB when `collect()` is run as a top-level function and 1 MiB when it's used within another function like `groupBy()`. If either limit is exceeded, the function returns partial results with a warning.

> **Warning**
> Collecting the `@timestamp` field directly only works when a single timestamp exists. The recommended workaround is to create a new field (e.g., `timestamp := @timestamp`) and collect that new field instead.

***

## Examples

### Collect and Group Events by Multiple Fields

This example shows how to use `collect()` within a `groupBy()` function to see all associated IPs (`aip`) for each pair of local and remote IP addresses, alongside a distinct count of those associated IPs.

* **Query Example**
    ```
    LocalAddressIP4=* RemoteAddressIP4=* aip=*
    | groupBy([LocalAddressIP4, RemoteAddressIP4], function=([count(aip, as=aipCount, distinct=true), collect([aip])]))
    ```

* **Step-by-Step**
    1.  The query first filters for events where the `LocalAddressIP4`, `RemoteAddressIP4`, and `aip` fields all exist.
    2.  `groupBy()` then groups the events by unique pairs of `LocalAddressIP4` and `RemoteAddressIP4`.
    3.  Within each group, two aggregations are performed:
        * `count()`: A distinct count of the `aip` values is calculated and stored in the `aipCount` field.
        * `collect()`: All of the `aip` values for that group are collected into a single array field named `aip`.

* **Result Event Data**
    The output shows each unique connection pair, the count of distinct associated IPs, and the full list of those IPs.

| LocalAddressIP4 | RemoteAddressIP4 | aipCount | aip |
| :--- | :--- | :--- | :--- |
| 192.168.1.100 | 203.0.113.50 | 3 | `[10.