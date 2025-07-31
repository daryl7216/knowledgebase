# urlDecode()

The `urlDecode()` function decodes a URL-encoded string field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_urldecode` | The name of the output field. |
| **field** | string | required | The name of the input field with the value to be URL-decoded. The parameter name `field` can be omitted. |

***

## Example

### URL-Decode a Field into a Custom Output Field

This example takes a URL-encoded string from a field named `Bar` and decodes it, storing the result in a new field named `Foo`.

* **Query Example**
    ```
    urlDecode("Bar", as="Foo")
    ```

* **Input Event Data**
    An event with a field `Bar` containing the URL-encoded string:
    `Bar=https%3A%2F%2Fwww.example.com`

* **Result Event Data**
    The query creates a new field named `Foo` with the decoded value.
    `Foo="https://www.example.com"`