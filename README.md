# Realsense-depthcamera-435i-integration-with-R-tab-ros2-humble-complete-guide
Robotics enthusiast showcasing the Intel RealSense D435i after building a full RGBD V-SLAM pipeline using ROS 2 Humble and RTAB-Map. The robotic helmet symbolizes innovation, autonomous perception, and real-time 3D mapping in a futuristic robotics workflow.
# Intel RealSense D435i + ROS 2 Humble + RTAB-Map V-SLAM Setup Guide

## Overview

This guide documents the complete setup process for running RGBD Visual SLAM using:

* Ubuntu 22.04
* ROS 2 Humble
* Intel RealSense D435i
* RTAB-Map
* RViz2

The setup includes:

* RealSense firmware recovery/update
* librealsense installation
* ROS 2 RealSense integration
* RGBD SLAM using RTAB-Map
* Live 3D mapping and visualization

---

# System Used

## Hardware

* Intel RealSense D435i
* RTX 4050 Laptop GPU
* Intel i7 CPU
* SSD Storage

## Software

* Ubuntu 22.04
* ROS 2 Humble
* librealsense 2.57+
* RTAB-Map

---

# Step 1 — Install ROS 2 Humble

## Setup Locale

```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

## Add ROS Repository

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

```bash
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc \
-o /usr/share/keyrings/ros-archive-keyring.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

## Install ROS 2 Humble

```bash
sudo apt update
sudo apt install ros-humble-desktop -y
```

## Source ROS

```bash
source /opt/ros/humble/setup.bash
```

## Add to bashrc

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## Verify Installation

```bash
ros2 -h
```

---

# Step 2 — Install Intel RealSense SDK

## Add Intel Repository

```bash
sudo mkdir -p /etc/apt/keyrings
curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | \
sudo gpg --dearmor -o /etc/apt/keyrings/librealsense.pgp
```

```bash
echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo jammy main" | \
sudo tee /etc/apt/sources.list.d/librealsense.list
```

## Install librealsense

```bash
sudo apt update
sudo apt install librealsense2-utils librealsense2-dev -y
```

## Verify Camera Detection

```bash
lsusb
```

Expected:

```text
Intel Corp. Intel(R) RealSense(TM) Depth Camera 435i
```

---

# Step 3 — Update D435i Firmware

## Problem Encountered

Old firmware caused:

* IMU crashes
* depth stream failures
* device instability
* ROS launch failures

Old firmware:

```text
05.11.15.00
```

Updated to:

```text
05.17.0.10
```

## Firmware Update Procedure

### On Windows

1. Install Intel RealSense Viewer
2. Connect D435i
3. Open Viewer
4. Update firmware manually
5. Select firmware file:

```text
Signed_Image_UVC_5_17_0_10.bin
```

6. Wait for update completion

---

# Step 4 — Install ROS 2 RealSense Packages

```bash
sudo apt update

sudo apt install -y \
  ros-humble-realsense2-camera \
  ros-humble-realsense2-description
```

## Verify Packages

```bash
ros2 pkg list | grep realsense
```

Expected:

```text
realsense2_camera
realsense2_camera_msgs
realsense2_description
```

---

# Step 5 — Launch Stable RealSense Configuration

## Final Stable Launch

```bash
source /opt/ros/humble/setup.bash

ros2 launch realsense2_camera rs_launch.py \
  enable_color:=true \
  enable_depth:=true \
  enable_gyro:=false \
  enable_accel:=false \
  depth_module.depth_profile:=640x480x30 \
  rgb_camera.color_profile:=640x480x30
```

## Verify Topics

```bash
ros2 topic list | grep image
```

---

# Step 6 — Install RTAB-Map

```bash
sudo apt update

sudo apt install -y \
  ros-humble-rtabmap \
  ros-humble-rtabmap-ros \
  ros-humble-rtabmap-launch \
  ros-humble-rtabmap-viz \
  ros-humble-rtabmap-rviz-plugins
```

---

# Step 7 — Launch RTAB-Map RGBD SLAM

## Final Working Launch

```bash
source /opt/ros/humble/setup.bash

ros2 launch rtabmap_launch rtabmap.launch.py \
  rtabmap_args:="--delete_db_on_start" \
  rgb_topic:=/camera/camera/color/image_raw \
  depth_topic:=/camera/camera/depth/image_rect_raw \
  camera_info_topic:=/camera/camera/color/camera_info \
  frame_id:=camera_link \
  approx_sync:=true \
  rviz:=true
```

## Important Notes

### approx_sync:=true

Required because RGB and depth timestamps are slightly different.

Without this:

* map frame errors occur
* synchronization warnings appear
* odometry becomes unstable

---

# Step 8 — Mapping Tips

For better SLAM quality:

* Move slowly
* Avoid fast rotations
* Use textured environments
* Avoid blank walls
* Ensure good lighting
* Revisit previous locations for loop closure

---

# Step 9 — Save Map

Inside RTAB-Map GUI:

```text
File → Save database
```

Default location:

```text
~/.ros/rtabmap.db
```

---

# Step 10 — Reload Saved Map

```bash
source /opt/ros/humble/setup.bash

ros2 launch rtabmap_launch rtabmap.launch.py \
  database_path:=~/.ros/rtabmap.db \
  rgb_topic:=/camera/camera/color/image_raw \
  depth_topic:=/camera/camera/depth/image_rect_raw \
  camera_info_topic:=/camera/camera/color/camera_info \
  frame_id:=camera_link \
  approx_sync:=true \
  rviz:=true
```

IMPORTANT:

Do NOT use:

```text
--delete_db_on_start
```

otherwise the saved map will be erased.

---

# Step 11 — Export Occupancy Grid

Inside RTAB-Map GUI:

```text
File → Export grid map
```

This generates:

* .pgm
* .yaml

These can be used with:

* Nav2
* autonomous navigation
* path planning

---

# Common Problems and Fixes

## Problem: ros2 command not found

### Fix

```bash
source /opt/ros/humble/setup.bash
```

---

## Problem: RealSense not detected

### Fix

1. Replug camera
2. Use USB 3 port
3. Verify with:

```bash
lsusb
```

---

## Problem: rs-enumerate-devices fails

### Cause

Old firmware instability.

### Fix

Update D435i firmware to:

```text
05.17.0.10
```

---

## Problem: IMU received doesn't have orientation set

### Cause

RTAB-Map expects orientation-enabled IMU data.

### Temporary Fix

Disable IMU and run RGBD-only SLAM.

---

## Problem: Fixed Frame [map] does not exist

### Cause

RGB and depth frames not synchronized.

### Fix

Use:

```text
approx_sync:=true
```

---

# Final Result

Successfully achieved:

* RGBD Visual SLAM
* Live 3D Mapping
* RTAB-Map Integration
* RealSense D435i Integration
* ROS 2 Humble Setup
* RViz Visualization
* Point Cloud Reconstruction
* Trajectory Tracking

---

# Future Improvements

* Add IMU orientation filter
* Integrate Nav2
* Run ORB-SLAM3
* Build autonomous robot navigation
* Add semantic mapping
* Perform drone SLAM experiments

---

# Credits

Built using:

* ROS 2 Humble
* Intel RealSense SDK
* RTAB-Map
* RViz2
* Ubuntu 22.04
