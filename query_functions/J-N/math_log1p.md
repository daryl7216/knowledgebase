# math:log1p()

The `math:log1p()` function calculates the natural logarithm of the sum of the field value and 1. For small values of x, the result of `log1p(x)` is much closer to the true result of $\\ln(1 + x)$ than the floating-point evaluation of `log(1.0 + x)`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `log1p` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log1p()` function computes the natural logarithm of $(1 + x)$, where $x$ is the numerical value from the specified input field. This function is particularly useful for maintaining precision when $x$ is a very small number, as it provides a more accurate result than directly calculating `log(1.0 + x)`. The calculated value is then stored in a new output field. By default, this output field is named `log1p`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log1p()` in the document.