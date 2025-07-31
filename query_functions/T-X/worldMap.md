# worldMap()

The `worldMap()` function is a helper function that processes geographical data to make it compatible with the World Map widget. It can take either IP addresses or latitude/longitude coordinates as input and groups the data points using a geohashing algorithm.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **ip** | string | optional | The field containing the IP address to look up geo-coordinates for. |
| **lat** | string | optional | The field containing the latitude coordinate. |
| **lon** | string | optional | The field containing the longitude coordinate. |
| **magnitude**| aggregate function | optional <br> default: `count()` | A function used to calculate the weight of each geographic bucket. This value determines the size or opacity of the map markers. |
| **precision**| number | optional <br> default: 4 | The precision of the geohash calculation (from 1 to 12). A lower number is suitable for a global view, while a higher number is better for a small, zoomed-in area. |

***

## Example

### Create a World Map with Average Latency as Magnitude

This example demonstrates how to use the `magnitude` parameter to create a more insightful map. Instead of just counting the number of events per location, it calculates the average latency for each location and uses that value to determine the size of the map markers.

* **Query Example**
    ```
    worldMap(ip=myIpField, magnitude=avg(latency))
    ```

* **Step-by-Step**
    1.  The query starts with events that contain an IP address in `myIpField` and a `latency` measurement.
    2.  `worldMap()` takes the IP address from `myIpField` and converts it to a geographical location.
    3.  It then groups the events by their location using geohash buckets.
    4.  The `magnitude=avg(latency)` parameter tells the function to calculate the average `latency` for all events within each geographic bucket.
    5.  The resulting value is used to size the markers on the world map, with higher average latencies resulting in larger or more intense points.

* **Summary and Results**
    This query is useful for visualizing network performance and quickly spotting geographical patterns in latency. It allows you to identify regions with high latency, which could indicate network bottlenecks or other performance issues.