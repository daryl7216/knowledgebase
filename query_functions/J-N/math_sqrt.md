# math:sqrt()

The `math:sqrt()` function calculates the rounded positive square root of a double field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_sqrt` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:sqrt()` function computes the positive square root of the numerical values present in the specified input field. The calculated square root is then rounded and stored in a new output field. By default, this output field is named `_sqrt`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:sqrt()` in the document.