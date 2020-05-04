<p align="center">
<img src="https://user-images.githubusercontent.com/28100951/79417266-e5be7c00-7f87-11ea-91a1-a6724834effe.png" width="600">
</p>

------------

<a id="top"></a>
### Contents
1. [Description](#1.0)
2. [Required packages - Kinetic Version](#2.0)
3. [Run GGCNN in Gazebo and RVIZ](#3.0)
4. [Connecting with real UR5](#4.0)

------------

<a name="1.0"></a>
### 1.0 - Description

This repository was created in order to develop an improved version of the [GGCNN]((https://github.com/dougsm/ggcnn_kinova_grasping)) Grasp Method created by Doug Morrison (2018).

> **_NOTE:_**  This package should be placed into your src folder. Please open an issue if you find any problem related to this package.

<a name="2.0"></a>
### 2.0 - Required packages - Kinetic Version

- [Realsense Gazebo Plugin](https://github.com/pal-robotics/realsense_gazebo_plugin)
- [Realsense-ros](https://github.com/IntelRealSense/realsense-ros) Release version 2.2.11
- [Librealsense](https://github.com/IntelRealSense/librealsense) Release version 2.31.0 - Install from source
- [Moveit Kinetic](https://moveit.ros.org/install/)
- [Moveit Python](https://github.com/mikeferguson/moveit_python)
- [Robotiq Gripper](https://github.com/crigroup/robotiq)
- [Universal Robot](https://github.com/ros-industrial/universal_robot)
- [ur_modern_driver](https://github.com/ros-industrial/ur_modern_driver)
- [Gluoncv](https://github.com/dmlc/gluon-cv)
- [Opencv](https://github.com/opencv/opencv)
- [Mxnet](https://mxnet.apache.org/) Install Mxnet for your CUDA version.

#### Easy install

In order to install all the required packages easily, create a new catkin workspace
```bash
mkdir -p ~/catkin_ws_new/src
```

Clone this repository into the src folder
```bash
cd ~/catkin_ws_new/src
git clone https://github.com/lar-deeufba/real_time_grasp
```

Run the install.sh file
```bash
cd ~/catkin_ws_new/src/real_time_grasp/install
sudo chmod +x ./install.sh
./install.sh
```

#### This repository also need the SSD512 implementation created by [czrcbl](https://github.com/czrcbl). Please follow the next procedures provided by him.

Wrapper for some object detection models trained on parts produced on a 3D printer.

Install bboxes before, you can install directly from `github`:
```bash
pip install git+https://github.com/czrcbl/bboxes
```

Or you can clone the repository and install on editable mode:
```bash
git clone https://github.com/czrcbl/bboxes
cd bboxes
git install -e .
```

Download the [model2.params](https://drive.google.com/file/d/1NamkTraRxDBBKDzN5p5D1lCBShqOHp36/view?usp=sharing) in the following link and move it to the `detection_pkg` folder.

<a name="3.0"></a>
### 3.0 - Run GGCNN and SSD512 in Gazebo and RVIZ

Launch Gazebo first:
obs: The robot may not start correctly due to a hack method used to set initial joint positions in gazebo as mentioned in this [issue](https://github.com/ros-simulation/gazebo_ros_pkgs/issues/93#). If it happens, try to restart gazebo.
`bico` is a parameter that loads a 3D part in order to test the proposed method. We are not allowed to share this stl file since it is part of a private project. Therefore you will not be able to run the SSD512 net with the pre-trained `bico` part.
```bash
roslaunch real_time_grasp gazebo_ur5.launch bico:=true
```

Run this node and let the UR5 robot move to the front of the 3D printer. Then launch the next node (SSD512 node). 
The arm will only perform the grasp after the GGCNN node is running.
```bash
rosrun real_time_grasp command_GGCNN_ur5.py --gazebo
```

Start the SSD512 detection node. This node is responsible for detecting the object to be filtered.
```bash
roslaunch real_time_grasp detection.launch
```

Launch the node responsible for filtering the point cloud of the object detected by SSD512 network. The pointcloud and the depth image generated by this node will then be applied to the GGCNN node in order to identify the best possible grasp.
```bash
roslaunch real_time_grasp filter.launch
```

Launch RVIZ if you want to see the frame (object_detected) corresponding to the object detected by GGCNN and the point cloud.
In order to see the point cloud, please add pointcloud2 into the RVIZ and select the correct topic:
```bash
roslaunch real_time_grasp rviz_ur5.launch
```

Run the GGCNN. This node will publish a frame corresponding to the object detected by the GGCNN.
```bash
rosrun real_time_grasp SSD512_GGCNN.py
```

Running the following command will speed up the Gazebo simulation a little bit :)
```bash
rosrun real_time_grasp change_gazebo_properties.py
```

You might want to see the grasp or any other image. In order to do that, you can use the rqt_image_view.
```bash
rosrun rqt_image_view
```

<a name="4.0"></a>
### 4.0 - Connecting with real UR5

Use the following command in order to connect with real UR5.
If you are using velocity control, do not use bring_up. Use ur5_ros_control instead.

```
roslaunch real_time_grasp ur5_ros_control.launch robot_ip:=192.168.131.13
```

Launch the real Intel Realsense D435
```
roslaunch real_time_grasp rs_d435_camera.launch
```

Launch the gripper control node
```
rosrun robotiq_2f_gripper_control Robotiq2FGripperRtuNode.py /dev/ttyUSB0
```

Launch the ggcnn node
```
rosrun real_time_grasp run_ggcnn_ur5.py --real
```

Launch the main node of the Intel Realsense D435
```
rosrun real_time_grasp command_GGCNN_ur5.py
```

If you want to visualize the depth or point cloud, you can launch RVIZ
```
roslaunch real_time_grasp rviz_ur5.launch
```
