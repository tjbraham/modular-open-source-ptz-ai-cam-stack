# Vision & PTZ Hardware

## Overview
This directory contains the hardware integration, mechanical assembly files, and motor control logic for the Modular PTZ AI Camera Platform. 

While the broader project handles the YOLO model training and AI logic, this subsystem acts as the physical "eyes and neck" of the platform. It is responsible for capturing high-resolution video and translating AI bounding-box data into smooth, calibrated mechanical tracking.

## Scope
* **Camera Integration:** Selection and setup of the Arducam IMX477 sensor. This sensor was chosen for its optimal balance of price, pixel size, and megapixel count. It connects via a MIPI CSI-2 connection, providing low-latency video ingestion.
* **Mechanical Design:** Iterative 3D printing (PLA) of the pan-tilt brackets and the primary enclosure, designed to house the Jetson Orin Nano, power distribution, and servo motors.
* **Tracking Logic:** A custom multithreaded Python controller (`motor_controller.py`) that calculates positional error from bounding boxes and drives the servos.
* **Power Isolation:** Implementation of an external I2C PWM driver to safely power the motors, isolating current draw from the Jetson's GPIO pins.

## Hardware & Bill of Materials (BOM)
* **Compute:** NVIDIA Jetson Orin Nano
* **Camera:** Arducam IMX477 (MIPI CSI-2 connection)
* **Motors:** 2x 9g Miuzei 180° Metal Geared Servos (Metal gears selected for durability during rapid tracking)
* **Motor Driver:** PCA9685 I2C PWM Driver Board
* **Power:** 5V, 1A (min) external power supply for the PWM board
* **Chassis:** Custom 3D printed PLA parts (see `bracket_files` and `enclosure_files` directories)

## Software Integration
Video ingestion is handled via OpenCV using a custom GStreamer pipeline optimized for the Jetson architecture. It pulls raw memory buffers and converts them for fast AI processing:

```python
def gstreamer_pipeline(self) -> str:
    return (
        f"nvarguscamerasrc sensor-id={self.sensor_id} ! "
        f"video/x-raw(memory:NVMM), width=(int){self.width}, height=(int){self.height}, "
        f"framerate=(fraction){self.fps}/1 ! "
        f"nvvidconv flip-method={self.flip_method} ! "
        f"video/x-raw, width=(int){self.width}, height=(int){self.height}, format=(string)BGRx ! "
        "videoconvert ! video/x-raw, format=(string)BGR ! "
        "appsink name=appsink0 emit-signals=false drop=true max-buffers=1 sync=false"
    )