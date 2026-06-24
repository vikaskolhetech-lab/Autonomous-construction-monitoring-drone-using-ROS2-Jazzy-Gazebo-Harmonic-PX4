# ROS2 Jazzy PX4 Drone Simulation with Keyboard Control

## Project Overview

This project demonstrates how to control a PX4 drone in Gazebo Harmonic using ROS2 Jazzy and keyboard commands.

The drone is simulated using PX4 SITL (Software In The Loop) and Gazebo Harmonic. A custom ROS2 Python node publishes Offboard Control commands to PX4, allowing the user to move the drone using keyboard inputs.

The project provides a complete workflow for:

* PX4 SITL Simulation
* Gazebo Harmonic Integration
* ROS2 Jazzy Communication
* DDS Communication using Micro XRCE DDS
* Offboard Control
* Keyboard-Based Drone Teleoperation

---

## Technologies Used

* Ubuntu 24.04
* ROS2 Jazzy
* PX4 Autopilot
* Gazebo Harmonic
* Micro XRCE DDS Agent
* Python 3

---

## System Architecture

Keyboard Input

↓

ROS2 Keyboard Node

↓

PX4 Offboard Topics

↓

Micro XRCE DDS Bridge

↓

PX4 SITL

↓

Gazebo Harmonic

↓

Drone Movement

---

## Project Structure

px4_ros_ws/

├── src/

│

├── drone_keyboard/

│   ├── drone_keyboard/

│   │   ├── **init**.py

│   │   └── keyboard_control.py

│   │

│   ├── package.xml

│   ├── setup.py

│   └── setup.cfg

│

├── px4_msgs/

├── px4_ros_com/

│

├── build/

├── install/

└── log/

---

## Problem Faced During Development

Initially the drone was not arming and could not take off.

PX4 displayed:

Preflight Fail: No connection to the GCS

Arming denied: Resolve system health failures first

The issue was caused by PX4 safety checks requiring:

* Ground Control Station connection
* GPS validation
* Data-link availability

For simulation-only development these checks were relaxed using:

param set COM_RCL_EXCEPT 4

param set COM_ARM_WO_GPS 1

param set NAV_DLL_ACT 0

param save

Explanation:

COM_RCL_EXCEPT = 4

Allows Offboard mode without requiring RC or Ground Control Station.

COM_ARM_WO_GPS = 1

Allows arming without GPS lock.

NAV_DLL_ACT = 0

Disables datalink-loss failsafe action.

After applying these parameters:

commander check

returned:

Preflight check: OK

and the drone was able to arm and take off successfully.

---

## Startup Procedure

### Terminal 1

Start DDS Agent

MicroXRCEAgent udp4 -p 8888

Expected Output:

running...
session established

---

### Terminal 2

Start PX4 SITL and Gazebo

cd ~/drone_sim/PX4-Autopilot

make px4_sitl gz_x500

Wait until:

* Gazebo opens
* Drone appears
* PX4 console displays:

pxh>

---

### Verify PX4 Health

Inside PX4:

commander check

Expected:

Preflight check: OK

Test Arm:

commander arm

Test Takeoff:

commander takeoff

Test Landing:

commander land

If all commands work, PX4 is healthy.

---

### Terminal 3

Load ROS2 Workspace

source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

Verify DDS Topics:

ros2 topic list | grep fmu

Expected Topics:

/fmu/in/offboard_control_mode
/fmu/in/trajectory_setpoint
/fmu/in/vehicle_command
/fmu/out/vehicle_status_v4
/fmu/out/vehicle_odometry

---

## Verify Offboard Communication

Inside PX4:

listener vehicle_status

Expected:

accepts_offboard_setpoints: True

This confirms PX4 is receiving ROS2 commands.

---

## Running Keyboard Controller

source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run drone_keyboard keyboard_control

---

## Keyboard Commands

W → Move Forward

S → Move Backward

A → Move Left

D → Move Right

Q → Move Up

E → Move Down

X → Exit

Example:

Key: W

Target -> X:2.5 Y:0.0 Z:-3.0

The target position is updated and continuously published to PX4 through ROS2.

---

## How It Works

The keyboard node continuously publishes:

1. OffboardControlMode

Enables PX4 Offboard mode.

2. TrajectorySetpoint

Contains desired drone position:

X coordinate

Y coordinate

Z coordinate

Yaw angle

3. VehicleCommand

Used to:

Arm the drone

Switch PX4 into Offboard Mode

When a keyboard key is pressed:

W increases X position

S decreases X position

A increases Y position

D decreases Y position

Q decreases Z position (move up)

E increases Z position (move down)

The updated setpoint is then sent to PX4, which moves the drone toward the requested location.

---

## Learning Outcomes

After completing this project, the following concepts are understood:

* ROS2 Node Development
* PX4 Offboard Control
* Gazebo Harmonic Simulation
* DDS Communication
* Drone Teleoperation
* Position Setpoint Control
* Autonomous Systems Architecture
* Robotics Software Integration

---

## Future Improvements

* Real-time keyboard control without pressing Enter
* Camera integration
* LiDAR integration
* Obstacle avoidance
* Waypoint navigation
* Autonomous mission planning
* Construction monitoring use case
* 3D mapping

---

## Author

Vikas Kolhe

STEM Innovation Engineer

Robotics, ROS2, PX4, Embedded Systems and Autonomous Systems Enthusiast
