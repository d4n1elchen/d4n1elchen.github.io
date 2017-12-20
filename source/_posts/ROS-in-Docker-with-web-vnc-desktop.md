---
title: ROS in Docker with web vnc desktop
date: 2017-12-19 20:59:02
tags: [Robotics, Docker]
categories: Robotics
---

Recently I'm trying to learn ROS. But unfortunately ROS haven't yet support Ubuntu 17.10. So I can only run ROS in docker container.

However, by default there isn't a x window if you use the official `ros` image. You can't use tools with GUI such as `rviz` or `gazebo`.

I found following docker image which also mentioned in the ROS official wiki that provide LXDE and HTML5 VNC interface.

[fcwu/docker-ubuntu-vnc-desktop](https://github.com/fcwu/docker-ubuntu-vnc-desktop)

So I integral these two images together and push it to docker hub.

## Run the image

```
docker run -it -p 6080:80 d4n1el/ros-vnc-desktop
```

My image will run `roscore` automatically at startup.

Now you can browse http://127.0.0.1:6080/ to access the desktop.

Open `lxterminal` from menu and install turtlesim for test.

```
# apt update && apt install ros-kinetic-turtlesim
```

Run turtlesim
```
# rosrun turtlesim turtlesim_node
```

If you see a turtle showing up on your screen, then you're success.

{% asset_img screenshot.png %}
