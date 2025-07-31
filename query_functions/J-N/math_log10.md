# math:log10()

The `math:log10()` function calculates the base 10 logarithm of a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_log10` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log10()` function computes the base 10 logarithm of the numerical values present in the specified input field. The calculated base 10 logarithm is then stored in a new output field. By default, this output field is named `_log10`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log10()` in the document.