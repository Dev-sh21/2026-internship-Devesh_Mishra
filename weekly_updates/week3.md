---
layout: page
title: "Internship Progress Week 3"
permalink: /weekly-updates/week3/
---

This week, my work focused on verifying OpenGL rendering in Gazebo using a camera sensor across different installation methods, investigating the Gazebo Scene Viewer, and setting up standalone VNC servers and clients to run graphical applications.

---

## 1. Gazebo Camera Simulation: Source Build vs. Package Installation

The objective of this task was to replace the previous Lidar sensor with an OpenGL Camera sensor (`<sensor type="camera">`) on our differential drive robot, and verify its rendering capabilities under both installation environments on macOS.

### Key Implementations:
* **Camera Integration:** Configured the camera inside the robot's `.sdf` file with a 640x480 resolution, a 1.05-radian horizontal field of view, and an `R8G8B8` color format.
* **Autonomous Navigation:** Developed a Python control script `obstacle_avoidance.py` that listens to `/robot/laser` range scans and publishes twist commands (`/cmd_vel`) to steer the camera robot dynamically around obstacles (Red Cube, Green Cylinder, Yellow Pillar).

### Testing Environments:
* **Source Build Workspace:** Tested under the `gazebo_source_ws` workspace. To prevent macOS System Integrity Protection (SIP) from stripping `DYLD_LIBRARY_PATH` when calling the Ruby-wrapped `gz` CLI, we forced execution via the Homebrew Ruby interpreter (`/opt/homebrew/bin/ruby`).
* **Binary Packages:** Tested natively using the Homebrew package installation of Gazebo (`gz-harmonic`).

### Video Documentation:
* Watch the simulation run and autonomous obstacle avoidance: [Watch Video](https://youtu.be/QK7v57S1Ais)

---

## 2. Investigating the Gazebo Scene Viewer

I investigated the features of the Gazebo Scene Viewer (the 3D visualization canvas) to verify OpenGL hardware rendering efficiency:

* **Visualizing Sensors:** Enabled sensor visualization to view the yellow translucent camera frustum (field-of-view cone) projecting from the robot.
* **Collision Outlines:** Toggled collision geometries to inspect wireframe bounding boxes of the chassis, wheels, and obstacles.
* **Image Display Plugin:** Loaded the Image Display plugin in the GUI, subscribing to `/camera/image_raw` to monitor the live camera feed within the scene viewport.
* **Performance Analysis (FPS):** Inspected the FPS (Frames Per Second) counter in the bottom status bar. A steady 30-60 FPS verified active OpenGL hardware acceleration.

### Video Documentation:
* Watch the Scene Viewer features and FPS check: [Watch Video](https://youtu.be/6WfaOMMUs5A)

---

## 3. Running Standalone VNC Server & Client (Outside Robotics Academy)

To test VNC functionality independently of the Robotics Academy framework, I set up a standalone Dockerized desktop container containing a VNC server and web client.

* **VNC Server Setup:** Started a container mapping the VNC RFB port (`5901`) and the noVNC HTTP port (`6080`).
* **VNC Clients Verified:**
  * **Web Client (noVNC):** Accessed the desktop remotely using a standard web browser at `http://localhost:6080`.
  * **Native Client:** Connected via the native macOS Screen Sharing client using `vnc://localhost:5901` from Finder.

### Video Documentation:
* Watch the VNC server connection and client testing: [Watch Video](https://youtu.be/3hWCTyUlOGc)
