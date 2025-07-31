# math:floor()

The `math:floor()` function rounds the field value to the largest (closest to positive infinity) double value that is less than or equal to the field value and is equal to a mathematical integer.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_floor` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:floor()` function computes the floor of the numerical values in the specified input field. This means it rounds the value down to the nearest whole number. The result is outputted into a new field, which by default is named `_floor` unless a different name is specified using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:floor()` in the document.