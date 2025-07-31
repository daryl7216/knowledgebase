# time:weekOfYear()

The `time:weekOfYear()` function gets the week number of a timestamp, returning a value from 1 to 53.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_weekOfYear` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Function Operation

This function calculates the week number according to the ISO 8601 standard, where weeks start on a Monday and are assigned to the year that contains the Thursday of that week.

***

## Examples

There are no examples provided in the document for this function.