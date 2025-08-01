# parseLEEF()

The `parseLEEF()` function parses Log Event Extended Format (LEEF) encoded messages. Only LEEF versions 1.0 and 2.0 are supported.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `delimiter` | string | optional | Delimiter to use when parsing the extension fields of a LEEF:1.0 message. When processing LEEF:2.0 messages this argument is ignored as the value is specified in a message header. The value must be a single character with unicode value in the range 0x0000-0xffff. |
| `field` | string | optional \<br\> default: `@rawstring` | Field that holds the input in LEEF form. |
| `headerprefix` | string | optional \<br\> default: `leef.` | Prefix to the field names for the header fields. |
| `keeplabels` | boolean | optional \<br\> default: `false` | Keep short form label fields (LEEF:1.0 only), which is interpreted as CEF. |
| `labelprefix` | string | optional \<br\> default: `leef.label` | Prefix to extension fields with labels (LEEF:1.0 only), which is interpreted as CEF. |
| `parsetime` | boolean | optional \<br\> default: `true` | Control if the `devTime` extension field should be parsed. Setting this to `false` can be used to disable the time parsing step. \<br\>**Values:**\<br\>`false`: Disables parsing of the `devTime` field. \<br\>`true`: Enables parsing of the `devTime` field. |
| `prefix` | string | optional \<br\> default: `leef.ext.` | Prefix to extension fields. Fields in the LEEF extension part are prefixed with this. |
| `timezone` | string | optional \<br\> default: `Z` | Time zone to use if none specified in `devTimeFormat` string. |

***

## Function Operation

The `parseLEEF()` function is used to parse Log Event Extended Format (LEEF) messages, specifically supporting versions 1.0 and 2.0. This function will automatically skip any prefix up to the `LEEF:1.0` or `LEEF:2.0` marker. This means that if LEEF messages are delivered via Syslog, the Syslog portion of the message needs to be parsed separately.

If the LEEF message contains a `devTime` (and optionally `devTimeFormat`), this function will extract the timestamp from there and assign it to `@timestamp`. You can control whether `devTime` is parsed using the `parsetime` parameter.

The `field` parameter (defaulting to `@rawstring`) specifies the input field containing the LEEF message. You can customize the prefixes added to extracted field names: `headerprefix` for header fields (default `leef.`), `labelprefix` for labeled extension fields in LEEF 1.0 (default `leef.label`), and `prefix` for general extension fields (default `leef.ext.`). The `keeplabels` parameter (LEEF 1.0 only) controls whether short-form label fields are kept. The `delimiter` parameter is used for parsing LEEF 1.0 extension fields. For LEEF 2.0, the delimiter is specified within the message header itself, so this argument is ignored. The `timezone` parameter allows specifying a default timezone if none is provided in the `devTimeFormat` string.

In general, the implementation is more permissive than the official LEEF specification to allow for common mistakes in log writers. Notably, a `LEEF:0` marker is parsed as LEEF 1.0 headers with a CEF:0-style body for legacy support.

***

## Examples

### Parse the @rawstring field to show how the LEEF format is parsed with the default field names

This example demonstrates parsing a LEEF 2.0 log line with default field naming.

  * **Query Example**

    ```
    parseLEEF(field=@rawstring)
    ```

  * **Input Event Data**
    (From a log line like this:)
    `<13>1 2019-01-18T11:07:53.520Z 192.168.1.1 LEEF:2.0|Lancope|StealthWatch|1.0|41|^|src=10.0.0.1^act=blocked an X^dst=1.1.1.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| leef.version | 2.0 |
| leef.device.vendor | Lancope |
| leef.device.product | StealthWatch |
| leef.device.version | 1.0 |
| leef.event id | 41 |
| leef.ext.src | 10.0.0.1 |
| leef.ext.act | "blocked an X" |
| leef.ext.dst | 1.1.1.1 |

---

### Show how the LEEF format is parsed with changed header prefix

This example demonstrates parsing a LEEF 2.0 log line with a custom header prefix.

  * **Query Example**

    ```
    parseLEEF(headerprefix="leef.header.")
    ```

  * **Input Event Data**
    (From a log line like this:)
    `LEEF:2.0 Palo Alto Networks|LF|2.0|THREAT||TimeReceived=2016-01-01T00:00:58.000000Z DeviceSN=123456789123 EventID=THREAT cat= ConfigVersion=9.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| leef.header.version | 2.0 |
| leef.header.device.vendor | Palo Alto Networks |
| leef.header.device.product | LF |
| leef.header.device.version | 2.0 |
| leef.ext.DeviceSN | 123456789123 |
| leef.ext.EventID | THREAT |
| leef.ext.TimeReceived | 2016-01-01T00:00:58.000000Z |
| leef.ext.cat |  |
| leef.ext.ConfigVersion | 9.1 |

---

### Show how the LEEF format is parsed with changed extension prefix

This example demonstrates parsing a LEEF 2.0 log line with a custom extension prefix.

  * **Query Example**

    ```
    parseLEEF(prefix="my.prefix.")
    ```

  * **Input Event Data**
    (From the same log line as the previous example):
    `LEEF:2.0 Palo Alto Networks|LF|2.0|THREAT||TimeReceived=2016-01-01T00:00:58.000000Z DeviceSN=123456789123 EventID=THREAT cat= ConfigVersion=9.1`

  * **Result Event Data**

| Field | Value |
| :--- | :--- |
| leef.version | 2.0 |
| leef.event id | THREAT |
| leef.device.vendor | Palo Alto Networks |
| leef.device.product | 2LF |
| leef.device.version | 2.0 |
| my.prefix.DeviceSN | 2123456789123 |
| my.prefix.EventID | THREAT |
| my.prefix.TimeReceived | 2016-01-01T00:00:58.0000002 |
| my.prefix.cat |  |
| my.prefix.ConfigVersion | 9.1 |