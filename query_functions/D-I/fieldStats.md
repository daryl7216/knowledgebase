# fieldstats()

The `fieldstats()` function displays statistics about the fields within the current event stream, such as the total count and the number of distinct values for each field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **limit** | integer | optional <br> default: 200 | Limits the number of fields to generate statistics for (min 1, max 10,000). |

***

## Function Operation

This function returns a new set of events where each event contains statistics for a field found in the original input stream. The new events will have the following fields:
* **#type**: The name of the parser used when the data was ingested.
* **field**: The name of the field these statistics are for.
* **_count**: The total number of times the field appears in the incoming event set.
* **_distinct**: The number of unique values for that field in the incoming event set.

***

## Example

### Extract Field Statistics

This example uses `fieldstats()` to get a statistical overview of all fields in the current repository.

* **Query Example**
    ```
    fieldstats()
    ```

* **Step-by-Step**
    1.  The query starts with all events in the current time range.
    2.  `fieldstats()` is called, which processes the stream and generates a new event for each field, containing its statistics.

* **Result Event Data**
    The output is a table where each row provides statistics for a single field from the original data.

| #type | field | _count | _distinct |
| :--- | :--- | :--- | :--- |
| humio | decodedContentLength | 60501 | 107 |
| humio | originalName | 154824 | 19 |
| humio | tx_fifo | 16296 | 1 |
| humio | humioClass | 1641588 | 320 |
| humio | p75 | 268658 | 42439 |
| humio | bucketId | 234418 | 2 |
| ... | ... | ... | ... |