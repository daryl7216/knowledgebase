# math:pow()

The `math:pow()` function calculates the field value to the exponent power, that is, $\\text{field}^{\\text{exponent}}$. It supports decimal values.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_pow` | The name of the output field. |
| `exponent` | number | required | The exponent to use, for example, $\\text{exp}$ in $x^{\\text{exp}}$. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:pow()` function computes the result of raising the numerical value from the specified `field` to the power of the `exponent`. Both the base (from `field`) and the `exponent` can be decimal values. The calculated result is then stored in a new output field. By default, this output field is named `_pow`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:pow()` in the document.