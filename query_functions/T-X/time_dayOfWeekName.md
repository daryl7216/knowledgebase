# time:dayOfWeekName()

The `time:dayOfWeekName()` function gets the English display name of the day of the week (e.g., "Monday") from a timestamp field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_dayOfWeekName` | The name of the output field. |
| **field** | string | optional <br> default: `@timestamp` | The name of the input field containing the timestamp. The parameter name `field` can be omitted. |
| **timezone** | string | optional | The time offset to use, such as `-01:00`. This will override the query's default timezone. |
| **timezoneField** | string | optional <br> default: `@timezone` | The name of a field that contains the timezone to use. This is ignored if the `timezone` parameter is also set. |

***

## Examples

There are no examples provided in the document for this function.