# time:month()

The `time:month()` function gets the month from a timestamp field, returning a number from 1 to 12.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_month` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.