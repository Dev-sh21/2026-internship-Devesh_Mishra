---
title: "Internship Progress Week 15 (October 08 ~ October 14)"
date: 2026-10-14 08:30:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-15, ros2, nav2, gazebo, harmonic, radi, lyrical-beta, line-following, rtf, monaco]
published: true
---

## Nav2 Solution Upstream PR, Follow Line RTF Optimization, and Monaco Circuit Investigation

This report covers the progress made during **Week 15** (October 08 ~ October 14, 2026). Following the validation of exercises on the new `jderobot/robotics-academy:lyrical-beta` Docker image, our work this week focused on three main objectives:
1. Submitting the upstream Nav2 Amazon Warehouse solution to `JdeRobot/RoboticsAcademy-solutions` for testing by other developers and mentors.
2. Diagnosing and resolving the Real-Time Factor (RTF) fluctuations on the Follow Line exercise, bringing simulation performance to a steady ~1.0 on CPU runtime.
3. Investigating and implementing fixes for the Monaco Circuit performance bottleneck (Issue #3493 / RoboticsInfrastructure #632).

---

## 1. Video Demonstration

### Real-Time Factor (RTF) Stabilization on RADI lyrical-beta (Follow Line)

<div align="center" style="margin: 20px 0;">
  <iframe width="95%" height="480" src="https://www.youtube-nocookie.com/embed/xj7M6wV-nEg" title="RTF Stabilization on RADI lyrical-beta" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);"></iframe>
</div>

<p align="center">
  <a href="https://www.youtube.com/watch?v=xj7M6wV-nEg">
    <img src="https://img.youtube.com/vi/xj7M6wV-nEg/maxresdefault.jpg" alt="RTF Stabilization on RADI lyrical-beta" width="90%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
  </a>
</p>

*Direct YouTube Link: [https://youtu.be/xj7M6wV-nEg](https://youtu.be/xj7M6wV-nEg)*

---

## 2. Upstreaming the Amazon Warehouse Nav2 Solution

Following mentor recommendations to provide our Nav2 implementation to other developers for cross-platform testing, we prepared and submitted our contribution:

1. **Issue Raised:** Opened a tracking issue on [RoboticsAcademy-solutions](https://github.com/JdeRobot/RoboticsAcademy-solutions) outlining the architecture of the native ROS 2 Nav2 warehouse solution.
2. **Pull Request Submitted:** Submitted a clean, documented PR containing:
   - Asynchronous `NavigateToPose` action client integration.
   - Waypoint coordination for rack pick and drop cycles.
   - Platform motorized lift control interfacing with warehouse pallet geometry.
   - Usage documentation for reproducing the run across different environments.

---

## 3. Diagnosing and Fixing the RTF Fluctuation on Follow Line

During initial runs of the Follow Line exercise on CPU-only macOS/Rosetta environments, the Real-Time Factor (RTF) was fluctuating heavily between 0.1 and 1.1. Profiling revealed three primary CPU bottlenecks:

1. **Camera Sensor Overheads (`f1_gz.urdf.xacro`):**
   - The camera was rendering at 640x480 resolution with a per-pixel Gaussian noise filter and `<visualize>true</visualize>`.
   - On CPU software rendering (software OpenGL / llvmpipe), calculating Gaussian noise across every pixel at 20 Hz saturated CPU cores.
   - **Fix:** Reduced resolution to 320x240, removed the CPU noise filter plugin, and disabled visualization.

2. **World Physics & Rendering Configuration (`simple_circuit.world`):**
   - Simulation step sizes were previously unbounded, resulting in variable iteration workloads.
   - Dynamic shadows (`<shadows>true</shadows>`, `<cast_shadows>true</cast_shadows>`) incurred heavy software rasterization costs.
   - **Fix:** Configured an explicit 4ms physics time step (`<max_step_size>0.004</max_step_size>`, target RTF 1.0), disabled dynamic shadows, and capped `SceneBroadcaster` publication to 25 Hz.

3. **CPU Busy-Wait in Python Templates:**
   - In `HAL.py`, `getImage()` executed a tight `while image is None:` loop without yielding execution, consuming 100% of an entire CPU core.
   - **Fix:** Added `time.sleep(0.005)` inside the wait loop and corrected loop timing logic in `Frequency.py`.

**Outcome:** These optimizations eliminated the CPU spikes entirely, achieving a stable RTF of **~0.96 - 0.98 (~1.00)** as shown in the video demonstration.

---

## 4. Monaco Circuit Investigation & Ongoing Work (Issue #3493)

We investigated the Monaco Circuit performance degradation where RTF crashed down to 0.1 - 0.2 on 3D slopes and elevation changes:

1. **Root Cause Analysis:**
   - On flat tracks (Simple Circuit), mesh collision is trivial (mesh-vs-plane).
   - In Monaco, `monaco_track.dae` contains ~18,700 triangles across complex 3D hills and cambers.
   - The F1 car model was using the high-polygon visual mesh (`Car.obj` with ~36,000 triangles) directly for its `<collision>` geometry.
   - On elevation changes, DART physics was forced to calculate thousands of triangle-triangle contact manifolds every timestep, causing the solver to stall.

2. **Fixes Implemented:**
   - **Collision Primitives:** Replaced the heavy `Car.obj` mesh collision in `f1_common.urdf.xacro` with lightweight geometric primitives (1 box for chassis, 4 spheres for wheels).
   - **Contact Softening:** Configured surface contact parameters in `model.sdf` to eliminate edge chatter.
   - **Spawn Coordinates:** Corrected the initial vehicle pose in `world_db` and world files from off-track water coordinates `(-105.2, -70.7)` to the start line `(-0.5, 10.85, 0.35)`.
   - **Camera Tracking:** Updated `default.config` to bind Gazebo GUI tracking directly to the `f1` vehicle.

3. **Ongoing Work:**
   - Tuning PID lateral tracking and dynamic speed profiles to handle tight elevation changes and sharp corners (Sainte-Devote) reliably without track departure.

---

## 5. Summary & Next Steps

- Raised issue and submitted PR for the Amazon Warehouse Nav2 solution to `RoboticsAcademy-solutions`.
- Fixed the RTF fluctuation bottleneck on Follow Line, stabilizing execution at ~1.0 RTF on CPU runtime.
- Completed root-cause diagnosis and collision optimization for the Monaco Circuit; currently refining controller parameters for elevated track sections.
