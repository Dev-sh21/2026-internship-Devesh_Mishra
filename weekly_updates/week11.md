# Weekly Update - Week 11

## Porting JdeRobot Robotics Academy to ROS 2 Jazzy and Gazebo Jetty on macOS Apple Silicon

As the ROS ecosystem migrates to newer distributions and simulation engines, ensuring cross-platform support remains critical for educational tools like **JdeRobot's Robotics Academy**. 

Historically, running Robotics Academy on macOS (Apple Silicon ARM64) faced performance bottlenecks, architecture emulation overhead via Rosetta 2, and deadlock issues when pairing older Gazebo versions (Classic and Harmonic) with headless virtual displays (`xvfb`). 

In this work, we explored, built, and validated a native ARM64 stack running **ROS 2 Jazzy Jalisco** and the new **Gazebo Jetty** within Docker on macOS, using the classic **Follow Line** exercise as the primary benchmark.

---

## Demonstration Video

Watch the complete demonstration of the F1 car running autonomously in the Gazebo Jetty simulation on macOS:

<iframe width="100%" height="450" src="https://www.youtube.com/embed/JQ7gxN86NEs" title="JdeRobot Robotics Academy - Follow Line (ROS 2 Jazzy + Gazebo Jetty)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

## The Challenge

Running modern robotics simulations on Apple Silicon presents three distinct engineering hurdles:
1. **CPU & Architecture Constraints:** Avoiding emulation layers by compiling native ARM64 Docker images.
2. **GPU Passthrough Limitations:** macOS Docker Desktop does not offer direct hardware GPU acceleration (`/dev/dri`), requiring highly optimized CPU-based software rasterization.
3. **Ecosystem Upgrades:** Accommodating API breaks, namespace shifts, and ABI incompatibilities introduced between ROS 2 Humble/Iron and ROS 2 Jazzy.

---

## Technical Obstacles & Solutions

### 1. Docker Build Pipeline & Compiler Throttling
Compiling large ROS 2 workspaces on Docker Desktop for Mac often triggers the Linux kernel Out-Of-Memory (OOM) killer on heavy C++ translation units.
- **The Fix:** We constrained `colcon build` to sequential compilation using `MAKEFLAGS="-j1"` and `--executor sequential`.
- **API Breakages:** We dynamically patched legacy C++ includes (`#include <cv_bridge/cv_bridge.h>` to `#include <cv_bridge/cv_bridge.hpp>`), added the `lark` parser dependency required by Jazzy's `rosidl_generator_rs`, and excluded unmaintained industrial robotics packages incompatible with MoveIt 2's refactored time parameterization.

### 2. The NumPy 2.0 vs. `cv_bridge` ABI Clash
When attempting to retrieve camera frames via `HAL.getImage()`, the Python process crashed immediately with:
```text
ImportError: A module that was compiled using NumPy 1.x cannot be run in NumPy 2.4.6 as it may crash.
```
ROS 2 Jazzy's system-installed `cv_bridge` binaries were built against the NumPy 1.x C-API, while standard pip installations in Python 3.12 pulled NumPy 2.x.
- **The Fix:** We downgraded and pinned the dependencies to `numpy<2` (`1.26.4`), `opencv-python<4.10`, and `contourpy<1.2`, then committed the layers permanently into the `jderobot/robotics-academy:test` image.

### 3. Resolving Extreme Simulation Lag (Real Time Factor)
We faced two distinct issues that caused the Gazebo Real Time Factor (RTF) to drop to unplayable levels (as low as `0.001` to `0.204`):

**A. Graphics Rasterizer Limitations:**
- **The Root Cause:** The container originally defaulted to `GALLIUM_DRIVER=softpipe`, a single-threaded reference software rasterizer.
- **The Fix:** We configured the environment to use `GALLIUM_DRIVER=llvmpipe`. This multi-threaded LLVM JIT rasterizer brought the simulation performance up significantly, allowing rendering to run smoothly on Apple Silicon CPUs.

**B. Python CPU Starvation:**
- **The Root Cause:** The user's autonomous control script ran inside an unthrottled `while True:` loop. Because Docker on Mac tightly shares CPU cores, this unconstrained OpenCV processing loop consumed 100% of a core, starving Gazebo's software renderer and causing the RTF to plummet to 0.2.
- **The Fix:** We implemented a strict loop rate limiter (`Frequency.tick(50)`) at the end of the control script. Throttling the logic to 50 iterations per second instantly freed up CPU resources, restoring the simulation RTF to a stable `1.0`.

### 4. Topic Remapping for Gazebo Jetty
In the new Gazebo Jetty integration, topics were bridged under the `/f1/` entity namespace:
- Camera stream: `/f1/camera/image_raw`
- Motor commands: `/f1/cmd_vel`
- Odometry: `/f1/odom`

However, the exercise template code (`HAL.py` and `WebGUI.py`) still queried legacy topic names (`/cam_f1_left/image_raw` and `/cmd_vel`). Because the legacy topic was never published, `HAL.getImage()` entered an infinite `while image is None:` loop, freezing code execution before any velocity commands could be sent. Updating `HAL.py` and `WebGUI.py` to target the active `/f1/` topics resolved the data flow instantly.

### 5. UI Frame Collision & Flicker Elimination
Once video frames started streaming, the web browser UI began rapidly alternating between the raw RGB camera feed and the binary threshold mask.
- **The Root Cause:** A background worker thread in `WebGUI.py` was streaming raw camera frames at 30 FPS, directly competing with the student script's explicit calls to `WebGUI.showImage(debug)`.
- **The Fix:** We disabled the background auto-streaming thread in `WebGUI.py`, giving the user's perception script complete authority over the displayed frame, cleanly showing either the RGB road or the threshold mask based on the script's output.

---

## Conclusion & Future Scope
This milestone confirms that **ROS 2 Jazzy and Gazebo Jetty** can run natively, stably, and interactively on macOS Apple Silicon via Docker without relying on emulation layers. 
