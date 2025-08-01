# parseUri()

This function extracts URI components from an input field and adds them as attributes to the event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `defaultBase` | string | optional | A scheme and authority prefix or `//`, used as the base to resolve the input as a URI reference. Valid values are of the form `//` or `scheme://` where `scheme` is a URI scheme. |
| `field` | string | required | The name of the input field that contains the value to analyze. |
| `prefix` | string | optional \<br\> default: input field name | An optional prefix for the field names for the components that are added to the event. It is an error to provide an empty prefix. |

***

## Function Operation

The `parseUri()` function is particularly useful for interpreting partial URIs, such as `www.example.com/path`. Instead of requiring fully valid URIs, `parseUri()` accepts a "base" (for example, `defaultBase=http://`) which is used to extract components like hostname, port, path, query, and fragment.

If the input URI has no authority component, the event remains unchanged, similar to the behavior of `parseUrl()`. For instance, `example.com` would be interpreted as the scheme, and no hostname or path would be found, as the function attempts to interpret the input as a URI. This is where providing a `defaultBase` can override such behavior. If a `defaultBase` is used with an already full URI (e.g., `ftp://example.com/path`), the `defaultBase` is ignored, and the existing scheme (e.g., `ftp`) will be extracted.

This function is more lenient than strict URI specifications and primarily matches based on the input's structure, by design, to enhance its utility in various contexts. It performs no normalization or decoding (like percent-encoding or path normalization) on the components; it processes the parts of the input exactly as they appeared in the logs. The URI components (such as `url.scheme`, `url.hostname`, `url.path`, `url.query`, `url.fragment`) are added to the event only if the input can be parsed as a URI itself or resolved to a URI reference by providing the `defaultBase` parameter. The `url.` prefix for these attributes is replaced by the input field name or the specified `prefix` value.

***

## Examples

### Parse with `defaultBase`

This example demonstrates parsing a partial URI with a specified `defaultBase`.

  * **Input Event Data**:
    `example.com:8080/foo`

  * **Query Example**:

    ```
    parseUri(field="url", defaultBase="http://")
    ```

  * **Output**:

      * `host=example.com`
      * `path=/foo`
      * `port=8080`
      * `scheme=http`

---

### Parse without `defaultBase`

This example shows parsing a partial URI without a `defaultBase`, illustrating how it might be misinterpreted.

  * **Input Event Data**:
    `example.com:8080/foo`

  * **Query Example**:

    ```
    parseUri(field="url")
    ```

  * **Output**:

      * `scheme=example.com`
      * `path=8080/foo`

---

### Parse full URI with `defaultBase`

This example illustrates that if a full URI is provided, `defaultBase` is ignored.

  * **Input Event Data**:
    `ftp://example.com:8080/foo`

  * **Query Example**:

    ```
    parseUri(field="url", defaultBase="http://")
    ```

  * **Output**:

      * `host=example.com`
      * `path=/foo`
      * `port=8080`
      * `scheme=ftp`