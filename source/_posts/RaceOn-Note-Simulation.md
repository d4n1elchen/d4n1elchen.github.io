---
title: RaceOn Note - Simulation
tags:
  - robotics
  - RaceOn
categories: robotics
date: 2020-03-10 00:11:52
---


It is quite common to do experiment in a simulation environment when doing robotics research because you don't want to break your hardware or injure people during tuning your algorithm.

I built a simulation environment for RaceOn competition. This post will provide instructions for setting up the environment and starting the RaceOn simulation.

## Simulation software - Gazebo

Gazebo is a free simulation software widely used in robotics research. It provide lots of built-in plugins for different type of sensors, actuators, and basic physics engine. Gazebo also works well with ROS. There's a lot of ROS packages that help you control the model and retrieve information of your simulation environment through ROS framework.

## Environment

The environment I use is Ubuntu Mate 18.04 + ROS Melodic desktop-full installation. With desktop-full installation, Gazebo is included.

If you don't have Ubuntu and ROS installed, you can try the Virtual Box image I built.

[Download the Image](https://drive.google.com/a/usc.edu/file/d/1FRY4Ysx_p8Ss7IdHxm0CgqmswSTQfGZk/view?usp=sharing)

## RACECAR

[RACECAR](https://mit-racecar.github.io/) is an opensource project from MIT, designed for robotics research and teaching. The RACECAR itself is the name of a 1/10-scale mini race car with some common sensors such as LiDar, Camera, Depth Camera, and IMU. The mechanical configuration of that race car is quite similar to the one RaceOn use. Moreover, they also build a simulation environment based on Gazebo. Most of my works are based on their code.

## `raceon-simulation`

Let's get start to setup a catkin workspace for raceon simulation.

The following are copied from [`d4n1elchen/raceon_simulation`](https://github.com/d4n1elchen/raceon_simulation). If there's inconsistency between my repo and this post, please refer to the one in repo as the latest.

### Make a workspace for simulation

```shell
mkdir -p ~/raceon_sim_ws/src
cd ~/raceon_sim_ws
catkin_make
```

### Source the workspace 

```
source ~/raceon_sim_ws/devel/setup.bash
```

(Optional) Add it to bashrc
```
echo "source ~/raceon_sim_ws/devel/setup.bash" >> ~/.bashrc
```

### Set ROS_PYTHON_VERSION to 3
```
export ROS_PYTHON_VERSION=3
```

(Optional) Add it to bashrc
```
echo "export ROS_PYTHON_VERSION=3" >> ~/.bashrc
```

### Install dependencies

```shell
sudo apt-get install ros-melodic-ros-control ros-melodic-gazebo-ros-control ros-melodic-ros-controllers python3-opencv ros-melodic-ackermann-msgs
pip3 install pynput
```

### Clone dependencies

```shell
cd ~/raceon_sim_ws/src
git clone https://github.com/wjwwood/serial.git
git clone https://github.com/ros-drivers/ackermann_msgs.git
git clone https://github.com/mit-racecar/racecar.git
git clone https://github.com/mit-racecar/vesc.git
git clone https://github.com/d4n1elchen/raceon.git
git clone https://github.com/d4n1elchen/raceon_simulation.git
git clone https://github.com/d4n1elchen/raceon_visualizer.git
git clone https://github.com/d4n1elchen/racecar_gazebo.git
```

### Build

```
cd ~/raceon_sim_ws
catkin_make
```

### Run

```
roslaunch raceon_simulation raceon_simulation.launch speed:=180 kp:=10
```

If you got python module not found error, install missing packages using `pip3 install <package_name>`.

## Video demo

{% youtuber video U3WNZOiOz7Q %}
{% endyoutuber %}

## Summary

To tune the simulation parameter to fit the real world well is not an easy work. For now, the car can replicate approximately 80% behavior of a real car. I use the same position estimation and control node running on the real car to control the simulated car. These nodes are from `raceon` repository, which is the same as the one in each team's raspberry pi. You can try to adjust the parameters in `raceon_simulation/launch/raceon_simulation.launch` to see how the car react to those parameters, and verify the result by running the same parameters on a real car.

This post is just for setting up environment. I'll write another post to explain how to modify the parameters or how I built this environment in detail ... if I have time. XD

If you feel this is cool or find it helpful for your work, please don't hesitate to give me a star and follow my GitHub profile!

Repository: https://github.com/d4n1elchen/raceon_simulation
GitHub Profile: https://github.com/d4n1elchen