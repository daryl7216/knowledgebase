# math:abs()

The `math:abs()` function calculates the absolute value of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `abs` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:abs()` function computes the absolute value of the numerical data in the specified input field. The result is outputted into a new field, which by default is named `abs` unless a different name is specified using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:abs()` in the document.