# math:sin()

The `math:sin()` function calculates the sine of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `sin` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:sin()` function computes the sine of the numerical values present in the specified input field. The calculated sine value is then stored in a new output field. By default, this output field is named `sin`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:sin()` in the document.