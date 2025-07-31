# math:log2()

The `math:log2()` function calculates the base 2 logarithm of a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `log2` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log2()` function computes the base 2 logarithm of the numerical values present in the specified input field. The calculated base 2 logarithm is then stored in a new output field. By default, this output field is named `log2`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log2()` in the document.