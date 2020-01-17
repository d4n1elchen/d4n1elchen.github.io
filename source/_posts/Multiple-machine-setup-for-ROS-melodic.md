---
title: Multiple machine setup for ROS melodic
date: 2020-01-16 01:46:41
tags: [Robotics, Docker]
categories: Robotics
---

The setup steps for multi-machine configuration (master node on laptop + raspberry pi node) will be included in this tutorial.

## Multiple Machine

It's very common to run multiple machine on the same ROS network. There're some network settings to be configured before nodes can communicate to each other.

In ROS1, there should be a master node in every ROS network. Every topic or command will send to master node first and then deliver to other nodes that needs. So you should decide which node is your master node. In my configuration, I use laptop running Ubuntu Mate with ROS melodic installed as master node, and the RPi will start a publisher node publishing messages.

### Decide to use hostname or IP

You can explictly specify IP or use hostname for node discovery. If hostname works in your system, I would recommend to use hostname. Otherwise, you'll need to set the environment variables everytime when your network environment is changed. This tutorial will cover both methods.

If you're running your master node in a virtual machine, you may want to use bridge mode for the virtual network adapter so that the VM and RPi are in the same network. I use VM to run Ubuntu. However, I have problem discovering other devices on the same network with hostname in VM and still not find a solution, so I use IP to configure. But I'll use hostname as example mainly.

### Environment Variables

There're two environment variables related to this topic. `ROS_MASTER_URI` and `ROS_HOSTNAME`/`ROS_IP`

#### `ROS_MASTER_URI`

This is to specify the address of master node. You can use hostname or ip for the URI.

```sh
export ROS_MASTER_URI=http://&lthostname or ip of your master node&gt:11311
```

11311 is the default port. If you change the default master node port, please remember to modify it.

This variable should be set to the same value for all the nodes so that they connect to the same master node. For master node itself, this is optional, but it will use as a reference when starting the master node or other nodes on the same machine, so it'd better to set it on the master node machine as well.

#### `ROS_HOSTNAME`/`ROS_IP`

When you're running publisher nodes, this variable is for subscribers to back reference to the publisher node. So this should be set to the hostname/ip of each machines. If this variable isn't properly set or set to some value that other nodes cannot connect through it, you'll encounter problem receving massages.

You can set `ROS_HOSTNAME` or `ROS_IP` based on your environment.

```sh
export ROS_HOSTNAME=&lthostname of the machine&gt
```

#### Set variable in `.bashrc`

You can include these settings in `.bashrc` so that you don't need to set variables everything you start a new terminal session.

```sh
echo "export ROS_MASTER_URI=http://&lthostname of your master node&gt:11311" >> ~/.bashrc
echo "export ROS_HOSTNAME=&lthostname of the machine&gt" >> ~/.bashrc
```

##### Example
On laptop
```sh
echo "export ROS_MASTER_URI=http://ubuntu-vm:11311" >> ~/.bashrc
echo "export ROS_HOSTNAME=ubuntu-vm" >> ~/.bashrc
```
On RPi
```sh
echo "export ROS_MASTER_URI=http://ubuntu-vm:11311" >> ~/.bashrc
echo "export ROS_HOSTNAME=rpi" >> ~/.bashrc
```

### Start a master node

Open a terminal on master node and run
```sh
roscore
```

You'll see some starting messages
```
SUMMARY
========

PARAMETERS
 * /rosdistro: melodic
 * /rosversion: 1.14.3

NODES

auto-starting new master
process[master]: started with pid [2439]
ROS_MASTER_URI=http://ubuntu-vm:11311/

setting /run_id to 74ca6d36-38ee-11ea-8948-08002788615a
process[rosout-1]: started with pid [2450]
started core service [/rosout]
```

You can check `ROS_MASTER_URI` on the 4th last line.

If it starts correctly and the environment variable settings are correct on all the machines, runing `rostopic list` on all the machines will result in the same topic list.
```
daniel@ubuntu-vm:~ $ rostopic list
/rosout
/rosout_agg
```
```
pi@rpi:~ $ rostopic list
/rosout
/rosout_agg
```

## Conclusion

It's not so complicated to setup multiple machine network. Just need to make sure the address you specified can be access by other machine on the same network.

Next, I'll write a publisher node on RPi and a subscriber node on laptop to test our setup.

## Link to other relavant post
* {% post_link ROS-melodic-on-Raspbian-Buster %}
* {% post_link Multiple-machine-setup-for-ROS-melodic %} (this post)
* {% post_link Create-ROS-publisher-node-using-rospy-on-Raspberry-Pi %}