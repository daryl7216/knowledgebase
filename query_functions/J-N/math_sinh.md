# math:sinh()

The `math:sinh()` function calculates the hyperbolic sine of a double field. The hyperbolic sine of $x$ is defined to be $(e^x - e^{-x})/2$ where $e$ is Euler's number.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `sinh` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:sinh()` function computes the hyperbolic sine of the numerical values in the specified input field. The hyperbolic sine of a number $x$ is mathematically defined as $(e^x - e^{-x})/2$, where $e$ is Euler's number (the base of the natural logarithm). The calculated hyperbolic sine value is then stored in a new output field. By default, this output field is named `sinh`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:sinh()` in the document.