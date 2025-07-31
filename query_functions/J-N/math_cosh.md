# math:cosh()

The `math:cosh()` function computes the hyperbolic cosine of a double field. The hyperbolic cosine of $x$ is defined to be $(e^x + e^{-x})/2$ where $e$ is Euler's number.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `cosh` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:cosh()` function calculates the hyperbolic cosine of the numerical values in the specified input field. The hyperbolic cosine of a number $x$ is mathematically defined as $(e^x + e^{-x})/2$, where $e$ is Euler's number (the base of the natural logarithm). The calculated hyperbolic cosine value is then stored in a new output field. By default, this output field is named `cosh`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:cosh()` in the document.