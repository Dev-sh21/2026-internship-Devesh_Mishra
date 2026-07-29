---
layout: page
title: "Internship Progress Week 4"
permalink: /weekly-updates/week4/
---

This week, my work focused on replicating the Robotics Academy camera-equipped robot application inside the containerized VNC environment, resolving emulation and version mismatches, and starting development of the Nav2-based global navigation system.

---

## 1. Replicating the Camera Robot Application inside RADI

The core goal was to run the simulation stack inside the Robotics Academy Docker Image (RADI) environment and display the live camera feed:

* **World Integration:** Mounted the local workspace to the Docker container, launching `simple_circuit.world` inside Gazebo Fortress.
* **Model Spawning:** Dynamically spawned the custom differential-drive `moving_camera_robot` directly on the track line coordinates `(x: -14.0, y: -30.0, z: 0.5)` with `yaw = 1.57` (facing positive Y along the track road).
* **Sensor Bridging:** Sourced the ROS 2 Humble setup and ran `ros_gz_bridge` to link `/camera/image_raw` from Gazebo to ROS 2. 
* **VNC Verification:** Launched `rviz2` within the VNC desktop display to verify that the camera sensor outputs correctly in the visual panel.

---

## 2. Failures Encountered & System Resolutions

We investigated several failures across native macOS execution and emulator containers before setting up the working environment:

* **Local macOS Simulation Failures:**
  Attempts to run Gazebo and ROS 2 natively on macOS failed to render camera visualization (displaying "No Image" in RViz2). This was due to macOS lacking native X11 render pipeline integration for Gazebo camera rendering and DDS loopback multicast blocks.
* **Rosetta Emulation Crashes (x86 image on ARM64):**
  Running the standard x86 `jderobot/robotics-academy` container on Apple Silicon failed. Rosetta 2 translation corrupted Mesa's OpenGL software JIT compiler (`llvmpipe`), freezing the VNC screen.
* **VNC Container Success & SDF Downgrades:**
  We switched to a native `linux/arm64` VNC image. To fix parsing errors, we downgraded the SDF worlds and models from version 1.10 (only supported in Gazebo Harmonic) to version 1.8/1.9 (compatible with Fortress).
* **Camera Angle Fix:**
  The camera initially pointed parallel to the chassis (looking at the horizon). We modified the sensor pose inside `moving_camera_robot.sdf` to pitch downward by `0.4` radians to bring the red track line into the visual frame.

![Working Follow Line Camera Feed](../docs/assets/img/posts/follow_line_working.png)
*Figure: Live VNC desktop showing Gazebo simulation, spawned robot on the simple circuit track, and active camera feed in RViz2.*

---

## 3. Nav2 Global Navigation (Work Initiated)

I initiated work on a Nav2-based path planning solution for the Global Navigation exercise:
* **Installation:** Installed the ROS 2 Humble Navigation Stack (`nav2-bringup`) in the container environment.
* **Configuration:** Sourced map and costmap parameters to test default global path planners (like `NavFn` and `SmacPlanner`) included in Nav2.
