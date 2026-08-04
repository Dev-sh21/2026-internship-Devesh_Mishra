---
title: "Internship Progress Week 5 (July 30 ~ August 05)"
date: 2026-08-05 08:00:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-5, gazebo, ros2, vnc, nav2, bugs]
published: true
---

This week, my work focused on integrating the ROS 2 Humble Navigation Stack (Nav2) with the Global Navigation exercise in JdeRobot's RoboticsAcademy. However, we are currently facing major blockages that prevent the simulation from rendering.

---

## 1. ROS 2 Humble Nav2 Integration Work

We configured the baseline packages for the Nav2-based path planning:
* **Static Map Configuration:** Created `cityLargenBin.yaml` to map the city world at a resolution of `1.25` meters per pixel.
* **Tuning costmaps:** Configured the planner and controller servers in `nav2_params.yaml`.
* **GUI Bridge:** Wrote a Python bridge (`nav2_bridge.py`) to connect WebGUI websocket messages with ROS2 Nav2 standard topics.

---

## 2. Ongoing Issues & Blockages (Not Fixed Yet)

We are currently blocked by the following unresolved bugs:

* **VNC Black Screen rendering crash (UNRESOLVED):**
  The VNC desktop screen (display `:2` and display `:1`) remains completely black. The Gazebo GUI client fails to render its main window, likely due to Mesa/OpenGL software rendering compiler failures or driver incompatibilities under Rosetta 2 emulation on Apple Silicon macOS host.
* **Costmap TF Timeout Warnings:**
  The `controller_server` prints recurring errors: `Timed out waiting for transform from base_link to map to become available, tf error: Could not find a connection because they are not part of the same tree.` This occurs because odometry is not published, as the simulation remains paused or the robot spawner transforms are unconnected.
* **Nav2 Parameters Typos & Crashes:**
  While the parameter syntax changes (changing `/` to `::` for controllers and fixing `plugin_lib_names`) were applied, we cannot verify their correctness because the overall simulator remains frozen.

---

## 3. Visual Verification of the Error

The following screenshot shows the active black screen error inside the VNC desktop display:

![VNC Black Screen Error](/assets/img/posts/global_navigation_vnc.png)
*Figure: The VNC desktop display output showing the persistent black screen blockage.*

Below is the static city occupancy grid map used for costmap planning:

![Static Occupancy Grid Map](/assets/img/posts/cityLargenBin.png)
*Figure: The binary occupancy grid map loaded into nav2_map_server.*
