# math:arccos()

The `math:arccos()` function calculates the arc cosine of a field; the returned angle is in the range $0.0$ through $\\pi$.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `arccos` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:arccos()` function computes the arc cosine (inverse cosine) of the numerical values in the specified input field. The resulting angle is expressed in radians and will fall within the range of $0.0$ to $\\pi$. The computed arc cosine value is stored in a new output field. By default, this output field is named `arccos`, but a custom name can be assigned using the `as` parameter.

It's important to note that math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arccos()` in the document.