---
title: ROS melodic on Raspbian Buster
date: 2020-01-14 22:49:21
tags: [Robotics, Docker]
categories: Robotics
---

This tutorial is to install ROS melodic on latest (Mon 2020) version of Raspbian.

## Hardware

The hardware I use is Raspberry Pi 4

{% asset_img rpi4.jpg %}

## Install Raspbian Buster

Download the latest Raspbian Buster image from the [official website](https://www.raspberrypi.org/downloads/raspbian/) and follow the [Installation Guide](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) to flash the image into a SD card.

For Windows users, I would recommend using [Etcher](https://www.balena.io/etcher/) to do such work. It's really easy to use and reliable. (I usually use my Windows laptop to flash a SD card, so I don't know what's the best approach on other system. Please refer to the official guide and follow the steps carefully).

### Headless installation
(Ref [Setting up a Raspberry Pi headless](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md))

If you do not have a computer screen for your Pi, you need to setup the wifi beforehand. 

Firstly, plug back your SD card reader with the SD card you just flashed to your laptop.

In Windows, you'll see a disk named "boot" in the navigator of File Explorer. (You'll also see a popup window saying that you need to format a disk. Be careful NOT to format it, otherwise you'll need to flash again)

{% asset_img boot.png %}

Create a txt file with filename "wpa_supplicant.conf".

Use some advanced text editor (eg: Visual Studio Code) to open the file, change the EoF sequence from CRLF to LF and insert following content (replace ssid and psk with your wireless network setup)

```sh
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
 ssid="&ltName of your WiFi&gt"
 psk="&ltPassword for your WiFi&gt"
}
```

Create a blank txt file with filename "ssh" (no extention)

{% asset_img headlessfile.png %}

Eject the SD card from laptop and boot your Pi using modified SD card.

If your setup is correct, you can find your Pi in the local network. (Using IP scanner or whatever you used to find your Pi usually)

## Upgrade system and install common software

Run following commands
```sh
sudo apt update && sudo apt -y dist-upgrade && sudo apt -y upgrade
sudo apt install -y vim curl wget git tmux unzip
```

Reboot your Pi after installation
```sh
sudo reboot
```

## Install ROS melodic
(Ref [Installing ROS Kinetic on the Raspberry Pi](http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Kinetic%20on%20the%20Raspberry%20Pi), please note that the link is for ROS Kinetic)

### Setup ROS Repository

```sh
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" &gt /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
sudo apt-get update && sudo apt-get -y upgrade
```

### Install dependencies

```sh
sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake
```

### Initialize `rosdep`
```sh
sudo rosdep init
rosdep update
```

### Compile ROS melodic from source code
> If you want to try the pre-build install, skip this and go to next section.

#### Create catkin workspace for compiling and installing ROS melodic
```sh
mkdir -p ~/ros_catkin_ws
cd ~/ros_catkin_ws
```

#### Fetch source code by wstool
```sh
rosinstall_generator ros_comm --rosdistro melodic --deps --wet-only --tar &gt melodic-ros_comm-wet.rosinstall
wstool init src melodic-ros_comm-wet.rosinstall
```

> **NOTE**
> The above is fetching "ros_comm" package which includes basic ROS commnication libs without GUI tools such as rqt, rviz. If you want to install GUI tools, change "ros_comm" to "desktop". Or you can find more ROS variant [here](https://www.ros.org/reps/rep-0131.html#variants)

> **NOTE**
> If `wstool init` got interrupted, you can resume downloading process by `wstool update -j4 -t src`

#### Resolve dependencies with rosdep
```sh
rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster
```

#### Build and install ROS!
```sh
sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic
```

#### Add ROS to your bashrc​
```sh
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
```

### Pre-build workspace

> If you already proceed through last section, skip this.

#### Download pre-build worksapce and unzip it
[ros_melodic_raspbian_buster.zip](https://drive.google.com/open?id=16qR3dG7ebRj2Eq4TAh7yL_DznRKaO37A&authuser=daniel@ccns.ncku.edu.tw&usp=drive_fs)

Upload to your RPi and place in home directory.

Unzip it
```sh
cd ~
unzip ros_melodic_raspbian_buster.zip
```

After that, you'll get a folder called `ros_catkin_ws`

#### Resolve dependencies with rosdep
```sh
cd ~/ros_catkin_ws
rosdep install -y --from-paths src --ignore-src --rosdistro melodic -r --os=debian:buster
```

#### Install ROS
```sh
sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic
```

#### Add ROS to your bashrc​
```sh
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
```

## About ROS package management on Raspbian

If you use Ubuntu, apt is used to manage ROS packages. However, this approach does not works on RPi currently. We need to build from the source code of the package you want to install.

The process to add new package is as follow

### Generate rosinstall file with rosinstall_generator

You may see similar command above somewhere during the installation. This is to generate a list of package source link.
```sh
cd ~/ros_catkin_ws
rosinstall_generator &ltpackage names seperated by space&gt --rosdistro melodic --deps --wet-only --tar &gt &ltcustom file name&gt.rosinstall
```

Example (to install [sensor_msgs](http://wiki.ros.org/sensor_msgs))
```sh
rosinstall_generator sensor_msgs --rosdistro melodic --deps --wet-only --tar &gt melodic-sensor_msgs.rosinstall
```

### Merge and update the workspace
```sh
wstool merge -t src &ltrosinstall file you just generated&gt.rosinstall
wstool update -t src
```

This will fetch all the source code listed in rosinstall files

Example
```sh
wstool merge -t src melodic-sensor_msgs.rosinstall
wstool update -t src
```

### Resolve dependency by rosdep
```
rosdep install --from-paths src --ignore-src --rosdistro melodic -y -r --os=debian:buster
```

### Rebuild the workspace
```sh
sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/melodic
```

## Conclusion
It's a bit tedious installing ROS and managing pakcage on RPi and the build time is long. So I wrote this post for memo and future reference.

Next post I'll talk about how to setup the multi-machine environment and then create a publisher node on RPi.

## Link to other relavant post
* {% post_link ROS-melodic-on-Raspbian-Buster %} (this post)
* {% post_link Multiple-machine-setup-for-ROS-melodic %}
* {% post_link Create-ROS-publisher-node-using-rospy-on-Raspberry-Pi %}