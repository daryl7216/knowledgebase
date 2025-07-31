# math:deg2rad()

The `math:deg2rad()` function converts angles from degrees to radians.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_deg2rad` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:deg2rad()` function takes numerical values from the specified input field, which are assumed to be angles in degrees, and converts them into their equivalent radian measures. The calculated radian value is then stored in a new output field. By default, this output field is named `_deg2rad`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:deg2rad()` in the document.