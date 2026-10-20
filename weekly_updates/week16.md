# Weekly Update - Week 16

## Nav2 Global Navigation, Follow Line Demonstration on macOS, and Monaco 3D Circuit Benchmark

## Overview
This report covers the progress made during **Week 16** (October 15 ~ October 21, 2026). Following the release of the updated `jderobot/robotics-academy:lyrical-beta` Docker container, our work this week focused on three major milestones:
1. Recording and validating full end-to-end demonstrations of **Nav2 Global Navigation** and **Follow Line** running natively on macOS (Apple Silicon) without dedicated GPU acceleration.
2. In-depth benchmarking and root-cause analysis of the Monaco Circuit 3D performance issue (Issue #3493 / RoboticsInfrastructure #632).
3. Formulating and testing concrete engineering fixes for the Fairmont Hairpin track departure and on-track Real-Time Factor (RTF) stabilization.

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
- **Navigation Stack:** ROS 2 + Nav2 (Navigation2) action client architecture.
- **Path Planning:** Real-time global costmap and local costmap generation with dynamic obstacle avoidance.
- **Simulation Environment:** Gazebo Sim running smoothly inside Docker on macOS via CPU software rasterization.
- **Result:** Successfully reached designated target waypoints with obstacle clearance and zero controller timeouts.

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
- **Vision Pipeline:** OpenCV HSV color segmentation and centroid calculation running at 30+ Hz.
- **Performance:** Maintained stable Real-Time Factor (RTF ~ 0.95 - 1.05) on macOS CPU with low latency WebGUI streaming.

---

## 1. Monaco 3D Circuit: Progress & Current Status (Issue #3493 / #632)

We conducted deep performance profiling and kinematic analysis of the Monaco Circuit 3D track ([Issue #3493](https://github.com/JdeRobot/RoboticsAcademy/issues/3493) / [RoboticsInfrastructure #632](https://github.com/JdeRobot/RoboticsInfrastructure/issues/632)).

### Work Completed So Far:
1. **Collision Mesh Optimization:** Replaced the 31,449-face visual mesh `Car.obj` in `f1_common.urdf.xacro` with optimized geometric collision primitives (1 box chassis collider, 4 sphere wheel colliders), eliminating DART solver stalls.
2. **Seamless Track Collision Mesh:** Built a clean, continuous 3D collision mesh (`monaco_collision.dae`) preserving exact elevation changes (-1.80m to +7.07m) and adding continuous 1.2m boundary barrier walls.
3. **2D Map & Red Dot Calibration:** Generated a synchronized 2D track map and implemented mathematical coordinate projection with 315-degree rotation in `showCarPositionFollowLine.js`, achieving sub-pixel red dot tracking.
4. **Camera View Configuration:** Created and registered a dedicated follow camera configuration (`monaco_circuit.config`) in PostgreSQL `academy_db` and world launchers.

---

## 2. Current Failure Points (Where It Fails Right Now)

Our testing across full lap runs revealed two specific issues that require resolution:

### A. Track Departure at Fairmont Hairpin (Grand Hotel Hairpin)
- **Symptom:** The F1 car successfully navigates Sainte Devote, the Beau Rivage uphill crest, Massenet, and Casino Square. However, on the downhill approach into the **Fairmont Hairpin (180° tight left turn)**, the vehicle runs off the circuit onto the grass embankment.
- **Root Cause Analysis:**
  1. **Short Lookahead Window:** Using a narrow near-bumper region of interest (rows 140–215) looks only ~1.5m ahead. When entering a 180° hairpin at velocity, the line bends and exits the camera frame abruptly before the steering controller can apply sufficient yaw rate.
  2. **Centroid Contamination:** Because the hairpin loops tightly below the current roadway, wide vertical ROIs capture both the immediate road line and the lower track loop, dragging the centroid toward the center and causing understeer.
  3. **Blind Recovery Fallback:** When the line is lost (`M["m00"] == 0`), relying strictly on `last_error` can command steering in the wrong direction if the prior straight section had an opposing curve bias.

### B. On-Track Real-Time Factor (RTF) Dips (0.56 ~ 0.70)
- **Symptom:** While the car is driving normally on the asphalt surface, live RTF drops to between 0.56 and 0.70 instead of holding at 1.00.
- **Root Cause Analysis:**
  - In `monaco_line.world`, physics is configured with `<max_step_size>0.004</max_step_size>`, requiring 250 physics iterations per real-world second.
  - In CPU software rendering (via TurboVNC / llvmpipe on macOS), actual computation takes ~5.28 ms per step.
  - Since 5.28 ms exceeds the 4.0 ms threshold, the physics engine can only execute ~189 steps per second ($189 / 250 \approx 0.75$ RTF), further dipping to ~0.56 under CPU contention from the Python control loop.

---

## 3. Planned Fixes & Implementation Strategy

To address both failure points, the following engineering solutions are being deployed:

1. **Physics Step Size Tuning (125 Hz):**
   - Update `<max_step_size>` in `monaco_line.world` from `0.004` (250 Hz) to `0.008` (125 Hz).
   - This reduces the physics iteration workload by 50%, providing an 8.0 ms budget per step. Because actual execution takes ~5.3 ms, RTF will comfortably lock at a rock-solid **1.00** on macOS CPU.
   - 125 Hz remains fully stable for vehicle kinematics at speeds between 1 and 4 m/s.

2. **Multi-Contour Nearest-Line Selection:**
   - Replace blind moments with contour segmentation (`cv2.findContours`).
   - Filter contours by proximity to the bottom edge of the image frame, guaranteeing that only the track immediately under the vehicle is followed and completely ignoring distant loops.

3. **Dual-Horizon Predictive Deceleration:**
   - Introduce a far lookahead point (row 100) to detect sharp curvature 2–3 seconds in advance.
   - Automatically decelerate the vehicle to `0.5 - 0.6 m/s` ahead of hairpin entry, granting the controller ample time to execute sharp steering (`w >= 2.5 rad/s`) without overshooting the road boundaries.

---

## Summary & Next Steps

- Published demonstration videos for Nav2 Global Navigation and Follow Line running on macOS without GPU.
- Completed comprehensive profiling of the Monaco 3D circuit, pinpointing the hairpin track departure and physics iteration overheads.
- Implementing physics step tuning to 125 Hz and dual-horizon contour tracking to achieve full autonomous lap completion on Monaco.
