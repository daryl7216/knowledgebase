# sankey()

A companion function to the Sankey that produces data compatible with the widget. For more information about Sankey widgets, see Sankey Diagram Widget.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `source` | string | required | The field containing the source node ID. |
| `target` | string | required | The field containing the target node ID. |
| `weight` | aggregate | optional \<br\> default: `count(as=_count)` | A function used to calculate the weight of the edges. Good candidates are functions like, for example, `sum()`, `count()` or `max()`. |

***

## Function Operation

The `sankey()` function is designed to prepare data in a format suitable for rendering Sankey diagrams in a widget. Sankey diagrams are powerful data visualization tools used to represent the flow of values between stages, offering valuable insights into data movement. They are particularly useful for visualizing complex processes, highlighting a single aspect or resource, and revealing data inconsistencies.

To produce Sankey-compatible data, the `sankey()` function requires at least two fields:

  * `source`: Specifies the field that contains the source node IDs.
  * `target`: Specifies the field that contains the target node IDs.

These two fields define the "flow" or connection between different entities.

Additionally, an optional `weight` parameter can be provided. This parameter takes an aggregate function (such as `sum()`, `count()`, or `max()`) to calculate the magnitude or quantity of the flow between the source and target nodes. This calculated weight determines the thickness of the edges in the Sankey diagram. If no `weight` function is specified, `count(as=_count)` is used by default.

***

## Examples

### Create Data Compatible With Sankey Diagram Widget - Example 1

This example uses the `sankey()` function to produce Sankey-compatible data for a web server log, showing which URLs match the HTTP methods.

  * **Query Example**:

    ```
    type=accesslog
    sankey(source=method, target=url)
    ```

  * **Input Event Data**:

    | type | timestamp | method | url |
    | :--- | :--- | :--- | :--- |
    | accesslog | 2025-05-02 11:48:15 | GET | /api/cart |
    | accesslog | 2025-05-02 11:48:45 | POST | /api/login |
    | accesslog | 2025-05-02 11:49:01 | GET | /api/logout |
    | accesslog | 2025-05-02 11:49:30 | GET | /api/users |
    | accesslog | 2025-05-02 11:50:00 | POST | /api/products |
    | accesslog | 2025-05-02 11:50:23 | PUT | /api/products |
    | accesslog | 2025-05-02 11:50:45 | GET | /api/users |
    | accesslog | 2025-05-02 11:51:12 | POST | /api/cart |
    | accesslog | 2025-05-02 11:51:33 | GET | /api/users |
    | accesslog | 2025-05-02 11:52:01 | GET | /api/products |
    | accesslog | 2025-05-02 11:52:30 | POST | /api/orders |
    | accesslog | 2025-05-02 11:53:00 | DELETE | /api/products |
    | accesslog | 2025-05-02 11:53:15 | GET | /api/dashboard |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `type=accesslog`: Filters for all web server logs. Web server logs provide valuable business insight as they list all requests for individual files that users have made from a website.
    3.  `sankey(source=method, target=url)`: Produces Sankey-compatible data for a web server log, showing which URLs match the HTTP methods. It shows how traffic is distributed across different endpoints. Note that `source` and `target` are required fields to be able to produce a Sankey Diagram.
    4.  Event Result set.

  * **Summary and Results**:
    The query is used to produce data compatible with Sankey diagram widgets, in this example visualizing the relationship between HTTP methods and URLs. Sankey diagram widgets are a powerful data visualization tool used to represent the flow of values between stages, offering valuable insights into the flow of data. When used with well-structured data, they can help identify patterns, bottlenecks, and significant resource allocation trends. Sankey diagram widgets are useful if you want to show complex processes visually, with a focus on a single aspect or resource required to be highlighted. Sankey diagram widgets can also be used to reveal inconsistent data, as the visualization of data inconsistencies makes detection easier.

---

### Create Data Compatible With Sankey Diagram Widget - Example 2

This example uses the `sankey()` function to produce Sankey-compatible data for a web server log, showing the URL address (the referrer) of the website that sends users to another website using a link.

  * **Query Example**:

    ```
    Type=accesslog referrer!="-"
    sankey(source=referrer, target=url)
    ```

  * **Input Event Data**:

    | Type | timestamp | method | url | referrer |
    | :--- | :--- | :--- | :--- | :--- |
    | accesslog | 2025-05-02 11:45:23 | GET | /api/users | `http://example.com/login` |
    | accesslog | 2025-05-02 11:45:30 | POST | /api/users | `http://example.org/signup` |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `Type=accesslog referrer!="-"`: Filters for all web server logs where the `referrer` is not equal to `-`. This returns all results where the value is not `-`. The value `-` is often used by analytics to define an empty value. In this example, we therefore filter for web server logs where there is referrer information.
    3.  `sankey(source=referrer, target=url)`: Produces Sankey-compatible data for a web server log, showing which target URLs match the referrer URL. If one URL links to the same website twice, the count of referring pages for that website will be one and the count of backlinks will be two. Note that `source` and `target` are required fields to be able to produce a Sankey Diagram.
    4.  Event Result set.

  * **Summary and Results**:
    The query is used to produce data compatible with Sankey diagram widgets, showing the referrer URL. The query is useful to get an overview of the web page that a user was on right before they landed on your page, the target (it shows how users navigate to different URLs on your site).

---

### Create Sankey Diagram Calculating Edge Thickness

This example creates a Sankey diagram calculating the edge thickness using the `sankey()` function with an aggregator.

  * **Query Example**:

    ```
    sankey(source="src", target="dst", weight=(sum(cnt)))
    ```

  * **Input Event Data**:

    | cnt | dst | src |
    | :--- | :--- | :--- |
    | 12 | apples | john |
    | 1 | bananas | john |
    | 1 | apples | joe |
    | 1 | apples | sarah |
    | 1 | apples | sarah |
    | 1 | apples | sarah |

  * **Step-by-Step**:

    1.  Starting with the source repository events.
    2.  `sankey(source="src", target="dst", weight=(sum(cnt)))`: Creates a Sankey diagram showing the sources on the left side (john, joe, sarah), and the targets on the right side (apples, bananas), and then provides the edge thickness based on the sum of `cnt`. The default `weight` parameter is `count()`.
    3.  Event Result set.

  * **Summary and Results**:
    The query is used to create a Sankey diagram showing the sources on the left side (john, joe, sarah), and the targets on the right side (apples, bananas), and then display the edge thickness using the `weight` parameter. In this example, the thickest edge is `john -> apples` (12), the medium edge is `sarah -> apples` (3), and the thin edges are `joe -> apples` (1) and `john -> bananas` (1). The query is useful for rendering results as a two-level Sankey diagram and visualizing flow relationships between nodes. It shows proportional relationships between data categories.

  * **Result Event Data**:

    | source | target | weight |
    | :--- | :--- | :--- |
    | joe | apples | 1 |
    | john | apples | 12 |
    | john | bananas | 1 |
    | sarah | apples | 3 |