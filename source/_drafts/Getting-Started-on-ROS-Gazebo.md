---
title: Getting Started on ROS Gazebo
date: 2020-01-27 21:24:00
tags: [Robotics, Docker]
categories: Robotics
---

This post includes some getting started practice for ROS + Gazebo stack.

## Environment

The environment I use is Ubuntu Mate 16.04 + ROS Melodic desktop-full installation. With desktop-full installation, Gazebo is included.

## `turtlebot3-gazebo`

### Install packages

```
sudo apt install ros-melodic-turtlebot3 ros-melodic-turtlebot3-gazebo
```

### Launch an empty world with a turtlebot

```shell
export TURTLEBOT3_MODEL=burger
roslaunch turtlebot3_gazebo turtlebot3_empty_world.launch
```

## simple vehicle with follower

{% asset_img vehicle_follower.png %}
Follow [Beginner: Model Editor](http://gazebosim.org/tutorials?cat=guided_b&tut=guided_b3)

## 

