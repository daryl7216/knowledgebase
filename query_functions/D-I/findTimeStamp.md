# findTimestamp()

The `findTimestamp()` function automatically finds and parses the first timestamp it can identify within a given field by trying a variety of common formats. It is primarily intended for use in generic parsers where the exact timestamp format may not be known in advance.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **addErrors** | boolean | optional <br> default: `true` | If `true`, adds an error field to the event if a timestamp cannot be found. |
| **as** | string | optional <br> default: `@timestamp` | The output field for the parsed timestamp, represented as milliseconds since the Unix epoch. |
| **field** | string | optional <br> default: `@rawstring` | The field to search for a timestamp in. |
| **timezone** | string | optional | Specifies the timezone (e.g., `UTC`, `America/New_York`) to use if the timestamp itself does not contain one. This is required for timestamps without timezone information. |
| **timezoneAs**| string | optional <br> default: `@timezone` | The output field for the parsed timezone. |

***

## Function Operation

The function searches for a timestamp only within the first 128 characters of the specified field. If the exact format of your timestamps is known, it is better to use the `parseTimestamp()` function.

### Supported Formats
The function attempts to match against several formats, including:
* `year month day hour minute second`
* `month day [year] hour minute second`
* `epochsecond [subsecond]`
* And others, with optional subseconds and timezones.

### Key Behaviors
* **Missing Timezone**: If a timestamp does not include a timezone, the `timezone` parameter must be provided, otherwise the timestamp will not be parsed.
* **Missing Date**: If the date is missing, the current date is used.
* **Missing Year**: If the year is missing, the system infers the year to be the most recent one that keeps the date within 7 days of the present.
* **Two-Digit Years**: Two-digit years are assumed to be between 2013 and 2099.

> **Note**
> For timestamps in a time zone with Daylight Saving Time, it is recommended to specify the `timezone` parameter as an offset (e.g., `UTC+02:00`) to avoid ambiguity during the switch between DST and Standard Time.

***

## Examples

The document provides several syntax examples demonstrating different parameter combinations.

### Set a Default Timezone

This is used in a parser when the incoming timestamps are known to be from a specific timezone but do not include that information in the string itself.