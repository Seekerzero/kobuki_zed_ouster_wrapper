# Kobuki ZED ORB SLAM3 Integration

This ROS package implements the ORB SLAM3 Integration using zed camera on Turtlebot2 Kobuki Model, where the package also includes ground truth recording function using ouster OS1 64 lidar with HDL graph slam. The embed board was tested on jetson xavier NX on [Jetpack 5.1](https://developer.nvidia.com/embedded/jetpack-sdk-51). 



### PC and Jetson Common Setup:

Install ROS Noetic

Setup workspace:

```bash
$ mkdir ~/catkin_ws/src -p
$ cd ~/catkin_ws
$ catkin_make
$ echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
$ source ~/.bashrc
$ cd src
```

[Turtlebot2 on noetic](https://github.com/hanruihua/Turtlebot2_on_Noetic)

```bash
$ sudo apt-get install ros-noetic-sophus ros-noetic-joy libusb-dev libftdi-dev ros-noetic-base-local-planner ros-noetic-move-base-msgs pyqt5-dev-tools
$ git clone https://github.com/hanruihua/Turtlebot2_on_Noetic.git
$ cd ~/catkin_ws
$ rosdep install --from-paths src --ignore-src -r -y
$ catkin_make -DCMAKE_BUILD_TYPE=Release
$ source ~/.bahsrc
```

Clone this wrapper:

```bash
$ cd ~/catkin_ws/src
$ git clone https://github.com/Seekerzero/kobuki_zed_orb_slam3.git
$ cd ~/catkin_ws
$ catkin_make -DCMAKE_BUILD_TYPE=Release
$ source ~/.bashrc
```





### Jetson Setup:

Install [ZED SDK 3.8  for L4T 35.1 (Jetpack 5.0)](https://www.stereolabs.com/developers/release/)

Install [ZED ROS wrapper](https://github.com/stereolabs/zed-ros-wrapper.git):

```bash
$ cd catkin_ws/src
$ git clone --recursive https://github.com/stereolabs/zed-ros-wrapper.git
$ cd catkin_ws
$ rosdep install --from-paths src --ignore-src -r -y
$ catkin_make -DCMAKE_BUILD_TYPE=Release
$ source ~/.bashrc
```

Setup udev rule for connection with kobuki:

```bash
$ rosrun kobuki_ftdi create_udev_rules
```





### Laptop Setup:

Setup [Ouster ROS Wrapper](https://github.com/ouster-lidar/ouster-ros):

```bash
$ sudo apt install -y ros-noetic-pcl-ros ros-noetic-rviz
$ sudo apt install -y build-essential libeigen3-dev libjsoncpp-dev libspdlog-dev libcurl4-openssl-dev cmake
$ cd ~/catkin_ws/src
$ git clone --recurse-submodules https://github.com/ouster-lidar/ouster-ros.git
$ cd ~/catkin_ws
$ catkin_make --cmake-args -DCMAKE_BUILD_TYPE=Release
$ source ~/.bashrc
```

Setup the [network with OS1 lidar](https://static.ouster.dev/sensor-docs/image_route1/image_route2/networking_guide/networking_guide.html):

Get the hostname using:

```bash
avahi-browse -lrt _roger._tcp
```

and change the **sensor_hostname** in **lidar_sensor.launch** to the lidar hostname.



Setup the [HDL graph SLAM](https://github.com/koide3/hdl_graph_slam) and [ethz_msf imu plugin](https://github.com/ethz-asl/ethzasl_msf.git):

```bash
$ cd ~/catkin_ws/src
$ sudo apt-get install ros-noetic-geodesy ros-noetic-pcl-ros ros-noetic-nmea-msgs ros-noetic-libg2o libgoogle-glog-dev
$ git clone https://github.com/koide3/ndt_omp.git
$ git clone https://github.com/SMRT-AIST/fast_gicp.git --recursive
$ git clone https://github.com/koide3/hdl_graph_slam
$ git clone https://github.com/ethz-asl/ethzasl_msf.git
$ git clone https://github.com/ethz-asl/glog_catkin.git
$ git clone https://github.com/catkin/catkin_simple.git
$ rosdep install --from-paths src --ignore-src -r -y
$ catkin_make --cmake-args -DCMAKE_BUILD_TYPE=Release
$ source ~/.bashrc
```



### Setup the ROS Network:

Follow the instruction on section 1.1.6 of [Turtlebot3 PC setup](https://emanual.robotis.com/docs/en/platform/turtlebot3/quick-start/#pc-setup) to setup the ROS connection between laptop and Jetson.



### Record data:

On the laptop:

```bash
$roscore
```

SSH to jetson board:

```bash
$ ssh jetson_name@ip
```

Bring up the kobuki:

```bash
$ roslaunch turtlebot_bringup minimal.launch
```

On a new ssh terminal, launch the zed node and the robot state publisher:

```bash
$ roslaunch kobuki_zed_orb_slam3 view_model.launch
```

On a new ssh terminal, rosbag the topics:

```bash
$ rosbag record /zed_node/left_raw/image_raw_color /zed_node/right_raw/image_raw_color /tf /tf_static /mobile_base/sensors/imu_data_raw /mobile_base/sensors/imu_data /joint_states /zed_node/depth/camera_info /zed_node/left_raw/camera_info /zed_node/right_raw/camera_info /zed_node/parameter_descriptions
```



On the laptop:

```bash
$ roslaunch kobuki_zed_orb_slam3 lidar_sensor.launch
```

rosbag the topics:

```bash
$ rosbag record /ouster/points /ouster/imu /ouster/range_image
```

control the kobuki with:

```bash
$ roslaunch turtlebot_teleop keyboard_teleop.launch
```



### Processing Data

Install [rosbag-merge](https://pypi.org/project/rosbag-merge/):

```bash
$ pip3 install rosbag-merge
```

get into the folder where contains two bag files:

```bash
$ rosbag-merge --outbag_name out.bag
```

play the merge bag:

```bash
$ rosbag play --clock out.bag
```

pause with **space**

launch the hdl slam node:

```bash
$ roslaunch kobuki_zed_orb_slam3 hdl_graph_slam_imu.launch
```

change the fixed frame to map and play the rosbag to view the hdl graph slam ground truth.





### ORB SLAM3 Installation

Follow the instruction on [orb_slam3_ros](https://github.com/thien94/orb_slam3_ros) tosetup the ORB SLAM3 on ROS. 