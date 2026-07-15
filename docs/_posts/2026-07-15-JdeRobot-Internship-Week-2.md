---
title: "Internship Progress Week 2 (July 9 ~ July 15)"
date: 2026-07-15 18:45:00 +0530
categories: [Internship 2026, Progress]
tags: [internship, progress, week-2, gazebo, ros2]
published: true
---

This week, my main task was to install, configure, and run the Gazebo simulator using three distinct methods and verify their working with a custom robot simulation.

---

## Method 1: Local Binary Package Installation (ROS 2 Humble)
The first method was installing Gazebo directly on the host system using official ROS 2 Humble binary packages via `apt`. 

To verify its working, I launched Gazebo and Echoed the `/lidar` sensor topic inside the host terminal. The output shows real-time lidar range coordinates successfully communicating over the ROS 2 network.

```bash
sudo apt update
sudo apt install -y ros-humble-ros-gz-sim ros-humble-ros-gz-interfaces ros-humble-ros-gz-bridge
# Verify sensor working
gz topic -e -t /lidar
```

![Method 1 - Package Installation & Lidar Topics]({{ site.baseurl }}/assets/img/week2/method2.png)

---

## Method 2: Local Source Build (colcon workspace)
The second method involved compiling Gazebo directly from source. I created a dedicated colcon workspace (`gazebo_source_ws`), checked out the source repositories, and compiled them manually. 

To verify the working of the source build, I sourced the local overlay and launched the Gazebo GUI, which successfully initialized the simulation environment.

```bash
cd ~/gazebo_source_ws
source install/setup.zsh
ruby $(which gz) sim -g
```

![Method 2 - Source Build Working]({{ site.baseurl }}/assets/img/week2/method3.png)

---

## Method 3: Ubuntu VNC Docker Container (tiryoh/ros2-desktop-vnc)
The third method used a Dockerized Ubuntu container pre-configured with ROS 2 Humble and a VNC server (`tiryoh/ros2-desktop-vnc:humble`). This allows running Gazebo in an isolated containerized Ubuntu environment while visualizing the graphical interface through a web browser on `localhost:6080`.

To verify its working, I accessed the container's Ubuntu desktop, sourced ROS 2, launched Gazebo, and spawned the custom robot inside the simulated arena.

```bash
# Inside Docker Ubuntu Container
source /opt/ros/humble/setup.bash
ign gazebo /home/ubuntu/laser_world.sdf
```

![Method 3 - Ubuntu VNC Docker Container Working]({{ site.baseurl }}/assets/img/week2/ubuntu_vnc.png)
