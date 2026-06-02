Since your setup is already working, here is the complete one-shot sequence from a fresh start up to a working autonomous drone simulation.

Terminal 1 — Start Micro XRCE DDS Agent
MicroXRCEAgent udp4 -p 8888

Expected:

running... port: 8888
session established
Terminal 2 — Start PX4 + Gazebo
source /opt/ros/jazzy/setup.bash

cd ~/PX4-Autopilot

make px4_sitl gz_x500

Wait until Gazebo opens and the drone appears.

You should get:

pxh>
PX4 Parameters (Only Once)

Inside pxh>:

param set COM_RCL_EXCEPT 4
param set COM_ARM_WO_GPS 1
param set NAV_DLL_ACT 0
param set COM_DISARM_LAND 0
param save

Verify:

commander arm

Expected:

Armed by external command

Test:

commander takeoff

Land:

commander land

If this works, PX4 is ready.

Terminal 3 — Build ROS2 Workspace

Only required once.

source /opt/ros/jazzy/setup.bash

mkdir -p ~/px4_ros2_ws/src

cd ~/px4_ros2_ws/src

git clone https://github.com/PX4/px4_msgs.git

git clone https://github.com/PX4/px4_ros_com.git

Build:

cd ~/px4_ros2_ws

colcon build

Source:

source install/setup.bash

Verify:

ros2 interface list | grep px4_msgs

You should see many PX4 message types.

Create Drone Control Package
cd ~/px4_ros2_ws/src

ros2 pkg create --build-type ament_python drone_control

Create scripts folder:

cd drone_control

mkdir scripts

Create control file:

cd scripts

nano offboard_control.py

Paste your working offboard-control code.

Make executable:

chmod +x offboard_control.py
Edit setup.py

Open:

cd ~/px4_ros2_ws/src/drone_control

nano setup.py

Ensure:

entry_points={
    'console_scripts': [
        'offboard_control = drone_control.offboard_control:main',
    ],
},

Place offboard_control.py inside:

~/px4_ros2_ws/src/drone_control/drone_control/

not inside scripts/.

This fixes:

ModuleNotFoundError: No module named 'drone_control.scripts'
Rebuild Workspace
cd ~/px4_ros2_ws

colcon build --packages-select drone_control

Source:

source install/setup.bash
Verify PX4 Topics
ros2 topic list | grep fmu

Expected:

/fmu/out/vehicle_odometry
/fmu/out/vehicle_status_v4
/fmu/out/vehicle_local_position_v1
...
Run Autonomous Drone
Terminal 1
MicroXRCEAgent udp4 -p 8888
Terminal 2
source /opt/ros/jazzy/setup.bash

cd ~/PX4-Autopilot

make px4_sitl gz_x500
Terminal 3
source /opt/ros/jazzy/setup.bash

source ~/px4_ros2_ws/install/setup.bash

ros2 run drone_control offboard_control

Expected:

Arm
↓
Takeoff
↓
Climb to 5 m
↓
Hover
↓
Land
For Construction Monitoring

Start depth-drone model:

pkill -f px4
pkill -f gz

cd ~/PX4-Autopilot

PX4_SIM_MODEL=x500_depth make px4_sitl

Verify sensors:

gz topic -l

You should see:

/world/default/model/x500_depth_0/link/camera_link/sensor/IMX214/image
/world/default/model/x500_depth_0/link/lidar_sensor_link/sensor/lidar/scan
Verify Camera
gz topic -e -t /world/default/model/x500_depth_0/link/camera_link/sensor/IMX214/image
Verify LiDAR
gz topic -e -t /world/default/model/x500_depth_0/link/lidar_sensor_link/sensor/lidar/scan
Final Project Flow
ROS2 Jazzy
      ↓
PX4 SITL
      ↓
Gazebo Harmonic
      ↓
Autonomous Takeoff
      ↓
Waypoint Navigation
      ↓
Camera Capture
      ↓
LiDAR Scan
      ↓
3D Mapping
      ↓
Construction Monitoring
      ↓
Return Home
      ↓
Auto Land

At this point, your foundation is complete. The next development task is waypoint navigation (survey mission), then image capture, then 3D mapping.
