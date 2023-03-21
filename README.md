# SLAM To Point Cloud

Various efforts to create point cloud data from SLAM sessions

## Method 1
### ROS2 bag to ROS1 bag to PCD


git checkout/switch to the KBR AutowareAuto built_for_mapping branch and colcon build it
```sh
cd ~/adehome/AutowareAuto
source install/setup.bash
```
set up workspace
```sh 
cd ~/adehome
mkdir -p slam_ws/src/ros_bags/numpy
cd slam_ws/src/ros_bags
```
enter ade
```sh 
ade --rc .aderc-c1tenth-arm64-jetson start --update --enter
cd ~/AutowareAuto
source install/setup.bash
```
start a ros2 bag recording of OccupancyGrid messages 
by bagging the map topic
```sh 
ros2 bag record /map
```
drop into another ade
```sh
cd ~/adehome/AutowareAuto
ade --rc .aderc-c1tenth-arm64-jetson enter
cd ~/AutowareAuto
source install/setup.bash
export DISPLAY=:0
```
launch the mappping demo. Press RB and drive around like crazy to build the map 
```sh
ros2 launch c1tenth_launch c1tenth_mapping_demo.launch.py with_joy:=True vehicle_interface:=vesc
```
Go back to the original ade session and Ctrl-c to stop recording
you should see a rosbag2_ folder with a .db3 in it
you can kill ade sessions now since the rosbags folder is under ~/adehome

open a new terminal
```sh
cd ~/adehome/slam_ws/src
```
install and build numpy tools
```sh
git clone https://github.com/Box-Robotics/ros2_numpy.git
cd ..
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} -yr
colcon build
cd ros2_numpy
source install/setup.bash
```
install and build the c1t-tools
```sh
cd ~/adehome/slam_ws/src
git clone https://github.com/usdot-fhwa-stol/c1t-tools.git
pip install rosbags
cd c1t-tools/Mapping/map_conversion/src/occupancy_grid_to_pcd2
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro ${ROS_DISTRO} -yr
colcon build --packages-select occupancy_grid_to_pcd2
source install/setup.bash
```
play and loop the ros bag you created from the mapping demo
by specifying the folder where the .db3 lives
```sh
ros2 bag play -l ~/adehome/slam_ws/src/ros_bags
```
while that's looping open another terminal window
```sh
cd ~/adehome/slam_ws/src/ros_bags/numpy
```
record another bag while the first bag is publishing on the numpy topic 
```sh
ros2 bag record /numpy_map/pointcloud 
```
let that run for a while until the first bag has looped back at least once 
Ctrl-c and you should now have a new bag folder under the numpy folder
go back to the first bag and Ctrl-c to stop it

Convert the ros2 bag to a rosbag using rosbags python package and by running
```sh
rosbags-convert ~/adehome/slam_ws/src/ros_bags/numpy/<numpybag name>
```
you should now have  a .bag file

open another terminal
```sh
cd ~
source /opt/ros/noetic/setup.bash
```
start roscore
```sh
roscore
```
open another terminal
```sh
cd ~
source /opt/ros/noetic/setup.bash
rosrun pcl_ros bag_to_pcd ~/adehome/slam_ws/src/ros_bags/numpy/<numpybag name> /numpy_map/pointcloud  <pcd_output_directory>
cd <pcd_output_directory>
```
you should now see 1 or more pcd files 
of the form  0.000000000.pcd

