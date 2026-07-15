---
title: "Internship Progress Week 2 (July 9 ~ July 15)"
date: 2026-07-15 18:45:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-2, gazebo, ros2]
published: true
---

This week's primary focus was advancing the simulation backend for the Robotics Academy. I was tasked with installing Gazebo from both package and source, implementing a custom robot (`f1` holonomic camera), and successfully integrating it into our web-based ecosystem.

---

## Main Goals for the Week
- Install Gazebo from binary packages (ROS 2 Humble).
- Install Gazebo from source for deeper integration.
- Implement the custom `f1` robot into the simulation.
- Debug and resolve major backend stability issues.

---

## 1. Gazebo Installation and Custom Robot Setup
I successfully set up Gazebo using both package (`ros-humble-ros-gz-sim`) and source (`colcon build`) installations. 
Following the installation, I integrated the custom `f1` robot by creating a new `f1.launch.py` file, linking it to the `simple_circuit.world`, and configuring the camera sensors.

![Gazebo Terminal Setup](/assets/img/week2/media__1784120950328.png)

---

## 2. Resolving Backend Deadlocks
During the integration with the Robotics Application Manager (RAM), I encountered severe, silent process hangs. The `subprocess.Popen` calls were deadlocking due to Python's internal `fork()` implementation crashing when paired with `start_new_session=True`. I successfully diagnosed this and resolved it by migrating the execution flow to use the native Linux `setsid -w` utility.

---

## 3. Fixing Xorg Socket Crashes
Another significant roadblock was the Robotics Academy web interface getting permanently stuck on the "loading" screen. I dug into the backend logs and discovered that Gazebo's headless physics engine was failing to spawn the robot because its dummy `Xorg` display server crashed instantly on startup. 

The root cause was a stale `/tmp/.X11-unix/X0` lock file left behind by previous incomplete shutdowns. I wrote a pre-launch cleanup script to purge these locks before `Xorg` initializes, completely resolving the infinite loading loops.

---

## 4. Container Persistence
Since the web UI dynamically recreates the Docker container every time an exercise launches, any hot-fixes I applied inside the container were instantly wiped. I updated the `docker-compose.yaml` to securely mount the host `RoboticsApplicationManager` source code into the container and explicitly configured the `PYTHONPATH`. This guarantees that all system-level patches are fully persistent.

---

## Screenshots of the Week

Here are a few other images captured during the debugging and setup processes:

![Screenshot 1](/assets/img/week2/media__1784092197249.png)
![Screenshot 2](/assets/img/week2/media__1784094399796.png)
![Screenshot 3](/assets/img/week2/media__1784096843821.png)
![Screenshot 4](/assets/img/week2/media__1784116608671.png)
![Screenshot 5](/assets/img/week2/media__1784111826045.png)

---

## Plan for Next Week
- Finalize sensor data pipelines to the web UI.
- Test performance optimization between CPU software rendering and hardware acceleration.
- Implement additional exercises using the custom `f1` robot.
