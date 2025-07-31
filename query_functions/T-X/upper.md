# upper()

The `upper()` function converts text from an event field to uppercase letters. It can use either the system's default locale or a specific locale if provided.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_upper` | The name of the output field. |
| **field** | string | required | The name of the input field with the value to convert to uppercase. The parameter name `field` can be omitted. |
| **locale** | string | optional <br> default: system locale | The locale to use for conversion, specified as an ISO 639 language and ISO 3166 country (e.g., `en`, `en_US`). |

***

## Function Operation

This function reads the string value from the specified `field` and outputs an uppercase version. Specifying the correct `locale` is important for languages with non-Latin alphabets to ensure correct character conversion. For example, you can specify `en_US` for US English or `en_UK` for UK English.

***

## Examples

### Correlate Two Scheduled Task Events

This complex example uses `upper()` to standardize task names before correlating task registration and deletion events. This ensures that a match will be found even if the casing of the task name is different in the two event types.

* **Query Example**
    ```
    correlate(
      ScheduledTaskRegistered: {
        #repo="base_sensor" #event_simpleName=ScheduledTaskRegistered
        | upper(field=TaskName, as=scheduledTaskName)
      },
      ScheduledTaskDeleted: {
        #repo="base_sensor" #event_simpleName=ScheduledTaskDeleted
        | upper(field=TaskName, as=scheduledTaskName)
        | aid <=> ScheduledTaskRegistered.aid
        | scheduledTaskName <=> ScheduledTaskRegistered.scheduledTaskName
      },
      sequence=false, within=5m)
    ```

* **Input Event Data**

| #event_simpleName | aid | TaskName |
| :--- | :-- | :--- |
| ScheduledTaskRegistered | aid123| backup_task |
| ScheduledTaskDeleted | aid123| backup_task |
| ScheduledTaskRegistered | aid456| cleanup_task|
| ScheduledTaskDeleted | aid456| cleanup_task|

* **Step-by-Step**
    1.  The `correlate()` function defines two subqueries, `ScheduledTaskRegistered` and `ScheduledTaskDeleted`.
    2.  In both subqueries, `upper(field=TaskName, as=scheduledTaskName)` is used to convert the `TaskName` to uppercase and store it in a new field, `scheduledTaskName`.
    3.  The correlation then matches events from both subqueries where the `aid` and the standardized `scheduledTaskName` are identical, and the events occurred within a 5-minute window.

* **Result Event Data**
    The output shows correlated events, demonstrating the complete lifecycle of a task within the specified timeframe.

| @timestamp | ScheduledTaskRegistered.aid | ScheduledTaskRegistered.scheduledTaskName |
| :--- | :--- | :--- |
| 2023-06-15T10:00:00Z | aid123 | BACKUP_TASK |
| 2023-06-15T10:05:00Z | aid456 | CLEANUP_TASK |

---

### Standardize and Combine Fields for Analysis

This example demonstrates a full data preparation pipeline where `upper()` and `lower()` are used to standardize fields, which are then combined and aggregated to find the most common patterns.

* **Query Example**
    ```
    lower(#severity, as=severity)
    | upper(#category, as=category)
    | concat([severity, category], as=test)
    | top(test)
    ```

* **Step-by-Step**
    1.  `lower(#severity, as=severity)`: Converts the `#severity` field to all lowercase.
    2.  `upper(#category, as=category)`: Converts the `#category` field to all uppercase.
    3.  `concat([severity, category], as=test)`: Combines the newly standardized `severity` and `category` fields into a single field named `test`.
    4.  `top(test)`: Finds the most common combinations of severity and category and provides a count for each.

* **Result Event Data**
    The final output is a table showing the most frequent combinations of severity and category, with standardized casing for easy grouping and analysis.

| test | _count |
| :--- | :--- |
| infoALERT | 90005 |
| infoFILTERALERT | 36640 |
| errorALERT | 17256 |
| warningGRAPHQL | 14240 |
| ... | ... |