# SLAM WITH L515



## Installation
1. ### Ubuntu and ROS
   1.1 - [Ubuntu 64-bit 18.04 installation](https://releases.ubuntu.com/18.04/)

   1.2 - [ROS melodic installation](http://wiki.ros.org/ROS/Installation)

2. ### Ceres Solver
    2.1 - [Ceres Solver installation](http://ceres-solver.org/installation.html)
   
   **or** you can type the following commands of step 2.2:

    2.2 -  Install Eigen3 and Ceres Solver directly from terminal
> #### Eigen part:
```sh
git clone -b 3.3 https://gitlab.com/libeigen/eigen.git eigen
```
```sh
cd /eigen/
```
```sh
mkdir build
```
```sh
sudo cmake .. && sudo make -j2 && sudo make install
```
> #### Ceres Solver part:
   
```sh
git clone https://ceres-solver.googlesource.com/ceres-solver ceres-solver
```
```sh
sudo apt install -y cmake libgoogle-glog-dev libgflags-dev libatlas-base-dev libsuitesparse-dev
```
```sh
cd ceres-solver
```
```sh
sudo cmake ../ceres-solver # i am not sure if this is the exact command, if you have a problem read the error message and debug
```
```sh
sudo make -j2 && sudo make install
```

3. ### gtsam 
```sh
git clone https://github.com/borglab/gtsam.git gtsam
```
```sh
cd gtsam
```
```sh
mkdir build
```
```sh
sudo cmake -DGTSAM_BUILD_WITH_MARCH_NATIVE=OFF ..
```
```sh
sudo make -j2 && sudo make install
```

4. ### PCL 1.8.1
   
```sh
wget https://github.com/PointCloudLibrary/pcl/archive/pcl-1.8.1.tar.gz && tar zvfx pcl-1.8.1.tar.gz
```
```sh
cd pcl-1.8.1
```
```sh
mkdir build
```
```sh
sudo cmake .. && sudo  make -j2  && sudo make install
```
```sh
sudo apt install -y ros-melodic-pcl-conversions ros-melodic-pcl-ros
```

5. ### hector trajectory server
```sh
sudo apt install -y ros-melodic-hector-trajectory-server
```
6. ### Sensor L515 Installation
   6.1 [librealsense installation](https://github.com/IntelRealSense/librealsense/blob/master/doc/installation.md)

   **or** you can type:
   ```sh
   sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
   ```
  **if you have a problem:**

  ----------------------------------

   >Do the following changes at your linux system in order to work:

   >cd /etc/apt

   >sudo nano sources.list

   and then do the following changes:


   >  Add the server to the list of repositories:
   
   **Ubuntu 16 LTS:**

   >sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u 

   >sudo add-apt-repository "deb http://librealsense.intel.com/Debian/apt-repo xenial main" -u

   **Ubuntu 18 LTS:**

   >sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo bionic main" -u

   >sudo add-apt-repository "deb http://librealsense.intel.com/Debian/apt-repo bionic main" -u

   **Ubuntu 20 LTS:**

   >sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo focal main" -u

   >sudo add-apt-repository "deb http://librealsense.intel.com/Debian/apt-repo focal main" -u


   ***i replaced the first sudo with the second in each case inside the /etc/apt/sources.list file in my linux system.***

   **this solution came from here: ***[issue link](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md)*****

   ---------------------------------
   **Now let's **continue** librealsense installation:**
```sh
git clone https://github.com/IntelRealSense/librealsense.git
```
```sh
sudo apt-get install git libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev
```
```sh
sudo apt-get install libglfw3-dev
```
```sh
./scripts/setup_udev_rules.sh
```
```sh
./scripts/patch-realsense-ubuntu-lts.sh
```
```sh
echo 'hid_sensor_custom' | sudo tee -a /etc/modules
```



6.2 realsense-ros installation
```sh 
cd ~/catkin_ws/src
```
```sh
git clone https://github.com/IntelRealSense/realsense-ros.git
```
```sh
cd ..
```
```sh
catkin_make
```
7. ### Build ssl-slam2
```sh
cd ~/catkin_ws/src
```
```sh
git clone https://github.com/wh200720041/ssl_slam2.git
```
```sh
cd ..
```
```sh
catkin_make
```
```
source ~/catkin_ws/devel/setup.bash
```
8. ### Update l515 firmware
   
   [l515 firmware update procedure](https://dev.intelrealsense.com/docs/firmware-update-tool)
-------------
## Record data

> open two tabs in terminal:

 - 1st tab:
```sh
roslaunch realsense2_camera rs_camera.launch filters:=pointcloud
```

 - 2nd tab:
```sh
rosbag record /camera/color/camera_info /camera/color/image_raw /camera/depth/camera_info /camera/depth/color/points /camera/depth/image_rect_raw /camera/extrinsics/depth_to_color /tf /tf_static -O name-of-my-file.bag
```
----
## SLAM data, Create 3D model and Save to your pc
- first, navigate to 
```sh
cd ~/catkin_ws/src/ssl_slam2/launch/
```
- next, type

```sh
 sudo nano ssl_slam2_mapping.launch
 ```

- and **change** in this line (3rd):
> node name="bag" pkg="rosbag" type="play" args="--clock -r 0.5 $(env HOME)/**name-of-my-file.bag**" /

   >the name of the file in the end, just like i did, in order the algorithm to be able to read your pre-recorder .bag file and perform the ssl_slam2 algorithm on it.

### Now we are **ready to slam and save**:

> open three tabs in terminal:

 - 1st tab:
```sh
roscore
```

 - 2nd tab:
```sh
roslaunch ssl_slam2 ssl_slam2_mapping.launch
```

 - 3rd tab:
```sh
rosrun pcl_ros pointcloud_to_pcd input:=/map
```

Now, at your home folder, you will find the .pcd files of your 3D model.

Put them inside the folder, and using a stable version of the [cloudcompare](http://www.danielgm.net/cc/release/) open them and save them as .E57 pointcloud format if you want.

That's all folks. Good luck.
In the future i will try to improve this guide.

---------

![slam_house](https://user-images.githubusercontent.com/70270581/113325472-2b06c580-9321-11eb-904c-ce66fac14d40.png)


> ### for more info, here is the link of the official ssl_slam2 repo: 
> - https://github.com/wh200720041/ssl_slam2 

> ### and some useful links from issues:
> - https://github.com/wh200720041/ssl_slam2/issues/3
> - https://github.com/IntelRealSense/realsense-ros/issues/1076#issuecomment-586702026 
> - https://github.com/wh200720041/ssl_slam/issues/2

