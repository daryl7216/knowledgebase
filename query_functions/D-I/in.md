# in()

The `in()` query function is a filter used to select events where a given field's value matches any of the values in a specified list.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **field** | string | required | The field on which to filter events. The parameter name `field` can be omitted. |
| **ignoreCase** | boolean | optional <br> default: `false` | If set to `true`, the search will be case-insensitive. |
| **values** | array of strings | required | The list of values to match against the field. Only one match is required. Values can contain wildcards (`*`). |

***

## Function Operation

This function checks if the value of the specified `field` is present in the provided `values` list. It acts as an "OR" condition for multiple values. It can be negated by prefixing the call with an exclamation mark (e.g., `!in(...)`) to find events where the field value is *not* in the list.

***

## Examples

### Categorize Errors in Log Levels

This example uses `in()` as the condition within an `if()` function to create a new field that categorizes events based on their status code.

* **Query Example**
    ```
    critical_status := if(in(status, values=["500", "404"]), then="Critical", else="Non-Critical")
    ```

* **Input Event Data**
    ```
    srcIP=192.168.1.5 loglevel=ERROR status=404 user=admin
    srcIP=10.0.0.1 loglevel=INFO status=200 user=user1
    srcIP=192.168.1.15 loglevel=ERROR status=500 user=admin
    ```

* **Step-by-Step**
    1.  The query creates a new field named `critical_status`.
    2.  The `if()` function's condition uses `in()` to check if the `status` field contains either "500" or "404".
    3.  If the condition is true, `critical_status` is set to "Critical".
    4.  Otherwise, it is set to "Non-Critical".

* **Result Event Data**
    The output shows the original events enriched with the new `critical_status` category.

| srcIP | loglevel | status | user | critical_status |
| :--- | :--- | :--- | :--- | :--- |
| 192.168.1.5 | ERROR | 404 | admin | Critical |
| 10.0.0.1 | INFO | 200 | user1 | Non-Critical |
| 172.16.0.5 | WARN | 422 | user2 | Non-Critical |
| 192.168.1.15| ERROR | 500 | admin | Critical |
| 10.0.0.12 | DEBUG | 302 | user1 | Non-Critical |

---

### Categorize Events Based on Values in Multiple Fields

This advanced example uses a `case` statement with both `in()` and its negated form `!in()` to apply a sequence of rules for categorizing events into a `type` field.

* **Query Example**
    ```
    case {
      in(srcIP, values=["192.168.1.*"]) | type := "Internal";
      !in(loglevel, values=["DEBUG", "INFO"]) | type := "Critical";
      * | type := "Other"
    }
    ```
* **Input Event Data**
    ```
    srcIP=192.168.1.5 loglevel=ERROR status=404 user=admin
    srcIP=10.0.0.1 loglevel=INFO status=200 user=user1
    srcIP=172.16.0.5 loglevel=WARN status=422 user=user2
    ```

* **Step-by-Step**
    The `case` statement evaluates conditions in order:
    1.  It first checks if the `srcIP` starts with `192.168.1.`. If it does, the `type` is set to "Internal" and the statement stops for that event.
    2.  If the first condition is false, it checks if the `loglevel` is *not* "DEBUG" or "INFO". If true, `type` is set to "Critical".
    3.  If neither of the first two conditions is met, the wildcard `*` acts as a default case, setting `type` to "Other".

* **Result Event Data**
    The query assigns a `type` to each event based on the prioritized logic.

| srcIP | loglevel | status | user | type |
| :--- | :--- | :--- | :--- | :--- |
| 192.168.1.5 | ERROR | 404 | admin | Internal |
| 10.0.0.1 | INFO | 200 | user1 | Other |
| 172.16.0.5 | WARN | 422 | user2 | Critical |
| 192.168.1.15| ERROR | 500 | admin | Internal |
| 10.0.0.12 | DEBUG | 302 | user1 | Other |

---

### Differentiate Between Types of Log Levels

This example uses `in()` within a `match` statement to create a `severity` field based on the `loglevel`.

* **Query Example**
    ```
    loglevel match {
      /.*ERROR.*/ => severity := "High";
      in(values=["DEBUG", "INFO"]) => severity := "Low";
      * => severity := "Medium"
    }
    ```

* **Step-by-Step**
    The `match` statement checks the `loglevel` field:
    1.  If it contains the word "ERROR", `severity` is set to "High".
    2.  If not, it checks if the `loglevel` is `in` the list `["DEBUG", "INFO"]`. If so, `severity` is set to "Low".
    3.  If neither matches, the default case sets `severity` to "Medium" (this would catch levels like "WARN").

* **Result Event Data**
    Each event gets a `severity` rating based on its log level.

| srcIP | loglevel | status | user | severity |
| :--- | :--- | :--- | :--- | :--- |
| 192.168.1.5 | ERROR | 404 | admin | High |
| 10.0.0.1 | INFO | 200 | user1 | Low |
| 172.16.0.5 | WARN | 422 | user2 | Medium |
| 192.168.1.15| ERROR | 500 | admin | High |
| 10.0.0.12 | DEBUG | 302 | user1 | Low |