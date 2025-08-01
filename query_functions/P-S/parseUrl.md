# parseUrl()

This function extracts URL components from an input field and adds them as attributes to the event.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional | Use a prefix for the attributes added to the event. |
| `field` | string | optional \<br\> default: `url` | The field from which to parse URL components. |

***

## Function Operation

The `parseUrl()` function uses the Java `java.net.URI` class to parse and extract components from a URL string. This includes attributes such as scheme, username, password, host, port, path, fragment, and query, which are then added as new fields to the event.

Due to the parsing mechanism, the function may misidentify URL elements if the provided URL is only partial. By default, the parser prioritizes identifying the URL scheme (the part before the first colon). This can lead to incorrect parsing for URLs that are not fully qualified. For example, if "[site.example.com:80/path/to/examp](https://www.google.com/search?q=https://site.example.com:80/path/to/examp)" is provided, "site.example.com" might be incorrectly identified as `url.scheme`. Therefore, it is important to supply fully-qualified URLs to ensure accurate parsing of all components.

The function is designed to process the actual parts of the input as they were in the logs, meaning it does not perform any normalization or decoding (like percent-encoding or path normalization) on the components.

***

## Examples

### Parsing the `uri` field to aggregate the list of paths used

This example demonstrates parsing the `uri` field to extract URL components and then grouping by the `uri.path` to aggregate paths.

  * **Query Example**

    ```
    parseUrl(uri)
    groupBy(uri.path)
    ```

  * **Step-by-Step**

    1.  `parseUrl(uri)`: Parses the `uri` field in the event, extracting various URL components like scheme, host, path, etc., and adds them as new fields (e.g., `uri.path`) to the event.
    2.  `groupBy(uri.path)`: Groups the events by their extracted `uri.path`, which allows for aggregation of similar paths.

  * **Result Event Data**

| uri | \_count |
| :--- | :--- |
| /graphql | 132 |
| /api/v1/refresh | 95 |
| /api/v1/repositories/humio/queryjobs | 30 |
| /api/v1/repositories/humio/queryjobs/P29-vxBwkHK6nxeDuC4CzOOmc0Oq | 3 |
| /api/v1/repositories/humio/queryjobs/P7-mHpupr5riWe5XEoRnY8VhicU | 2 |
| /api/v1/repositories/humio/queryjobs/P7-eaRjXUkTzse6fB3M6kQcbh4m | 2 |
| /api/v1/repositories/humio/queryjobs/P7-WjVRVoJWmdh9REgpuoGHyoOC | 2 |
| /api/v1/repositories/humio/queryjobs/P7-VadNjnjvQaeXcfJRhfRb5Uuv | 2 |
| /api/v1/repositories/humio/queryjobs/P7-GlsGrU405Dml2PPeL33PwT9n | 2 |
| /api/v1/repositories/humio/queryjobs/P7-Bwg99bEHk6NeBmSU1fSZG0Fa | 2 |
| /api/v1/repositories/humio/queryjobs/P6-6VusohlK49WisjAYttFlbpfm | 2 |

---

### Parses the field named `endpoint` and adds URL components to the event

This example shows how to parse a field named `endpoint` to extract URL components.

  * **Query Example**

    ```
    parseUrl(field=endpoint)
    ```

  * **Step-by-Step**

    1.  `parseUrl(field=endpoint)`: Parses the content of the `endpoint` field to extract URL components and add them as new fields to the event.

---

### Parses the field named `endpoint` and adds URL components with `url` as a prefix

This example demonstrates parsing the `endpoint` field and prefixing the extracted URL components with `url`.

  * **Query Example**

    ```
    url: parseUrl(field=endpoint)
    ```

  * **Step-by-Step**

    1.  `url: parseUrl(field=endpoint)`: Parses the content of the `endpoint` field and adds URL components to the event with the prefix `url`, such as `url.path`, `url.scheme`, etc.

---

### Parses the field named `endpoint` and adds URL components with `apiEndpoint` as a prefix

This example illustrates parsing the `endpoint` field and using a custom prefix `apiEndpoint` for the extracted URL components.

  * **Query Example**

    ```
    parseUrl(field=endpoint, as=apiEndpoint)
    ```

  * **Step-by-Step**

    1.  `parseUrl(field=endpoint, as=apiEndpoint)`: Parses the content of the `endpoint` field and adds URL components to the event with `apiEndpoint` as a prefix, resulting in fields like `apiEndpoint.path`, `apiEndpoint.scheme`, etc.