# math:log()

The `math:log()` function calculates the natural logarithm (base $e$) of the value in a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_log` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:log()` function computes the natural logarithm (logarithm to base $e$) of the numerical values in the specified input field. The calculated natural logarithm is then stored in a new output field. By default, this output field is named `_log`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:log()` in the document.