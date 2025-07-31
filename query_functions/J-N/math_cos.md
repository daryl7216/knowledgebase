# math:cos()

The `math:cos()` function calculates the cosine of a field.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `COS` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:cos()` function computes the cosine of the numerical values present in the specified input field. The calculated cosine value is then stored in a new output field. By default, this output field is named `COS`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:cos()` in the document.