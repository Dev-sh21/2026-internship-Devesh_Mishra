# Weekly Update - Week 7

## Overview
This week, progress focused on executing and validating **Task 1** under mentor guidance, evaluating multi-sensor simulation and visualization workflows in RADI on macOS (Apple Silicon via Rosetta 2).

---

## Tasks Completed

### Task 1 (Part A): Headless Simulation Server with RViz2 Multi-Sensor Validation
* Executed headless Gazebo simulation with a mobile robot equipped with Onboard Camera, 2D LiDAR, Odometry, TF broadcasting, and Motor Control in front of a 5.0m obstacle wall.
* Validated real-time optical perspective rendering on `/camera/image_raw` with live Telemetry HUD overlay.
* Validated planar laser raycast shortening on `/scan` and displacement tracking on `/odom`.
* Validated rigid body collision stopping at obstacle boundary ($X \approx 4.15\text{ m}$).
* Recorded and uploaded video demonstration: [Watch Part A Video on YouTube](https://youtu.be/N-JlsCwEfZE).

### Task 1 (Part B): Standalone Gazebo GUI Viewer Evaluation (`--render-engine ogre`)
* Tested headless server (`gz sim -s -v 4 shapes.sdf`) alongside standalone GUI viewer (`gz sim -g -v 4 --render-engine ogre`) with explicit virtual display variables (`export DISPLAY=:1`, `export LIBGL_ALWAYS_SOFTWARE=1`).
* Documented findings: Headless server runs physics without error; standalone QtQuick/QML GUI fails to composite OpenGL 3.3 Core profile shaders under macOS Rosetta 2 software OpenGL (`llvmpipe`).
* Recorded and uploaded video demonstration: [Watch Part B Video on YouTube](https://youtu.be/Hg59doWXcec).

---

## Key Conclusions
* Running Gazebo headlessly while offloading visualization to RViz2 over noVNC provides a stable, 30 FPS multi-sensor simulation environment on macOS.

---

## Next Steps
* **Task 2:** Explore Nav2 stack locally on macOS with native ROS 2 + Gazebo installation.
* **Task 3:** Explore Nav2 challenge solutions on Windows using official RADI.
