# math:exp()

The `math:exp()` function calculates Euler's number $e$ raised to the power of a double value in a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `exp` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:exp()` function computes the value of Euler's number ($e$, approximately $2.71828$) raised to the power of the numerical value present in the specified input field. The result of this exponential calculation is then stored in a new output field. By default, this output field is named `exp`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:exp()` in the document.