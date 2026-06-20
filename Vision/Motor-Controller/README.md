# PTZ Motor Controller 

## Overview
This directory contains the `MotorController` class, which serves as the hardware driver for the pan-tilt mechanics. It translates raw bounding-box coordinates from the YOLO vision model into smooth, proportional servo movements. Essentially, the software that handles the tracking aspect.

## Architecture & Threading
To ensure the main AI vision loop runs as fast as possible, this controller operates on a **dedicated background thread**. 
* The main vision loop passes center coordinates to the `update_target(obj_x, obj_y)` method.
* A queue system (max size = 1) is used so that if the motors are busy moving, older frames are automatically dropped in favor of the newest data.
* Proportional tuning (`kp_pan`, `kp_tilt`) and deadzone thresholds ensure the camera smoothly converges on the target without jittering or overshooting.

## Dependencies
This module requires the Adafruit ServoKit library to communicate with the PCA9685 PWM board over I2C:
`pip install adafruit-circuitpython-servokit`

### Simulation Mode
If the code is run on a machine without the I2C hardware connected (e.g., for testing the vision loop on a local laptop), the `MotorController` will gracefully fall back to **Simulation Mode**. It will bypass the `ImportError`, print the calculated angle steps to the console (`logger.debug`), and prevent the software from crashing.

## Usage Example
```python
from motor_control.motor_controller import MotorController

# 1. Initialize with your camera frame dimensions
ptz = MotorController(frame_h=720, frame_w=1280)

# 2. Start the background motor thread
ptz.start()

# 3. Inside your main vision/YOLO loop:
ptz.update_target(yolo_center_x, yolo_center_y)

# 4. Cleanly stop the thread on exit
ptz.stop()