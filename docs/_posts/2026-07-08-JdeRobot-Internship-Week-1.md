---
title: "Internship Progress Week 1 (July 2 ~ July 8)"
date: 2026-07-08 10:00:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-1]
published: true
---

This week marks the beginning of my JdeRobot 2026 Internship. The primary focus of this week has been setting up the local workspace, debugging the Gazebo simulator display pipeline on my macOS computer, optimizing local system storage, and aligning the local repository branch.

---

## Main Goals for the Week
- Resolve local storage limitations affecting the container tools.
- Set up and test display forwarding from the container to the host Mac.
- Sync local repository files with the upstream development branch.
- Prepare and build the local ROS 2 container image (In Progress).

---

## 1. Storage Optimization
I performed a system cleaning on the Mac to free up disk space. By clearing unnecessary cache files, I successfully restored sufficient storage capacity, allowing Docker and container actions to run reliably.

---

## 2. Display Forwarding Diagnostics
To understand why the simulation layout failed to render, I set up XQuartz on macOS to handle graphical display forwarding. I verified the connection bridge by launching test interface windows from the container onto the Mac screen, verifying that the display socket bridge is functional.

---

## 3. Repository Alignment and React Compilation
I synchronized the local repository files to match the latest development release on GitHub to ensure consistent API communication. Additionally, I resolved a webpack conflict where duplicate package instances caused errors in the browser, and compiled the updated front-end assets. The Follow Line exercise is now loading successfully.

![Follow Line Working](/assets/img/posts/follow_line_working.png)

---

## 4. Local Container Customization (In Progress)
I am currently working on configuring and compiling the local ROS 2 Humble application docker image. This build process is heavy and takes time to compile all packages, which will continue into next week.

---

## Plan for Next Week
- Complete the local container build.
- Test simulation rendering stability.
- Run and verify template logic execution.
