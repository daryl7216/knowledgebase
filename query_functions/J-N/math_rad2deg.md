# math:rad2deg()

The `math:rad2deg()` function converts angles from radians to degrees.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_rad2deg` | The name of the output field. |
| `field` | string | required | The name of the input field. |

***

## Function Operation

The `math:rad2deg()` function takes numerical values from the specified input field, which are assumed to be angles in radians, and converts them into their equivalent degree measures. The calculated degree value is then stored in a new output field. By default, this output field is named `_rad2deg`, but a custom name can be assigned using the `as` parameter.

A note on precision: Math functions on ARM architecture may return different results in very high-precision calculations compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:rad2deg()` in the document.