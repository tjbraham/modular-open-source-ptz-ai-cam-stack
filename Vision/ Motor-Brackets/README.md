# PTZ Motor Brackets

## Overview
This folder contains the `.stl` files and assembly drawing for the pan and tilt brackets. These components were iteratively designed in Onshape to provide a wide, unobstructed range of motion for the camera while maintaining a compact footprint.

## Printing Details
* **Material:** PLA (Polylactic Acid)
* **Supports:** Recommended depending on print orientation.
* **Hardware Compatibility:** Designed specifically to mount [9g Miuzei 180° metal-geared servo motors](https://a.co/d/02dfNC2N).

## Assembly & Calibration Notes
When assembling the brackets, ensure the servos are calibrated *before* screwing them into the final mounts:
1. **Pan Axis:** Initialize the pan servo to 90 degrees (center). Mount the bracket so the camera faces directly forward.
2. **Tilt Axis:** Manually send test angles to find the physical limits of the downward range. Ensure the starting angle prevents the camera sensor from colliding with the lower bracket or the stationary base. For this case, 25 degrees was the starting angle to prevent collision with the bottom bracket.

These motor brackets are designed to be external from the enclosure. The lower bracket is later integrated into the enclosure itself with the same design and measurements.