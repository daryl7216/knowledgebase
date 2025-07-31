# geohash()

The `geohash()` function calculates a geohash value from two fields that represent latitude and longitude coordinates.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_geohash` | The name of the field that will be created to store the geohash value. |
| **lat** | string | optional <br> default: `ip.lat` | The field to use for the latitude coordinate. |
| **lon** | string | optional <br> default: `ip.lon` | The field to use for the longitude coordinate. |
| **precision**| number | optional <br> default: 12 | The precision to use for the calculation. |

***

## Example

### Calculate the Geohash Value for a Set of Coordinates

This example takes the latitude and longitude for London and calculates the corresponding geohash value. The latitude and longitude coordinates must be in decimal degrees.

* **Query Example**
    ```
    londonLat := 51.507222
    | londonLon := -0.1275
    | geohash(lat=londonLat, lon=londonLon)
    ```

* **Step-by-Step**
    1.  The query first creates two fields, `londonLat` and `londonLon`, with the respective coordinates for London.
    2.  The `geohash()` function is then called, using `londonLat` for the latitude and `londonLon` for the longitude.
    3.  The result is stored in a new field named `_geohash` by default.

* **Result Event Data**

| _geohash | londonLat | londonLon |
| :--- | :--- | :--- |
| gcpvj0e5m415 | 51.507222 | -0.1275 |