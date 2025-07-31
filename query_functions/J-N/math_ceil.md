# math:ceil()

The `math:ceil()` function rounds the field value to the smallest (closest to negative infinity) double value that is greater than or equal to the field value and is equal to a mathematical integer.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_ceil` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:ceil()` function computes the ceiling of the numerical values in the specified input field. This means it rounds the value up to the next whole number. The result is outputted into a new field, which by default is named `_ceil` unless a different name is specified using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:ceil()` in the document.