# math:arctan2()

The `math:arctan2()` function calculates the angle in radians from the positive x-axis to the point $(x, y)$ in the plane.

***

## Parameters

| Parameter | Type | Required/Default | Description |
| :--- | :--- | :--- | :--- |
| `as` | string | optional \<br\> default: `_arctan2` | The name of the output field. |
| `X` | string | required | The x-coordinate. |
| `y` | string | required | The y-coordinate. |

***

## Function Operation

The `math:arctan2()` function computes the angle (in radians) from the positive x-axis to a point specified by its x and y coordinates. The `X` parameter represents the x-coordinate, and the `y` parameter represents the y-coordinate. The calculated angle is then outputted into a new field, which is named `_arctan2` by default but can be customized using the `as` parameter.

It is important to note that math functions performed on ARM architecture may yield slightly different results in very high-precision calculations when compared to Intel/AMD architectures.

***

## Examples

There are no examples provided for `math:arctan2()` in the document.