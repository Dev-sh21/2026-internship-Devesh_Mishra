# Weekly Update - Week 15

## Nav2 Solution Upstream PR, Follow Line RTF Optimization, and Monaco Circuit Investigation

## Overview
This report covers the progress made during **Week 15** (October 08 ~ October 14, 2026). Following the validation of the exercises on the new `jderobot/robotics-academy:lyrical-beta` Docker image, our primary focus this week was threefold:
1. Packaging and submitting our ROS 2 Nav2 Amazon Warehouse solution to `JdeRobot/RoboticsAcademy-solutions` for community and mentor testing.
2. Diagnosing and resolving the Real-Time Factor (RTF) fluctuations on the Follow Line exercise, bringing simulation performance to a stable ~1.0 on CPU runtime.
3. Investigating and implementing fixes for the Monaco Circuit performance bottleneck (Issue #3493 / RoboticsInfrastructure #632).

---

## Video Demonstration

### Real-Time Factor (RTF) Stabilization on RADI lyrical-beta (Follow Line)

<p align="center">
  <a href="https://www.youtube.com/watch?v=xj7M6wV-nEg">
    <img src="https://img.youtube.com/vi/xj7M6wV-nEg/maxresdefault.jpg" alt="RTF Stabilization on RADI lyrical-beta" width="90%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
  </a>
</p>

*Direct YouTube Link: [https://youtu.be/xj7M6wV-nEg](https://youtu.be/xj7M6wV-nEg)*

---

## 1. Upstreaming the Amazon Warehouse Nav2 Solution

Following mentor recommendations to provide the Nav2 implementation for cross-developer testing, we structured and contributed our complete solution:

1. **Issue Raised:** Opened a tracking issue on [RoboticsAcademy-solutions](https://github.com/JdeRobot/RoboticsAcademy-solutions) outlining the migration from basic navigation to the native ROS 2 Nav2 action architecture.
2. **Pull Request Submitted:** Submitted a clean, modular PR with our complete solution code including:
   - Asynchronous `NavigateToPose` action client integration.
   - Waypoint coordination for rack pick and drop cycles.
   - Platform motorized lift control interfacing with warehouse pallet geometry.
   - Documentation and usage instructions for reproducing the run.

---

## 2. Diagnosing and Fixing the RTF Fluctuation on Follow Line

During initial runs of the Follow Line exercise on CPU-only macOS/Rosetta environments, the Real-Time Factor (RTF) exhibited severe volatility, oscillating erratically between 0.1 and 1.1. Profiling revealed three primary CPU bottlenecks:

1. **Camera Sensor Overheads (`f1_gz.urdf.xacro`):**
   - The onboard camera was configured to render at 640x480 resolution with a per-pixel Gaussian noise filter and `<visualize>true</visualize>`.
   - On CPU software rendering (software OpenGL / llvmpipe), calculating Gaussian noise across every pixel at 20 Hz saturated CPU cores.
   - **Fix:** Scaled image resolution to 320x240, removed the CPU noise plugin, and disabled visualization.

2. **World Physics & Rendering Configuration (`simple_circuit.world`):**
   - Simulation step sizes were previously unbounded, resulting in variable iteration workloads.
   - Dynamic shadows (`<shadows>true</shadows>`, `<cast_shadows>true</cast_shadows>`) incurred heavy software rasterization costs.
   - **Fix:** Configured an explicit 4ms physics time step (`<max_step_size>0.004</max_step_size>`, target RTF 1.0), disabled dynamic shadows, and capped `SceneBroadcaster` publication to 25 Hz.

3. **CPU Busy-Wait in Python Templates:**
   - In `HAL.py`, `getImage()` executed a tight `while image is None:` loop without yielding execution, consuming 100% of an entire CPU core.
   - **Fix:** Added `time.sleep(0.005)` inside the wait loop and corrected loop timing logic in `Frequency.py`.

**Outcome:** These optimizations eliminated the CPU spikes entirely, achieving a stable RTF of **~0.96 - 0.98 (~1.00)** as verified in the linked video demonstration.

---

## 3. Monaco Circuit Investigation & Ongoing Work (Issue #3493)

We investigated the Monaco Circuit performance degradation where RTF crashed down to 0.1 - 0.2 on 3D slopes and elevation changes:

1. **Root Cause Analysis:**
   - On flat tracks (Simple Circuit), mesh collision is trivial (mesh-vs-plane).
   - In Monaco, `monaco_track.dae` has ~18,700 triangles across complex 3D hills and cambers.
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

## Summary & Next Steps

- Raised issue and submitted PR for the Amazon Warehouse Nav2 solution to `RoboticsAcademy-solutions`.
- Fixed the RTF fluctuation bottleneck on Follow Line, stabilizing execution at ~1.0 RTF on CPU runtime.
- Completed root-cause diagnosis and collision optimization for the Monaco Circuit; currently refining controller parameters for elevated track sections.
