---
layout: page
title: "Internship Progress Week 5"
permalink: /weekly-updates/week5/
---

This week, my progress was aligned with **Task 3: Play with a Nav-2 based solution for the Global Navigation exercise of RoboticsAcademy (using standard global navigation algorithms included in Nav2 instead of the custom GPP algorithm).**

Here is the progress detailing how we implemented Nav2 and the current blockages encountered:

---

## 1. Implementing Nav2-based Solution for Global Navigation (Task 3)

Instead of implementing a custom GPP path-planning algorithm, we worked on utilizing standard ROS 2 Humble Navigation Stack (Nav2) packages:

* **Static Map Configuration:** Created `cityLargenBin.yaml` to map the $500\text{m} \times 500\text{m}$ Gazebo city world coordinates to standard ROS 2 occupancy grid metrics (at `1.25` meters per pixel). This enables standard Nav2 map servers to feed the costmap.
* **Costmap and Nav2 Parameters:** Configured `planner_server` (using standard `NavfnPlanner` global path planner) and `controller_server` (using `RegulatedPurePursuitController` local tracker) inside `nav2_params.yaml`. Static costmaps were tuned to run without LiDAR sensor dependencies.
* **Bridge Interface (`nav2_bridge.py`):** Developed a ROS 2 bridge node to translate JdeRobot's WebGUI mouse click coordinates to standard `/goal_pose` topics and publish the resulting Nav2 planned paths back to the user GUI canvas.
* **Unified Launch Integration:** Combined the Gazebo world server, robot spawner, static transforms, map server, Nav2 planners, and the GUI bridge into a single executable launch file `global_navigation_nav2.launch.py` linked natively to the database.

---

## 2. Current Blockages & Unresolved Issues

We are currently blocked by the following errors, preventing final verification of the Nav2 navigation loops:

* **VNC Black Screen (UNRESOLVED):**
  The VNC desktop screen (displays `:1` and `:2`) remains completely black. The Gazebo GUI client fails to create its main rendering window, caused by Mesa software OpenGL emulation rendering bugs under Rosetta 2 translator on ARM64 macOS host.
* **Costmap TF Timeout Warnings:**
  The costmap server throws continuous warnings: `Timed out waiting for transform from base_link to map to become available...` because odometry is not published while the simulation physics is paused or the vehicle description joints are disconnected.
* **Verification Blocked:**
  Because the simulation frontend and window manager rendering is black, we cannot visually verify if the Nav2 planner successfully drives the taxi vehicle to the selected target.

---

## 3. Visual Verification of the Error

The following screenshot shows the active black screen error inside the VNC desktop display:

![VNC Black Screen Error](../docs/assets/img/posts/global_navigation_vnc.png)
*Figure: The VNC desktop display output showing the persistent black screen blockage.*

Below is the static city occupancy grid map used for costmap planning:

![Static Occupancy Grid Map](../docs/assets/img/posts/cityLargenBin.png)
*Figure: The binary occupancy grid map loaded into nav2_map_server.*
