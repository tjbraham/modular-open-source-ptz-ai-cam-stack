# PTZ Motor Brackets

## Overview
This folder contains the `.stl` files and assembly drawing for the pan and tilt brackets. These components were iteratively designed in Onshape to provide a wide, unobstructed range of motion for the camera while maintaining a compact footprint.

## Printing Details
* **Material:** PLA
* **Supports:** Recommended depending on print orientation.
* **Hardware Compatibility:** Designed specifically to mount [9g Miuzei 180° metal-geared servo motors](https://a.co/d/02dfNC2N).

## Assembly & Motor Calibration Guide

Proper calibration is critical *before* permanently mounting the camera. The 9g Miuzei servos have a physical 180° limit, and if they are mounted at the wrong starting angle, the YOLO tracking controller will force the camera to collide with the 3D-printed PLA base.

### Step 1: Pre-Assembly Motor Centering
Before attaching the brackets to the servo splines, you must electronically center the motors.
1. Wire the **PCA9685 I2C PWM Driver** to the Jetson Orin Nano's I2C, ground, and 3V power GPIO pins.
2. Connect a dedicated 5V (~1A) power supply to the PCA9685 terminal block. **Do not power the servos directly from the Jetson's 5V GPIO pins.**
3. Plug the Pan servo into Channel 0 and the Tilt servo into Channel 1.
4. Run a simple Python script (or use the `MotorController.reset_to_center()` method) to send a `90.0` degree command to both channels. 

### Step 2: Pan Axis Assembly
With the Pan servo (Channel 0) actively held at 90 degrees by the driver:
1. Mount the lower Pan bracket onto the stationary base tower.
2. Attach the servo horn perpendicular to the motor. This may take some adjustments after fully assembled to get the camera to be perpendicular to the pan motor. This guarantees you have a full 90 degrees of panning motion to both the left and right.

### Step 3: Tilt Axis Assembly & Clearance Testing
The Tilt axis requires manual tuning to ensure the bottom of the camera sensor does not strike the Pan bracket when looking downward.
1. With the Tilt servo (Channel 1) electronically held at its starting position, attach the camera mount bracket.
2. Using the `manual_move()` API (or a test script), slowly send angle adjustments to test the physical limits. 
3. *Calibration Target:* Find the lowest possible angle where the camera has a clear downward view without the PLA parts colliding. 
4. Once the safe downward limit is found, update the default starting angle in the software. *(Note: In our configuration, the safe starting tilt angle was calibrated to 25°).*

These motor brackets are designed to be external from the enclosure. The lower bracket is later integrated into the enclosure itself with the same design and measurements. The top bracket can be used with both the enclosure bracket or the separated bottom bracket.