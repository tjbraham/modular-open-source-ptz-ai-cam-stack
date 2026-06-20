# Stationary Base Enclosure

## Overview
This folder contains the `.stl` files for the main electronics enclosure. Designed in Onshape, this base uses a "tower" architecture to keep the footprint minimal while providing enough height for the camera to have a clear vantage point.

## Printing Details
* **Material:** PLA
* **Supports:** Print the bottom on the flat side using required supports. Printing the lid right side up, tree supports were used to reduce the amount of material used and speed up print time.

## Assembly Notes
The enclosure acts as the foundation for the entire physical system. It was dimensioned to house the following core electronics safely:
* NVIDIA Jetson Orin Nano Devkit
* [PCA9685 I2C PWM Driver Board](https://a.co/d/045lCh1O)
* Internal wire routing and power distribution

The top of the tower integrates the bottom bracket into the enclosure itself for the pan servo and bracket assembly found in the `Motor-Brackets` directory.
