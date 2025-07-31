# math:tanh()

The `math:tanh()` function calculates the hyperbolic tangent of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `tanh` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:tanh()` function computes the hyperbolic tangent of the numerical values present in the specified input field. The calculated hyperbolic tangent value is then stored in a new output field. By default, this output field is named `tanh`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:tanh()` in the document.