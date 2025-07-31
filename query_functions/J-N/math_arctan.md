# math:arctan()

The `math:arctan()` function calculates the arc tangent of a value; the returned angle is in the range $-\\pi/2$ through $\\pi/2$.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `arctan` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:arctan()` function computes the arc tangent (inverse tangent) of the numerical values in the specified input field. The resulting angle is expressed in radians and will fall within the range of $-\\pi/2$ to $\\pi/2$. The computed arc tangent value is stored in a new output field. By default, this output field is named `arctan`, but a custom name can be assigned using the `as` parameter.

It's important to note that math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arctan()` in the document.