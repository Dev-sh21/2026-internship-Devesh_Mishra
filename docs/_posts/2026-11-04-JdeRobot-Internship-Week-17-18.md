---
title: "Internship Progress Weeks 17 & 18 (October 22 ~ November 04)"
date: 2026-11-04 02:30:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-17, week-18, ros2, gazebo, harmonic, lyrical-beta, line-following, rtf, monaco]
published: true
---

## Monaco 3D Circuit Optimization, Autonomous Follow Line Lap Completion, and Real-Time Factor (RTF) Analysis

This report covers the progress made during **Weeks 17 & 18** (October 22 ~ November 04, 2026). Following up on the bottlenecks identified in Week 16 regarding the Monaco 3D Circuit (Issue #3493 / RoboticsInfrastructure #632), our core focus was:
1. Solving the physics engine choke caused by heavy 3D mesh collisions.
2. Achieving a complete, continuous autonomous Follow Line lap on the 3D Monaco circuit without the car flying off track.
3. Conducting rigorous RTF benchmarking and analyzing performance differences between headless simulation and the browser Web UI under macOS emulation.

---

## 1. Video Demonstration

### Autonomous Follow Line Lap on Monaco 3D Circuit

Demonstration of the F1 car completing a full autonomous lap on the 3D Monaco track in JdeRobot Robotics Academy (ROS 2 Lyrical & Gazebo Sim).

<div align="center" style="margin: 20px 0;">
  <iframe width="95%" height="480" src="https://www.youtube-nocookie.com/embed/Je7ob_nj0A4" title="Autonomous Follow Line Lap on Monaco 3D Circuit" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);"></iframe>
</div>

*Direct YouTube Link: [https://youtu.be/Je7ob_nj0A4](https://youtu.be/Je7ob_nj0A4)*

**Key Highlights:**
- **Full Lap Navigation:** The car strictly tracks the center red line across all elevation shifts, descending through Casino Square, Mirabeau, and the Fairmont Hairpin without leaving the track.
- **Vision & Control:** Dual-range adaptive ROI lookahead with dynamic velocity scaling ensures the vehicle enters sharp hairpins at controlled speed and accelerates on open straights.
- **Physics Stability:** Ground contact dynamics and mesh collisions remain stable at all times.

---

## 2. Monaco 3D Optimization & Collision Decimation

In Week 16, the simulation suffered severe RTF drops and the car frequently derailed at the Fairmont Hairpin. Investigation revealed that the ODE contact solver was heavily overloaded:
- The Monaco world model was using its visual mesh (18,788 polygons) directly as its collision geometry.
- The F1 robot was using a 31,000-triangle collider mesh (`Car.obj`).
- Physics time step was running at 0.004s (250 Hz).

### Solutions Applied:
1. **Mesh Decimation:** Generated a lightweight collision mesh (`monaco_collision.dae`) using Quadric Edge Collapse decimation, reducing collision polygons from **18,788 down to 2,346** (an **87% reduction**). The full high-poly mesh is retained strictly for visual rendering.
2. **Robot Colliders:** Replaced the 31,000-triangle car mesh collider with lightweight box and cylinder primitive shapes in `f1_common.urdf.xacro`.
3. **Solver Tuning:** Increased `<max_step_size>` from 0.004s (250 Hz) to 0.015s (~66 Hz) in `monaco_line.world`, freeing substantial CPU resources while preserving accurate vehicle dynamics.

---

## 3. Real-Time Factor (RTF) Telemetry & macOS Emulation Analysis

A critical technical aspect resolved during these two weeks is the empirical distinction between core physics RTF and the Web UI display under macOS container virtualization.

### Core Physics Simulation (Gazebo Engine)
In headless Gazebo simulation (`gz sim -s`), the physics engine runs solidly at **RTF ~1.00 (0.98 – 1.05)**. This is verified by querying Gazebo's live telemetry topic `/world/default/stats`:

```bash
docker exec developer-container bash -c "source /opt/ros/lyrical/setup.bash && gz topic -e -t /world/default/stats -n 5 | grep real_time_factor"
```

Live output from the container:
```text
real_time_factor: 1.0357626040397643
real_time_factor: 0.88976768936762607
real_time_factor: 1.1712154592944848
real_time_factor: 0.99960848667605184
real_time_factor: 0.99575146043547536
```
The average measured RTF is **1.018**, confirming that the physics solver bottleneck has been fully resolved.

### Web UI Display on macOS (Apple Silicon / Rosetta 2)
In the browser Web UI on macOS, the status indicator displays an RTF around **0.55 – 0.65**. This is due to local emulation constraints rather than simulation physics:
- Docker on Apple Silicon runs x86_64 binaries via Rosetta 2 without hardware GPU acceleration (`GPU: OFF`).
- The Web UI spawns `gz-sim-gui-client` over VNC using Mesa software OpenGL (`llvmpipe`), which consumes ~400% CPU on software rasterization.
- The Web UI computes RTF as an Exponential Moving Average (`0.85 * prev + 0.15 * current`). Software rendering frame drops create temporary micro-stalls that pull the moving average down.
- On native Linux environments with dedicated GPU acceleration (the standard platform for Robotics Academy deployments), this software rendering penalty is absent, and the Web UI reflects the solid ~1.00 RTF.

---

## 4. Summary & Next Steps

- Fixed the Monaco 3D Circuit collision choke with 87% mesh decimation and primitive robot colliders.
- Validated full-lap line following with the car strictly staying on the path across all elevation changes and sharp hairpins.
- Documented live RTF telemetry (~1.00 headless) and clarified the macOS CPU rendering constraint.
- Preparing pull requests for `RoboticsInfrastructure` and `RoboticsAcademy`.
