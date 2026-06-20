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
* **Camera:** [Arducam IMX477](https://a.co/d/02Vyd7Xm) (Compatible with Nvidia Jetson Board)
* **Motors:** 2x [9g Miuzei 180° Metal Geared Servos](https://a.co/d/02dfNC2N) (Metal gears selected for durability during rapid tracking)
* **Motor Driver:** [PCA9685 I2C PWM Driver Board](https://a.co/d/045lCh1O)
* **Power:** 5V, ~1A external power supply for the PWM board (will come from Power Stack)
* **Chassis:** Custom 3D printed PLA parts (see [`Motor-Brackets`](Motor-Brackets/README.md) and [`Enclosure`](Enclosure/README.md) directories) 

## Software Integration
Video ingestion is handled via a custom GStreamer pipeline optimized for the Jetson architecture. It pulls raw memory buffers via the MIPI CSI-2 connection and converts them for fast, direct ingestion into the YOLO AI model, minimizing unnecessary processing overhead:

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
```
## Software Pipeline
The vision pipeline is designed for maximum performance on the Jetson architecture, separating tasks across dedicated threads:

1. **Video Ingestion (GStreamer):** Bypasses standard OpenCV capture methods. It utilizes `nvarguscamerasrc` to pull raw memory buffers directly via the MIPI CSI-2 connection, minimizing latency.
2. **AI Inference (YOLO):** Frames are passed to an Ultralytics YOLO model. The pipeline uses a "latest-frame-wins" queue strategy to drop stale frames if the AI falls behind, ensuring real-time tracking.
3. **Visual Overlays (OpenCV):** Once bounding boxes are calculated, OpenCV (`cv2`) handles the visual processing. It draws tracking boxes, confidence labels, and applies a dynamic alpha-blending effect to fade out non-selected targets.
4. **Hardware Translation (`motor_controller.py`):** The exact pixel coordinates of the active target are passed to a dedicated background motor thread. This applies an Exponential Moving Average (EMA) filter to smooth the YOLO data, then calculates proportional pan/tilt step adjustments.
