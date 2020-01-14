---
title: Adding Hokuyo Laser Finder to Turtlebot in Gazebo Simulation
date: 2016-10-01
permalink: /posts/adding-hokuyo-lasers
tags: 
    - ROS
    - hokuyo-laser
    - turtlebot
    - mobile-robot
---
This tutorial presents the steps for adding the Hokuyo laser range finder (LRF) sensor to TurtleBot, and explains how to write URDF description of the Hokuyo LRF, and how to modify the URDF of the Turtlebot robot to account for the addition of the Hokuyo sensor. Before going through this tutorial, it is highly recommended to have a background on [URDF](http://wiki.ros.org/urdf), and for this you may need to look at [URDF Tutorials](http://wiki.ros.org/urdf/Tutorials).

It is better to create a new package to test the code rather than changing the turtlebot package. Firstly, we must create a new workspace by following [Creating a workspace for catkin](http://wiki.ros.org/catkin/Tutorials/create_a_workspace). First change to the source space directory of the catkin workspace:

```shell
# You should have created this in the Creating a Workspace Tutorial
$ cd ~/catkin_ws/src
```
Clone the turtlebot source into this folder:

```shell
git clone https://github.com/turtlebot/turtlebot.git
```
Now, your src dirctory should contain a directory and a file, like this:

```
turtlebot@turtlebot:~/catkin_ws/src$ ls
CMakeLists.txt  turtlebot
```
To use that new enviroment, you will need to source the new setup.bash from it. Source that with the following:

```shell
source ~/catkin_ws/devel/setup.bash
```
Actually, we can add it to ~/.bashrc so that we donot have to source it everytime. 

### Installing Hokuyo Related Packages ###
We must install *hokuyo_node* package as

```shell
sudo apt-get install ros-indigo-hokuyo-node
```
More information on *hokuyo_node* package can be found at [hokuyo_node package](http://wiki.ros.org/hokuyo_node)

### Creating the Hokuyo Description ###

We will be using mesh file **hokuyo.dae** for a visual representation of Hokuyo Laser Range Finder. First we should add the **hokuyo.dae** file into ~/catkin_ws/src/turtlebot/ turtlebot_description/meshes/sensors from /opt/ros/indigo/share/gazebo_plugins/ test/multi_robot_scenario/meshes/laser. We'll start by creating a description of the Hokuyo physical specifications like size and position on our Turtlebot.  Create a new file ~/catkin_ws/src/turtlebot/turtlebot_description/urdf/sensors/hokuyo.urdf.xacro and add following to it.

```xml
<?xml version="1.0"?>
<robot name="sensor_hokuyo" xmlns:xacro="http://ros.org/wiki/xacro">
  <xacro:include filename="$(find turtlebot_description)/urdf/turtlebot_gazebo.urdf.xacro"/>
  <xacro:include filename="$(find turtlebot_description)/urdf/turtlebot_properties.urdf.xacro"/>

  <xacro:macro name="sensor_hokuyo" params="parent">
    <link name="hokuyo_link">
      <collision>
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <geometry>
          <box size="0.1 0.1 0.1"/>
        </geometry>
      </collision>
      <visual>
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <geometry>
          <mesh filename="package://turtlebot_description/meshes/sensors/hokuyo.dae"/>
        </geometry>
      </visual>
      <inertial>
        <mass value="1e-5" />
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <inertia ixx="1e-6" ixy="0" ixz="0" iyy="1e-6" iyz="0" izz="1e-6" />
      </inertial>
    </link>
    <joint name="hokuyo_joint" type="fixed">
      <!--<axis xyz="0 0 1" />-->
      <origin xyz="0.08 0 0.430" rpy="0 0 0"/>
      <parent link="${parent}"/>
      <child link="hokuyo_link"/>
     </joint>
     <!-- Hokuyo sensor for simulation -->
     <turtlebot_sim_laser_range_finder/>
  </xacro:macro>
</robot>

```
 
The first elements of this block are an extra link (hokuyo_link) and joint (hokuyo_joint) added to the URDF file that represents the hokuyo position and orientation realtive to turtlebot. In this xacro description *sensor_hukoyo*, we have passed parameter parent which functions as *parent_link* for hokuyo links and joints. We create a macro named *turtlebot_sim_laser_range_finder* that specifies all of the necessary information for the laser joint and link just before the closing tag.

### Setting Up the Gazebo Plugin ###
Next we must define the Gazebo plugin that gives us the laser range finder  functionality and publishes the laser scans to a ROS message. Add the following code to your turtlebot_gazebo.urdf.xacro after the </xacro:macro> tag closing the Kinect macro.

```xml
<xacro:macro name="turtlebot_sim_laser_range_finder">
  <!-- ULM-30LX Hokuyo Laser Range Finder -->
  <gazebo reference="hokuyo_link">
    <sensor type="gpu_ray" name="head_hokuyo_sensor">
      <pose>0 0 0 0 0 0</pose>
      <visualize>true</visualize>
      <update_rate>40</update_rate>
      <ray>
        <scan>
          <horizontal>
            <samples>1080</samples>
            <resolution>0.25</resolution>
            <min_angle>-2.3561945</min_angle>
            <max_angle>2.3561945</max_angle>
          </horizontal>
        </scan>
        <range>
          <min>0.10</min>
          <max>30.0</max>
          <resolution>0.01</resolution>
        </range>
        <noise>
          <type>gaussian</type>
          <!-- Noise parameters based on published spec for Hokuyo laser
               achieving "+-30mm" accuracy at range < 10m.  A mean of 0.0m and
               stddev of 0.01m will put 99.7% of samples within 0.03m of the true
               reading. -->
          <mean>0.0</mean>
          <stddev>0.01</stddev>
        </noise>
      </ray>
      <plugin name="gazebo_ros_head_hokuyo_controller" filename="libgazebo_ros_gpu_laser.so">
        <topicName>laserscan</topicName>
        <frameName>hokuyo_link</frameName>
      </plugin>
    </sensor>
  </gazebo>
  </xacro:macro>
```
We will review some of the properties

```xml
  <visualize>true</visualize>
  <topicName>LaserScan</topicName>
```
When true, a semi-translucent laser ray is visualized within the scanning zone of the gpu laser. This can be an informative visualization to test if Laser Sensor is working.
topicname define the ROS topic, laserscans will be published, here /laserscan.

This macro will set up the plugin in Gazebo when expanded from the hokuyo.urdf.xacro file.  We now add the Hokuyo urdf to the Turtlebot xacro library file in /turtlebot_description/urdf/turtlebot_library.urdf.xacro.  Add the following line to the end of the file before the </robot> tag.

```xml
 <!--  Hokuyo Laser Sensor-->
 <xacro:include filename="$(find turtlebot_advanced)/urdf/sensors/hokuyo.urdf.xacro"/>
```


### Creating a Description File for the Hokuyo-equipped Bot ###
Now that we've got our macros defined, we want to create a robot description file that actually uses them to specify the robot as having the Hokuyo equipped.  Navigate to the /turtlebot_description/robots folder.  Create a copy of whichever file corresponds to your robot's configuration (ours was create_circles_kinect.urdf.xacro) as < base >_< stack >_hokuyo.urdf.xacro.  As an example that left us with a copy named create_circles_hokuyo.urdf.xacro.  Now edit that file to add the following line.

```xml
<sensor_hokuyo parent="base_link"/>
```

Now we can use this file to launch our Turtlebot with the Hokuyo equipped and working.
To specify you want to use the Hokuyo urdf you just created, run the following in the terminal:

```shell
export TURTLEBOT_3D_SENSOR="hokuyo"
```

Now when roslaunching any of the turtlebot_gazebo launch files, your robot should launch in Gazebo with simulated Hokuyo laser scans. Note that the Kinect will still be equipped and publishing fake laser scans on the /scan topic. To change this, modify the remap section of the launch file to remap the fake laser scans to another topic (/kinect_scan for example).

### Testing in Gazebo Simulation ###

We can launch the gazebo simulation as

```shell
roslaunch turtlebot_gazebo turtlebot_world.launch
```
This should open gazebo as in fig 1. showing Hokuyo Range Finder semi-translucent laser ray in blue color as we have set visualize to true.

{% include image.html url="/images/posts/turtlebot_gazebo.png" description="Fig 1. Turtlebot Gazebo Simulation with Hokuyo Range Finder" %}

To view actual laserscan readings, we must open rviz and then subscribe to /laserscan topic.

```shell
rosrun rviz rviz
```
{% include image.html url="/images/posts/turtlebot_rviz.png" description="Fig 1. Turtlebot Rviz Visualization of Hokuyo Range Finder Data" %}

We can see the location of Hokuyo Range Finder in Rviz model of Turtlebot clearly.



