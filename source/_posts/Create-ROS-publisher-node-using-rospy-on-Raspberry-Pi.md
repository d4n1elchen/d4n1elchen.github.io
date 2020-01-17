---
title: Create ROS publisher node using rospy on Raspberry Pi
date: 2020-01-16 22:10:26
tags: [Robotics, Docker]
categories: Robotics
---

This tutorial will cover some tips about writing publisher using python on the system we setup.

## Dependencies

In this tutorial, we need `rospy` and `std_msg` to create our publisher/subscriber node.

Usually, these two package is included in the basic ROS installation. But if you do not have these two packages install, please refer to [Installing Packages](https://industrial-training-master.readthedocs.io/en/melodic/_source/session1/Installing-Existing-Packages.html) if your ROS run on ubuntu on laptop and {% post_link ROS-melodic-on-Raspbian-Buster %} if your ROS is on RPi for package installation.

## Master Node

Be sure that your master node is up during the following step. Please refer to {% post_link Multiple-machine-setup-for-ROS-melodic %}

## Create a catkin workspace
If you didn't have a catkin workspace, create one first. (Please do not use the one for building ROS distribution, i.e. `ros_catkin_ws` in the previous tutorial. It will cause re-build for every package when you run `catking_make` and waste time)

```sh
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make
```

Source the workspace setup
```
source ~/catkin_ws/devel/setup.bash
```

Or you can add it to your `.bashrc` to save time
```
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
```

## Publisher Node
(Ref [WritingPublisherSubscriber(python)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29))

We'll add a publisher node called talker on RPi, so all the following steps are performed on RPi.

### Create pakcage for talker
```sh
cd ~/catkin_ws/src
catkin_create_pkg talker std_msgs rospy
```

### Create a python script for talker
```sh
roscd talker
mkdir scripts
cd scripts
wget https://raw.github.com/ros/ros_tutorials/kinetic-devel/rospy_tutorials/001_talker_listener/talker.py
chmod +x talker.py
```

You can check the file content
```python
#!/usr/bin/env python
# license removed for brevity
import rospy
from std_msgs.msg import String

def talker():
    pub = rospy.Publisher('chatter', String, queue_size=10)
    rospy.init_node('talker', anonymous=True)
    rate = rospy.Rate(10) # 10hz
    while not rospy.is_shutdown():
        hello_str = "hello world %s" % rospy.get_time()
        rospy.loginfo(hello_str)
        pub.publish(hello_str)
        rate.sleep()

if __name__ == '__main__':
    try:
        talker()
    except rospy.ROSInterruptException:
        pass
```

Read the explaination for the code in [Writing a Simple Publisher and Subscriber (Python)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29)

### Build and start the talker node

Build workspace
```sh
cd ~/catkin_ws
catkin_make
```

Start the node
```sh
rosrun talker talker.py
```

If things work correctly, you'll see
```sh
pi@rpi:~/catkin_ws $ rosrun talker talker.py
[INFO] [1579244074.678847]: hello world 1579244074.68
[INFO] [1579244074.778945]: hello world 1579244074.78
[INFO] [1579244074.878956]: hello world 1579244074.88
[INFO] [1579244074.978917]: hello world 1579244074.98
[INFO] [1579244075.079286]: hello world 1579244075.08
[INFO] [1579244075.179355]: hello world 1579244075.18
[INFO] [1579244075.279409]: hello world 1579244075.28
[INFO] [1579244075.379345]: hello world 1579244075.38
[INFO] [1579244075.479343]: hello world 1579244075.48
...
```

The talker node is sending mseeages every 100ms.

### Check rostopic

After the node has started succefully, we can check topic list on both machies.
```
daniel@ubuntu-vm:~ $ rostopic list
/chatter
/rosout
/rosout_agg
```

`/chatter` is the one talker publishing to.

## Subscriber node

We'll add a subscriber node called listener on laptop, so all the following steps are performed on laptop.

### Create pakcage for listener
```sh
cd ~/catkin_ws/src
catkin_create_pkg listener std_msgs rospy
```

### Create a python script for listener
```sh
roscd listener
mkdir scripts
cd scripts
wget https://raw.github.com/ros/ros_tutorials/kinetic-devel/rospy_tutorials/001_talker_listener/listener.py
chmod +x listener.py
```

You can check the file content
```python
#!/usr/bin/env python
import rospy
from std_msgs.msg import String

def callback(data):
    rospy.loginfo(rospy.get_caller_id() + "I heard %s", data.data)
    
def listener():

    # In ROS, nodes are uniquely named. If two nodes with the same
    # name are launched, the previous one is kicked off. The
    # anonymous=True flag means that rospy will choose a unique
    # name for our 'listener' node so that multiple listeners can
    # run simultaneously.
    rospy.init_node('listener', anonymous=True)

    rospy.Subscriber("chatter", String, callback)

    # spin() simply keeps python from exiting until this node is stopped
    rospy.spin()

if __name__ == '__main__':
    listener()
```

Read the explaination for the code in [Writing a Simple Publisher and Subscriber (Python)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29)

### Build and start the listener node

Build workspace
```sh
cd ~/catkin_ws
catkin_make
```

Start the node
```sh
rosrun listener listener.py
```

If things work correctly, you'll see
```sh
daniel@ubuntu-vm:~ $ rosrun listener listener.py
[INFO] [1579244845.721338]: /listener_2836_1579244845478I heard hello world 1579244846.14
[INFO] [1579244845.820093]: /listener_2836_1579244845478I heard hello world 1579244846.24
[INFO] [1579244845.935054]: /listener_2836_1579244845478I heard hello world 1579244846.34
[INFO] [1579244846.020669]: /listener_2836_1579244845478I heard hello world 1579244846.44
[INFO] [1579244846.133621]: /listener_2836_1579244845478I heard hello world 1579244846.54
[INFO] [1579244846.241096]: /listener_2836_1579244845478I heard hello world 1579244846.64
[INFO] [1579244846.347801]: /listener_2836_1579244845478I heard hello world 1579244846.74
[INFO] [1579244846.424322]: /listener_2836_1579244845478I heard hello world 1579244846.84
[INFO] [1579244846.551422]: /listener_2836_1579244845478I heard hello world 1579244846.94
...
```

The listener node is receiving mseeages every 100ms.

## Node diagram

Run `rqt_graph` on laptop to observe the node diagram

{% asset_img rqt_graph.png %}

## Conclusion

In this tutorial, we leart how to create workspace, create package, and create publisher/subscribe nodes on different machine. In this example, we use rospy to write the node. There's another official example using roscpp. You can check it if interest you [Writing a Simple Publisher and Subscriber (C++)
](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29)

## Link to other relavant post
* {% post_link ROS-melodic-on-Raspbian-Buster %}
* {% post_link Multiple-machine-setup-for-ROS-melodic %}
* {% post_link Create-ROS-publisher-node-using-rospy-on-Raspberry-Pi %} (this post)