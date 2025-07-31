# xml:prettyPrint()

The `xml:prettyPrint()` function formats an XML field to improve its readability by adding indentation and line breaks. This can be a computationally expensive operation, so it is best to use it at the end of a query after all filtering is complete.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional | The name of the field to store the formatted output in. |
| **field** | string | required <br> default: `@rawstring` | The name of the field containing the XML to format. |
| **step** | integer | optional <br> default: 2 | The number of characters to use for indentation. |
| **strict** | boolean | optional <br> default: `false` | If `true`, only valid XML will be formatted; invalid XML will result in no output field. If `false`, invalid XML is copied to the output field unmodified. |
| **width** | integer | optional <br> default: 80 | The desired maximum width (in characters) of the output lines. |

***

## Examples

### Format Only Valid XML Input

This example demonstrates how to use the `strict` parameter to ensure that only well-formed XML is formatted and passed on, which is useful for validating XML integrity.

* **Query Example**
    ```
    formattedXml := xml:prettyPrint(field=message, strict=true)
    ```

* **Step-by-Step**
    1.  The query starts with events containing XML in a field named `message`.
    2.  `xml:prettyPrint()` is called on the `message` field.
    3.  Because `strict=true` is set, the function first validates the XML. If the XML in the `message` field is valid, it is formatted and stored in the `formattedXml` field.
    4.  If the XML is invalid, the `formattedXml` field is not created for that event.

---

### Format XML with Custom Line Length and Indentation

This example shows how to control the visual layout of the formatted XML by specifying the line width and indentation step.

* **Query Example**
    ```
    formattedXml := xml:prettyPrint(field=payload, step=4, width=100)
    ```

* **Step-by-Step**
    1.  The query targets a field named `payload` that contains XML data.
    2.  `xml:prettyPrint()` is called with two custom formatting parameters:
        * `step=4`: This sets the indentation for each level of the XML tree to 4 spaces.
        * `width=100`: This sets the maximum line length to 100 characters before wrapping.
    3.  The resulting, nicely formatted XML is stored in the `formattedXml` field.

---

### Format XML After Filtering Data (Recommended Practice)

This example shows the recommended best practice of applying `xml:prettyPrint()` at the end of a query, after the data has been filtered, to avoid unnecessary processing on events that will be discarded.

* **Query Example**
    ```
    #type=SOAP
    | account=123
    | xml:prettyPrint()
    ```
* **Step-by-Step**
    1.  The query first filters for all events where the `#type` is `SOAP`.
    2.  It then further filters those results to keep only events where the `account` field is `123`.
    3.  `xml:prettyPrint()` is then called on only this small, relevant subset of events. Since no `field` is specified, it formats the `@rawstring` of the filtered events.