---
title: "Internship Progress Week 12 (September 17 ~ September 23)"
date: 2026-09-23 18:00:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-12, ros2, jazzy, jderobot, gazebo, harmonic, jetty, amazon-warehouse, nav2]
published: true
---

## Porting and Testing ROS 2 Jazzy and Amazon Warehouse

This week, we focused on testing and extending the cross-platform capabilities of Robotics Academy on macOS Apple Silicon, validating the core stack on **ROS 2 Jazzy** and running the **Amazon Warehouse** exercise.

---

## 1. Testing and Validating ROS 2 Jazzy

We began by testing our updated environment running **ROS 2 Jazzy Jalisco**. All primary subsystems—including workspace compilation, node execution, and bridging with the new Gazebo Harmonic / Jetty environment—passed validation without requiring Rosetta 2 translation.

---

## 2. Implementing Amazon Warehouse

Following the environment tests, we tackled the **Amazon Warehouse** exercise, which involves an autonomous holonomic robot navigating warehouse aisles to pick up pallets and deliver them to a packing zone.

### Nav2 Architecture vs. Classical Approach
Initially, the objective was targeted towards running the exercise through the **Nav2 (Navigation2)** stack, using action clients to send target poses. However, the fundamental design of this exercise in Robotics Academy emphasizes learning low-level robotics concepts (such as coordinate transformations, A* path planning, and direct motor velocity control). In this container configuration, the full Nav2 background server stack is not spun up by default. 

To achieve full autonomy while honoring the container environment, we developed an autonomous navigation and task execution state machine using odometry feedback and velocity commands.

### Resolving Backend Template Bugs
During testing, we discovered two major bugs in the upstream `HAL.py` and `WebGUI.py` templates for Gazebo Harmonic:
1. **Namespace Mismatch:** The default templates attempted to publish and subscribe under the `/amazon_robot/` namespace (e.g., `/amazon_robot/odom` and `/amazon_robot/cmd_vel`). In Gazebo Harmonic, the active robot entity publishes under `/logistic_robot/`.
2. **Platform Actuator Topic Bug:** The pallet lift function published to `/platform/cmd_vel`, whereas the simulator listens on `/logistic_robot/platform/cmd_vel`.
3. **Map Display Synchronization:** The web map viewer was bound to the legacy odometry topic, preventing the position indicator (red marker) from updating.

By applying monkey-patches in our control script, we re-routed all motor controllers, odometry streams, platform lift triggers, and the GUI map pose callback to the active `/logistic_robot` topics.

### Manhattan Routing (Aisle Navigation)
To prevent collisions with warehouse shelves, we implemented **Manhattan Routing** (corridor navigation). Instead of driving along a direct diagonal vector—which clips the corners of adjacent shelves—the robot navigates along the main aisle before making a 90-degree turn directly into the target shelf.

Using coordinates extracted directly from Gazebo state telemetry:
- **Pallet 0:** `(X: 3.84, Y: 0.54)`
- **Packing Area:** `(X: 0.0, Y: 0.0)`

The robot aligns with the aisle, enters the pod bay, activates the platform lift, exits back into the clear corridor, and delivers the pallet to the packing station.

---

## Demonstration Video

Watch the complete autonomous execution of the Amazon Warehouse pallet pick-and-place sequence:

[![Amazon Warehouse Autonomous Run](https://img.youtube.com/vi/JP5-pee_yoE/0.jpg)](https://www.youtube.com/watch?v=JP5-pee_yoE)

*(Click the image above to watch the video on YouTube)*

---

## Simulation Screenshots

Below are screenshots of the Amazon Warehouse simulation and navigation interface:

<div align="center">
  <img src="{{ site.baseurl }}/assets/img/posts/amazonWarehouse_1.png" alt="Amazon Warehouse Nav2 Simulation" style="width: 90%; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15); margin-bottom: 20px;" />
  <img src="{{ site.baseurl }}/assets/img/posts/amazonWarehouse_2.png" alt="Amazon Warehouse Execution View" style="width: 90%; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);" />
</div>
