# Weekly Update - Week 16

## Nav2 Global Navigation, Follow Line Demonstration on macOS, and Monaco 3D Circuit Benchmark

## Overview
This report covers the progress made during **Week 16** (October 15 ~ October 21, 2026). Our work this week focused on:
1. Validating and recording demonstrations for **Nav2 Global Navigation** and **Follow Line** running on macOS (Apple Silicon) without dedicated GPU.
2. Benchmarking the Monaco 3D Circuit performance and identifying key bottlenecks (Issue #3493 / RoboticsInfrastructure #632).

---

## Video Demonstrations

### 1. Nav2 Global Navigation on macOS (Apple Silicon, CPU Runtime)

Demonstration of autonomous navigation using ROS 2 and Navigation2 (Nav2) inside the updated RADI container on Apple Silicon without dedicated GPU acceleration.

<p align="center">
  <a href="https://www.youtube.com/watch?v=ERUeN0VsJEE">
    <img src="https://img.youtube.com/vi/ERUeN0VsJEE/maxresdefault.jpg" alt="Nav2 Global Navigation on macOS" width="90%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
  </a>
</p>

*Direct YouTube Link: [https://youtu.be/ERUeN0VsJEE](https://youtu.be/ERUeN0VsJEE)*

**Key Highlights:**
- **Stack:** ROS 2 + Nav2 (Navigation2) action client architecture.
- **Features:** Real-time global costmap, local costmap, and dynamic obstacle avoidance.
- **Platform:** Gazebo Sim running on macOS via Docker CPU runtime.

---

### 2. Follow Line on macOS with ROS 2 Lyrical & Gazebo Jetty

Demonstration of the Follow Line exercise running in ROS 2 Lyrical and Gazebo Jetty on macOS without GPU, validating camera stream capture, real-time image processing, and RTF performance.

<p align="center">
  <a href="https://www.youtube.com/watch?v=TeJxnLOyRio">
    <img src="https://img.youtube.com/vi/TeJxnLOyRio/maxresdefault.jpg" alt="Follow Line on macOS" width="90%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
  </a>
</p>

*Direct YouTube Link: [https://youtu.be/TeJxnLOyRio](https://youtu.be/TeJxnLOyRio)*

**Key Highlights:**
- **Stack:** ROS 2 Lyrical and Gazebo Jetty.
- **Vision:** OpenCV HSV color filtering and centroid tracking running at 30+ Hz.
- **Performance:** Maintained stable Real-Time Factor (RTF ~ 0.95 - 1.05) on macOS CPU.

---

## Monaco 3D Circuit Benchmark (Issue #3493)

We profiled and benchmarked the Monaco 3D Circuit ([Issue #3493](https://github.com/JdeRobot/RoboticsAcademy/issues/3493) / [RoboticsInfrastructure #632](https://github.com/JdeRobot/RoboticsInfrastructure/issues/632)):

- **Progress Made:** Replaced the heavy 31k-triangle `Car.obj` collision mesh with primitive colliders (box chassis, sphere wheels) in `f1_common.urdf.xacro`, generated a clean 3D collision mesh (`monaco_collision.dae`) with 1.2m boundary walls, and calibrated 2D map tracking in `showCarPositionFollowLine.js`.
- **Current Failure Point:** The vehicle navigates the first half of the track smoothly (Sainte Devote, Beau Rivage crest, Casino Square), but currently runs off track at the sharp downhill **Fairmont Hairpin (180° turn)** due to short camera lookahead and line loss at speed. Additionally, live RTF dips to ~0.60 on track due to the 250 Hz physics step size (`max_step_size 0.004`).
- **Planned Fix:** 
  1. Update physics step size to 0.008 (125 Hz) in `monaco_line.world` to cut solver load in half and lock RTF at a steady 1.00.
  2. Implement contour-based nearest-line tracking and predictive deceleration to slow the car ahead of hairpins and complete full laps reliably.

---

## Summary & Next Steps

- Published demonstration videos for Nav2 Global Navigation and Follow Line on macOS.
- Identified physics step size and hairpin steering lookahead as the core bottlenecks for Monaco.
- Implementing 125 Hz physics and contour-based cornering control to complete full Monaco laps.
