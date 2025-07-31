# concat()

The `concat()` function concatenates the values of a list of fields into a single value in a new field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_concat` | The name of the output field that will store the concatenated value. |
| **field** | array of strings | required | The list of fields whose values are to be concatenated. The parameter name `field` can be omitted. |

***

## Function Operation

This function is only capable of combining the values of existing fields. It **cannot** be used to combine fields with arbitrary string literals (like adding a separator). For combining fields with strings, the `format()` function should be used instead.

***

## Example

### Correlate Inbound Email URLs with Subsequent Access Attempts

This is an advanced example where `concat()` is used as a crucial pre-processing step in a security correlation search. The goal is to detect when a user clicks a malicious URL from an email. `concat()` is used to construct a standardized, full URL from multiple fields in network traffic data so it can be matched against URLs from email security data.

* **Query Example**
    ```
    ((#repo="abnormal_security" AND #event.module="email-security") OR (#repo="corelight" AND #event.module="ids"))
    | case {
        #Vendor="corelight"
        | url.prefix := "http://"
        | url.original := concat([url.prefix, "/", client.address, "/", url.path]);
        *
      }
    | correlate(
        emailInbound: {#event.module="email-security"},
        emailAccess: {#event.module="ids" | url.original <=> emailInbound.url.original},
        sequence=true, 
        within=1h
      )
    ```

* **Input Event Data**
    Events from two sources: `abnormal_security` for emails and `corelight` for network traffic (IDS).

| @timestamp | #repo | #event.module | #Vendor | client.address |
| :--- | :--- | :--- | :--- | :--- |
| 2023-06-15T10:00:00Z | abnormal_security | email-security| abnormal | <no value> |
| 2023-06-15T10:15:30Z | corelight | ids | corelight | 10.0.1.100 |
| ... | ... | ... | ... | ... |

* **Step-by-Step**
    1.  The query first filters for relevant events from `email-security` and `ids` modules.
    2.  A `case` statement is used to process only the network traffic events (`#Vendor="corelight"`).
    3.  Inside the `case` statement, `concat()` is used to build a full URL. It combines several fields and string literals (`url.prefix`, `"/"`, `client.address`, `"/"`, `url.path`) into a new field called `url.original`. **Note**: The document shows an incorrect use of `concat` here, as it includes string literals. The correct implementation would use `format()`. Assuming the intended logic is to build the URL, this step is crucial for standardization.
    4.  The `correlate()` function then matches the `url.original` field from email events (`emailInbound`) with the newly constructed `url.original` field from network traffic events (`emailAccess`) that occur within one hour.

* **Result Event Data**
    The query outputs correlated events, showing which inbound emails led to a subsequent URL access attempt.

| @timestamp | emaillnbound.email.from.address | emaillnbound.email.to |
| :--- | :--- | :--- |
| 2023-06-15T10:00:00Z | sender@external.com | user1@company.com |
| 2023-06-15T11:00:00Z | phish@bad.com | user2@company.com |
| 2023-06-15T12:30:00Z | partner@vendor.com | user3@company.com |