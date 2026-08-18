---
title: "Internship Progress Week 7 (August 13 ~ August 19)"
date: 2026-08-18 08:00:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-7, gazebo, rviz2, novnc, macos, docker, sensors, camera, lidar, odometry]
published: true
---

This week, progress focused on executing and validating **Task 1** under the guidance of mentor Jose Sir (`jmplaza`), evaluating the multi-sensor simulation capabilities of the Robotics Academy Docker Image (RADI) running on macOS (Apple Silicon via Rosetta 2).

---

## 1. Task 1 (Part A): Headless Simulation Server with RViz2 Multi-Sensor Validation

### Objective
To evaluate the simulation of a mobile robot equipped with an Onboard Camera, 2D LiDAR, Odometry, TF broadcasting, and Motor Control placed directly in front of an obstacle wall inside the RADI container, visualizing all data streams headlessly via RViz2 exported over noVNC.

### Architecture & Setup
* **Container Platform:** RADI (`jderobot/robotics-academy`) executed under `--platform linux/amd64` on macOS.
* **Physics & Sensor Simulation Engine:** Headless Gazebo Harmonic (`gz sim -s`) with the OGRE rendering plugin for offscreen sensor frame generation.
* **Robot Configuration:**
  * **Onboard RGB Camera Sensor:** Positioned on the front bumper, publishing to `/camera/image_raw` with real-time optical projection.
  * **2D Planar LiDAR Scanner:** Raycasting 180 horizontal samples over a 180-degree field of view, publishing to `/scan`.
  * **Odometry & TF Frame Broadcaster:** Tracking real-time displacement in the `odom` coordinate frame, publishing to `/odom` and `/tf`.
  * **Actuation System:** Differential drive motor controller subscribing to Twist velocity commands via `/cmd_vel`.
* **Environment:** A 3D solid obstacle wall placed at $X = 5.0\text{ m}$ directly in front of the robot.
* **Visualization Pipeline:** RViz2 rendered to X11 virtual display `DISPLAY=:1`, streamed via TurboVNC (Port 5900) and noVNC WebSocket proxy (Port 6080) to the macOS web browser.

### Multi-Sensor Testing & Verification
1. **Actuation & Odometry Verification:** Sent forward and angular velocity commands via `/cmd_vel`. The robot advanced towards the obstacle wall, and the odometry pose accurately tracked position changes in real time.
2. **LiDAR Distance Raycasts:** The planar `/scan` rays dynamically shortened in direct proportion to the physical distance to the wall.
3. **Real-time Optical Camera Perspective:** The `/camera/image_raw` feed rendered optical perspective scaling where the 3D orange obstacle expanded across the field of view as the robot closed distance. A sharp Telemetry HUD displayed live metrics:
   `[DISTANCE TO WALL: X.XX m] [POS: X=..., Y=...] [HEADING: ... DEG] [LIDAR: ... m]`
4. **Rigid Collision Boundary:** Verified rigid physical collision stopping at the obstacle boundary ($X \approx 4.15\text{ m}$, bumper distance $= 0.00\text{ m}$), preventing mesh penetration.

### Video Demonstration
The full multi-sensor simulation test, actuation response, and camera perspective scaling can be viewed below:

[![Task 1 Part A: Headless Simulation Server with RViz2 Validation](https://img.youtube.com/vi/N-JlsCwEfZE/hqdefault.jpg)](https://youtu.be/N-JlsCwEfZE)

*Video 1: Demonstration of Headless Simulation Server with RViz2 Multi-Sensor Validation on macOS.*

---

## 2. Task 1 (Part B): Standalone Gazebo GUI Viewer Evaluation (`--render-engine ogre`)

### Objective
To evaluate the standalone Gazebo Sim 8 GUI viewer client with the OGRE rendering engine (`--render-engine ogre`) alongside the headless simulation server on macOS, following the explicit environment setup directives.

### Execution Procedure
1. Sourced the ROS 2 Humble environment and exported virtual display parameters:
   ```bash
   export DISPLAY=:1
   export QT_QPA_PLATFORM=xcb
   export LIBGL_ALWAYS_SOFTWARE=1
   source /opt/ros/humble/setup.bash
   ```
2. Launched the headless Gazebo simulation server:
   ```bash
   gz sim -s -v 4 shapes.sdf &
   ```
3. Launched the standalone Gazebo GUI viewer with the OGRE rendering engine:
   ```bash
   gz sim -g -v 4 --render-engine ogre
   ```

### Technical Findings & Analysis
* **Server Status:** The headless Gazebo server (`gz sim -s`) successfully initialized physics, loaded the SDF world, and broadcasted scene states without errors.
* **GUI Client Behavior:** The standalone Gazebo GUI client (`gz sim -g`) initialized its Qt5 QML/QtQuick application context (`[GUI] Create main window`).
* **Root Cause Analysis:** Due to architectural constraints of software OpenGL emulation (`llvmpipe`) under Rosetta 2 x86 translation on Apple Silicon, the QtQuick 3D viewport fails to composite core profile shaders, resulting in an unrendered canvas.

### Video Demonstration
The standalone Gazebo server and OGRE GUI viewer execution test can be viewed below:

[![Task 1 Part B: Gazebo Standalone OGRE GUI Evaluation](https://img.youtube.com/vi/Hg59doWXcec/hqdefault.jpg)](https://youtu.be/Hg59doWXcec)

*Video 2: Demonstration of Standalone Gazebo GUI Viewer Execution with OGRE Engine on macOS.*

---

## 3. Summary & Key Conclusions

* **Simulation Engine (Headless Server):** Gazebo Sim 8 running headlessly inside RADI on macOS is 100% functional, accurately simulating kinematics, dynamics, and multi-sensor data streams (Camera, LiDAR, Odometry).
* **Visualization Stack:** While the standalone Gazebo Qt5 QML GUI is constrained by software OpenGL emulation under Rosetta 2, **RViz2 over noVNC** provides a fully stable, 30 FPS visualizer for all robotics tasks.
* **Outcome:** The Headless Server + RViz2 visualization pipeline is confirmed as the recommended architecture for macOS Apple Silicon RADI users.

---

## 4. Next Steps

* **Task 2:** Explore the Nav2 navigation stack locally using a native ROS 2 and Gazebo installation on macOS.
* **Task 3:** Explore Nav2-based solutions for RoboticsAcademy challenges on Windows using the official RADI environment.
