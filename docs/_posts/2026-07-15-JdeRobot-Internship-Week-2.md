---
title: "Internship Progress Week 2 (July 9 ~ July 15)"
date: 2026-07-15 18:45:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-2, gazebo, ros2]
published: true
---

This week, my main task was to install and configure the Gazebo simulator using three different methods, and to verify the setup by running a custom robot simulation.

---

## Method 1: Install Gazebo from Binary Packages
For the first setup, I installed Gazebo via the standard package manager using official ROS 2 Humble packages. This is the simplest and most stable method for quick deployments.

```bash
sudo apt update
sudo apt install -y ros-humble-ros-gz-sim ros-humble-ros-gz-interfaces ros-humble-ros-gz-bridge
```

![Method 1 - Package Installation](/assets/img/week2/method1.png)

---

## Method 2: Install Gazebo from Source
For the second setup, I compiled Gazebo directly from source. I set up a colcon workspace, imported the repositories, and built them manually. This method is crucial when custom modifications to the simulator are required.

```bash
colcon build --merge-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```

![Method 2 - Source Build](/assets/img/week2/method2.png)

---

## Method 3: Running Simulation on Custom Robot
Finally, I tested both installation environments by running the simulation using a custom robot package (`laser_robot`). I launched the robot inside a custom arena with obstacle physics enabled and verified topic publisher/subscriber communication in ROS 2.

```bash
source /opt/ros/humble/setup.bash
ign gazebo /home/ubuntu/laser_world.sdf
```

![Method 3 - Running Robot Simulation](/assets/img/week2/method3.png)
