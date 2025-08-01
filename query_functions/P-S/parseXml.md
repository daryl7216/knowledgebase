# parseXml()

The `parseXml()` function parses data as XML. Specify `field=@rawstring` to parse the `@rawstring` into XML. If the specified field does not exist, the event is skipped. If the specified field exists but contains non-XML data, the behavior depends on the `strict` parameter.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `field` | string | required \<br\> default: `@rawstring` | Field that should be parsed as XML. |
| `prefix` | string | optional | Prefix the name of the fields extracted from XML with the value of this parameter. |
| `strict` | boolean | optional \<br\> default: `false` | Specifies if events where the field does not contain valid XML should be filtered out of the result set. \<br\>**Values:**\<br\>`false`: All events are passed, including an error flag describing why the event parsing failed. \<br\>`true`: Events that do not contain valid XML will be filtered in the result set. |

***

## Function Operation

The `parseXml()` function is used to parse XML-formatted data from a specified field into new, structured fields within an event. By default, it attempts to parse the `@rawstring` field.

The behavior of `parseXml()` depends on the validity of the XML data and the `strict` parameter:

  * If the `field` specified does not exist in an event, that event is simply skipped.
  * If the `field` exists but contains data that is not valid XML:
      * When `strict` is `false` (the default), all events are passed through. However, an internal error flag will be set on events where parsing failed, describing why the XML parsing was unsuccessful.
      * When `strict` is `true`, any events where the specified field does not contain valid XML data will be filtered out from the result set, meaning they will not appear in the output.

You can also use the `prefix` parameter to add a custom prefix to the names of all fields extracted from the XML, which helps in organizing data and avoiding naming conflicts.

***

## Examples

### Parse entire event as XML and extract timestamp

This example demonstrates parsing an entire event that is formatted as XML and then extracting and parsing a timestamp field from the XML.

  * **Query Example**

    ```
    parseXml()
    parseTimestamp(field=timestamp)
    ```

  * **Input Event Data**

    ```xml
    <log>
        <service>userService</service>
        <timestamp>2017-12-18T20:39:35Z</timestamp>
        <msg>user with id=47 logged in</msg>
    </log>
    ```

  * **Step-by-Step**

    1.  `parseXml()`: Parses the entire `@rawstring` (assuming the whole event is XML) into structured fields. This will create fields such as `log.service`, `log.timestamp`, and `log.msg`.
    2.  `parseTimestamp(field=timestamp)`: Takes the `timestamp` field (which would be `log.timestamp` after `parseXml()` if the `log` element is the root, or directly `timestamp` if no root element hierarchy is applied) and converts its string value into a proper timestamp.

---

### Parse XML embedded within a field after `kvParse()`

This example shows how to extract an XML string from a field using `kvParse()` and then parse that extracted XML.

  * **Query Example**

    ```
    /(?<timestamp>\S+)/
    parseTimestamp(field=timestamp)
    | kvParse()
    | parseXml(field=details)
    ```

  * **Input Event Data**
    (A log line that might look like this, where `details` contains XML):
    `2017-12-18T20:39:35Z user id=47 logged in details="<log><service>userService</service><timestamp>2017-12-18T20:39:35Z</timestamp><msg>user with id=47 logged in</msg></log>"`

  * **Step-by-Step**

    1.  `/(?<timestamp>\S+)/`: Extracts the initial timestamp from the raw event into a field named `timestamp`.
    2.  `parseTimestamp(field=timestamp)`: Parses this `timestamp` field into a structured timestamp.
    3.  `| kvParse()`: Parses the raw event string into key-value pairs, which would extract `details` as a field containing the XML string.
    4.  `| parseXml(field=details)`: Takes the `details` field (extracted by `kvParse()`) and parses its content as XML, converting the XML elements into new fields within the event (e.g., `log.service`, `log.timestamp`, `log.msg`).