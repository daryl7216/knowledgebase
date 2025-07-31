# urlEncode()

The `urlEncode()` function URL-encodes the contents of a string field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| **as** | string | optional <br> default: `_urlencode` | The name of the output field. |
| **field** | string | required | The name of the input field with the value to be URL-encoded. The parameter name `field` can be omitted. |
| **type**| string | optional | The type of encoding to perform. |

***

## Example

### URL-Encode a Field into a Custom Output Field

This example takes a URL string from a field named `Bar` and encodes it, storing the result in a new field named `Foo`.

* **Query Example**
    ```
    urlEncode("Bar", as="Foo")
    ```

* **Input Event Data**
    An event with a field `Bar` containing the URL string:
    `Bar=https://www.example.com`

* **Result Event Data**
    The query creates a new field named `Foo` with the URL-encoded value.
    `Foo="https%3A%2F%2Fwww.example.com"`