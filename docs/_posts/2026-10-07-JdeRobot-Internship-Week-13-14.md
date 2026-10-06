---
title: "Internship Progress Weeks 13 & 14 (September 24 ~ October 07)"
date: 2026-10-07 01:00:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-13, week-14, ros2, nav2, gazebo, harmonic, radi, lyrical-beta, line-following, amazon-warehouse]
published: true
---

## Validating Exercises on the New RADI lyrical-beta Image and Implementing Full Nav2 Amazon Warehouse Automation

This combined report covers the progress made across **Weeks 13 and 14**. Following guidance and recommendations from lead mentor **José María Plaza (@jmplaza)**, we transitioned our development and validation workflow to JdeRobot's new in-development Docker image: **RADI lyrical-beta** (`jderobot/robotics-academy:lyrical-beta`).

This next-generation image brings native ROS 2 Lyrical integration together with the modernized **Gazebo Harmonic (gz-sim)** simulation engine and an updated Robotics Application Manager (RAM) toolchain. Working directly on this development-stage image allowed us to test upcoming platform features, resolve low-level runtime integration issues, and successfully build and validate two core exercises: **Autonomous Line Following** and the **Amazon Warehouse** logistics pipeline.

---

## 1. Video Demonstrations & Previews

### A. Autonomous Line Following (Vision & PID on RADI lyrical-beta)

<div align="center" style="margin: 20px 0;">
  <iframe width="95%" height="480" src="https://www.youtube-nocookie.com/embed/BsxOF49f_bM" title="Autonomous Line Following on RADI lyrical-beta" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);"></iframe>
</div>

<p align="center">
  <a href="https://www.youtube.com/watch?v=BsxOF49f_bM">
    <img src="https://img.youtube.com/vi/BsxOF49f_bM/maxresdefault.jpg" alt="Autonomous Line Following on RADI lyrical-beta" width="90%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
  </a>
</p>

*Direct YouTube Link: [https://youtu.be/BsxOF49f_bM](https://youtu.be/BsxOF49f_bM)*

---

### B. Amazon Warehouse Logistics (ROS 2 Nav2 on RADI lyrical-beta)

<div align="center" style="margin: 20px 0;">
  <iframe width="95%" height="480" src="https://www.youtube-nocookie.com/embed/2nGsmhxEtA0" title="Amazon Warehouse Nav2 Autonomous Run on RADI lyrical-beta" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);"></iframe>
</div>

<p align="center">
  <a href="https://www.youtube.com/watch?v=2nGsmhxEtA0">
    <img src="https://img.youtube.com/vi/2nGsmhxEtA0/maxresdefault.jpg" alt="Amazon Warehouse Nav2 Autonomous Run on RADI lyrical-beta" width="90%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
  </a>
</p>

*Direct YouTube Link: [https://youtu.be/2nGsmhxEtA0](https://youtu.be/2nGsmhxEtA0)*

---

## 2. Migration to the New RADI lyrical-beta Image

Following guidance from José María Plaza (@jmplaza), we migrated our environment away from older base containers to focus on the `jderobot/robotics-academy:lyrical-beta` build.

Key architectural improvements in the new image:
- **Modern Simulation Stack:** Native Gazebo Harmonic (`gz-sim`) replacing older simulation binaries.
- **Updated Middleware:** Direct ROS 2 communication nodes without legacy ROS 1 / ROS 2 bridge bottlenecks.
- **Modern Web Application Architecture:** Integrated RAM WebSocket server and React-based frontend telemetry.

Because the image is actively in development, getting our workflow properly configured required resolving containerized Python virtual environment (`/.venv`) path configurations, workspace sourcing scripts, and TurboVNC / noVNC headless rendering pipelines on macOS Apple Silicon.

---

## 3. Autonomous Line Following on RADI lyrical-beta

Using the new RADI environment, we ported and validated the visual servoing control loop for the Formula 1 racing car:

1. **Perception Pipeline:**
   - Captured front-facing monocular camera frames via ROS 2 image transports.
   - Converted color representations from BGR to HSV for reliable segmentation of the guidance line under variable lighting.
   - Applied morphological opening and closing to suppress ground noise.
   - Calculated spatial image moments ($M_{00}$, $M_{10}$, $M_{01}$) to determine the line centroid:
     $$c_x = \frac{M_{10}}{M_{00}}$$

2. **Control Loop & Dynamic Velocity Profiling:**
   - Evaluated lateral tracking error: $e_t = \text{image\_center}_x - c_x$.
   - Implemented a discrete PID controller generating smooth steering commands ($\omega_z$).
   - Implemented dynamic velocity regulation that scales linear throttle inversely with steering curvature, preventing track departure on hairpin turns.

---

## 4. Amazon Warehouse with Full ROS 2 Nav2 Autonomy

Building upon our initial exploration from Week 12, we completed the full autonomous logistics pipeline for the Amazon Warehouse using the **ROS 2 Nav2** architecture on the new `lyrical-beta` image.

### Technical Architecture & Workflow
1. **Nav2 NavigateToPose Action Client:**
   Implemented a dedicated ROS 2 Action Client communicating with the Nav2 server to dispatch goals asynchronously.
2. **Global A* Path Planning:**
   The navigation server ingests the 2D warehouse occupancy grid, applies obstacle inflation for shelf buffers, and computes optimal collision-free routes.
3. **Automated Pallet Handling Sequence:**
   - **Phase 1 (Dispatch):** Robot navigates to the target pallet rack at `(X: 3.84, Y: 0.54)`.
   - **Phase 2 (Pick):** Robot centers beneath the pod bay and commands the motorized vertical platform lift (`/logistic_robot/platform/cmd_vel`) to raise the rack.
   - **Phase 3 (Transit):** Nav2 plans a collision-free return route to the fulfillment area at `(X: 0.0, Y: 0.0)`.
   - **Phase 4 (Drop):** Lowers the platform and confirms task completion.

---

## 5. Debugging & Infrastructure Fixes on the New Development Stack

While validating the exercises on the new `lyrical-beta` build, we identified and resolved critical interface bugs:

1. **WebGUI WebSocket Async/Sync Interface Crash:**
   - **Bug:** In `measuring_threading_gui_harmonic.py`, the GUI sender thread executed `await self.update_gui()`. In `WebGUI.py`, `update_gui()` is a standard synchronous method returning `None`. This raised `TypeError: 'NoneType' object can't be awaited`, killing the telemetry loop immediately.
   - **Fix:** Added inspectable awaiting (`if inspect.isawaitable(res): await res`) and made `send_to_client()` queue-safe.

2. **WebSocket Handshake Deadlock:**
   - **Bug:** `MeasuringThreadingGUI` initialized `ack_frontend = False`, stalling telemetry output if the frontend sent the `start` handshake prior to the internal WebSocket connection establishing.
   - **Fix:** Initialized `ack_frontend = True` and introduced a 0.5s timeout fallback so telemetry streams continuously.

3. **Map Coordinate Orientation & Trajectory Rendering:**
   - **Bug:** Nav2 server published waypoints as `[px_x, px_y]`, whereas the frontend `updatePath` scales `element[0]` by canvas `width` (`left`) and `element[1]` by canvas `height` (`top`).
   - **Fix:** Re-aligned waypoint coordinates to `[round(px_y, 1), round(px_x, 1)]`, rebuilt `common.zip`, and recompiled the React frontend bundle. Both the live vehicle marker (`#vehic-pos`) and the green trajectory line now render accurately on the 2D map.

---

## Summary & Next Steps

- Successfully validated both the Autonomous Line Following and Amazon Warehouse Nav2 exercises on the in-development **RADI lyrical-beta** image recommended by mentor José María Plaza.
- Documented and resolved low-level interface bugs in the next-generation simulation and telemetry stack.
- Next steps involve benchmarking Real-Time Factor (RTF) across different host environments and testing further exercises slated for the new RADI release.
