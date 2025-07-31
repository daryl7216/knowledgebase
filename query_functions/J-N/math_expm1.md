# math:expm1()

The `math:expm1()` function calculates $e^x - 1$. For values of $x$ close to 0, the exact sum of `expm1(x) + 1` is much closer to the true result of $e^x$ than `exp(x)`.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `expm1` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:expm1()` function computes the value of Euler's number ($e$) raised to the power of the numerical value in the specified input field, and then subtracts 1 from the result. This function is particularly useful when the value of $x$ (from the input field) is very close to 0, as `expm1(x) + 1` provides a more accurate approximation of $e^x$ than `exp(x)` in such cases. The calculated value is then stored in a new output field. By default, this output field is named `expm1`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:expm1()` in the document.