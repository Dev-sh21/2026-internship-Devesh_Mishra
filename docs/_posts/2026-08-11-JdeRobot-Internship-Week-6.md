---
title: "Internship Progress Week 6 (August 06 ~ August 11)"
date: 2026-08-11 08:00:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-6, gazebo, vnc, macos, docker, x11]
published: true
---

This week, progress focused on evaluating and debugging the local VNC and display streaming architecture for Gazebo simulation components on macOS host and inside the RADI Docker container environment.

---

## 1. Task 1: Native macOS Host VNC & Simulation Testing

### Objective
To manually set up and execute a native local pipeline on macOS consisting of a virtual display server, headless Gazebo server (`gz sim -s`), Gazebo GUI client (`gz sim -g`), and a VNC server (`x11vnc`) to verify if host-level VNC streaming is viable.

### Execution & Observations
Native Gazebo Harmonic components were launched on the macOS host machine. The headless server (`gz sim -s`) initialized successfully and began publishing physics states and transport topics.

However, when launching `x11vnc` to serve the display on port 5902 and attempting connection via a VNC client, the connection hung indefinitely.

### Error Analysis & Root Cause
Inspection of the `x11vnc` terminal logs revealed repeated warnings and framebuffer initialization failures:

![x11vnc Framebuffer Error on macOS Host](/assets/img/posts/x11vnc_mac_error.png)
*Figure 1: Terminal log displaying CGDisplayBaseAddress disabled by OpenGL and fallback geometry w: 0 h: 0.*

In-depth technical analysis identified two primary root causes:
* **Apple API Deprecation:** On modern macOS versions (Sonoma and Sequoia) running on Apple Silicon M-series processors, Apple deprecated the low-level screen capture function `CGDisplayBaseAddress()`. The function returns zero (`w: 0 h: 0`), preventing command-line VNC utilities from accessing the host framebuffer.
* **Qt Graphics Backend:** Native macOS Gazebo builds rely on Apple Cocoa/Metal graphics architecture. The macOS Qt build does not include the X11 `xcb` platform plugin, making it impossible to redirect native Mac GUI rendering into an X11 virtual display (`Xvfb`).

The technical limitations of native macOS host VNC capture were documented in `task1_limitations_report.txt`.

---

## 2. Task 2: Manual Execution Inside RADI Docker Container

### Objective
To replicate the manual setup inside the RADI Docker container (`developer-container`) using a Linux X11 display architecture (`Xvfb`), TurboVNC (`openbox`), and noVNC web proxy.

### Execution & Pipeline Verification
Inside the container, `Xvfb :2`, TurboVNC (`TVNC_WM=openbox`), and the `noVNC` proxy (listening on port 6080) were initialized manually.

To verify whether the display, window manager, and WebSocket proxy pipeline functioned correctly prior to running 3D simulation GUI clients, a standard 2D X11 application (`xclock`) was executed inside the container display environment.

![noVNC Pipeline Verification with 2D Application](/assets/img/posts/vnc_pipeline_verification.png)
*Figure 2: Live streaming verification of container VNC pipeline displaying 2D X11 window on http://localhost:6080/vnc.html.*

### Findings & Analysis
* **VNC & Web Streaming Operational:** As shown in Figure 2, `noVNC` successfully connected and rendered the live 2D window (`xclock`) inside the browser on port 6080. This confirmed that the virtual display, Openbox window manager, TurboVNC server, and noVNC proxy pipeline are fully functional.
* **Gazebo 3D GUI Rendering Behavior:** While 2D applications render and stream without issues, Gazebo Sim 8 GUI relies on OGRE 2 (`ogre2`), which requires hardware OpenGL 4.3+ GPU acceleration (`/dev/dri`). Running x86_64 Docker containers on Apple Silicon Macs via Rosetta 2 CPU software emulation (`softpipe`) prevents OGRE 2 framebuffer initialization, causing the Gazebo GUI rendering thread to wait during startup.
