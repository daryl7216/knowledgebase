# math:mod()

The `math:mod()` function calculates the floor modulus of a field value and a divisor, for example, $x \\pmod y$ where $y$ is the divisor and is a field. Both the field or divisor are floor rounded before the calculation. Decimal values are supported.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `mod` | The name of the output field. |
| `divisor` | number | required | The divisor to use, for example, $x \\pmod{\\text{divisor}}$. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:mod()` function computes the floor modulus of a given field's value (dividend) and a specified divisor. This operation returns the remainder of the division, ensuring the result has the same sign as the divisor. Both the field value and the divisor are rounded down (floor-rounded) to the nearest integer before the calculation is performed. Decimal values are supported for both the dividend and the divisor. The calculated modulus is then stored in a new output field. By default, this output field is named `mod`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:mod()` in the document.