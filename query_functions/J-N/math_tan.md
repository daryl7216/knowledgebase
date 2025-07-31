# math:tan()

The `math:tan()` function calculates the trigonometric tangent of an angle in a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `tan` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:tan()` function computes the trigonometric tangent of the numerical angle values present in the specified input field. The calculated tangent value is then stored in a new output field. By default, this output field is named `tan`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:tan()` in the document.