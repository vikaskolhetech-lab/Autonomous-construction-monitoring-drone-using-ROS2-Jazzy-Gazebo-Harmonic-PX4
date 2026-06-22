# ROS2 Jazzy PX4 Keyboard Controlled Drone Simulation

## Project Overview

This project demonstrates how to control a PX4 drone in Gazebo Harmonic using keyboard commands through ROS2 Jazzy Offboard Control.

The drone runs inside the PX4 SITL simulator and receives movement commands from a custom ROS2 node. Users can move the drone forward, backward, left, right, up, and down using keyboard inputs.

The communication chain is:

Keyboard → ROS2 Node → PX4 Offboard Control → Gazebo Harmonic → Drone Movement

---

# Technologies Used

* ROS2 Jazzy
* PX4 Autopilot
* Gazebo Harmonic
* Micro XRCE DDS Agent
* Python
* Ubuntu 24.04

---

# System Architecture

Keyboard Input
↓
ROS2 Node
↓
Offboard Control Mode
↓
PX4 SITL
↓
Gazebo Harmonic
↓
Drone Movement

---

# Workspace Structure

px4_ros_ws/

├── src/

│   ├── drone_keyboard/

│   │   ├── drone_keyboard/

│   │   │   ├── **init**.py

│   │   │   └── keyboard_control.py

│   │   ├── package.xml

│   │   ├── setup.py

│   │   └── setup.cfg

│

│   ├── px4_msgs/

│   └── px4_ros_com/

│

├── build/

├── install/

└── log/

---

# Step 1: Stop Existing Processes

Before starting a new simulation:

```bash
pkill -f px4
pkill -f gz
pkill -f MicroXRCEAgent
```

---

# Step 2: Start DDS Agent

Terminal 1

```bash
MicroXRCEAgent udp4 -p 8888
```

Expected:

```text
running...
session established
```

---

# Step 3: Start PX4 SITL and Gazebo

Terminal 2

```bash
cd ~/drone_sim/PX4-Autopilot

make px4_sitl gz_x500
```

Wait for:

* Gazebo opens
* Drone appears
* PX4 console shows:

```text
pxh>
```

---

# Step 4: Verify PX4 Health

Inside PX4 console:

```bash
commander check
```

Expected:

```text
Preflight check: OK
```

Test arming:

```bash
commander arm
```

Test takeoff:

```bash
commander takeoff
```

Drone should rise.

Test landing:

```bash
commander land
```

Drone should land.

If all commands work, PX4 is healthy.

---

# Step 5: Load ROS2 Workspace

Terminal 3

```bash
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash
```

Verify PX4 topics:

```bash
ros2 topic list | grep fmu
```

Expected:

```text
/fmu/in/offboard_control_mode
/fmu/in/trajectory_setpoint
/fmu/in/vehicle_command
...
```

---

# Step 6: Verify Offboard Communication

Inside PX4 console:

```bash
listener vehicle_status
```

Expected:

```text
accepts_offboard_setpoints: True
```

This confirms PX4 accepts commands from ROS2.

---

# Step 7: Keyboard Control Node

File:

```text
~/px4_ros_ws/src/drone_keyboard/drone_keyboard/keyboard_control.py
```

Code:

```python
import rclpy
from rclpy.node import Node

from px4_msgs.msg import (
    OffboardControlMode,
    TrajectorySetpoint,
    VehicleCommand
)

import threading

class KeyboardControl(Node):

    def __init__(self):

        super().__init__('keyboard_control')

        self.offboard_pub = self.create_publisher(
            OffboardControlMode,
            '/fmu/in/offboard_control_mode',
            10)

        self.traj_pub = self.create_publisher(
            TrajectorySetpoint,
            '/fmu/in/trajectory_setpoint',
            10)

        self.cmd_pub = self.create_publisher(
            VehicleCommand,
            '/fmu/in/vehicle_command',
            10)

        self.x = 0.0
        self.y = 0.0
        self.z = -3.0
        self.yaw = 0.0

        self.counter = 0

        self.timer = self.create_timer(
            0.05,
            self.publish_setpoint)

    def publish_setpoint(self):

        offboard = OffboardControlMode()
        offboard.position = True

        self.offboard_pub.publish(offboard)

        traj = TrajectorySetpoint()
        traj.position = [
            self.x,
            self.y,
            self.z
        ]
        traj.yaw = self.yaw

        self.traj_pub.publish(traj)

        if self.counter == 20:
            self.arm()
            self.offboard()

        self.counter += 1

    def arm(self):

        msg = VehicleCommand()

        msg.command = VehicleCommand.VEHICLE_CMD_COMPONENT_ARM_DISARM
        msg.param1 = 1.0

        self.cmd_pub.publish(msg)

    def offboard(self):

        msg = VehicleCommand()

        msg.command = VehicleCommand.VEHICLE_CMD_DO_SET_MODE
        msg.param1 = 1.0
        msg.param2 = 6.0

        self.cmd_pub.publish(msg)

def keyboard_thread(node):

    print("\nControls")
    print("W Forward")
    print("S Backward")
    print("A Left")
    print("D Right")
    print("Q Up")
    print("E Down")
    print("X Exit")

    while True:

        key = input("Key: ").lower()

        if key == 'w':
            node.x += 2.5

        elif key == 's':
            node.x -= 2.5

        elif key == 'a':
            node.y += 2.5

        elif key == 'd':
            node.y -= 2.5

        elif key == 'q':
            node.z -= 2.5

        elif key == 'e':
            node.z += 2.5

        elif key == 'x':
            break

        print(
            f"Target -> X:{node.x} "
            f"Y:{node.y} "
            f"Z:{node.z}"
        )

def main():

    rclpy.init()

    node = KeyboardControl()

    thread = threading.Thread(
        target=keyboard_thread,
        args=(node,),
        daemon=True)

    thread.start()

    rclpy.spin(node)

    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

# Step 8: Build Package

```bash
cd ~/px4_ros_ws

colcon build --packages-select drone_keyboard --symlink-install
```

---

# Step 9: Source Workspace

```bash
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash
```

---

# Step 10: Run Keyboard Controller

```bash
ros2 run drone_keyboard keyboard_control
```

---

# Keyboard Commands

| Key | Action              |
| --- | ------------------- |
| W   | Move Forward 2.5 m  |
| S   | Move Backward 2.5 m |
| A   | Move Left 2.5 m     |
| D   | Move Right 2.5 m    |
| Q   | Move Up 2.5 m       |
| E   | Move Down 2.5 m     |
| X   | Exit                |

---

# Expected Output

```text
Controls

W Forward
S Backward
A Left
D Right
Q Up
E Down
X Exit

Key: W

Target -> X:2.5 Y:0.0 Z:-3.0
```

---

# Learning Outcomes

After completing this project you will understand:

* PX4 SITL Simulation
* Gazebo Harmonic Integration
* ROS2 Jazzy Communication
* DDS Middleware
* Offboard Control
* Drone Teleoperation
* Position Setpoints
* Real-Time Flight Control

---

# Author

Vikas Kolhe

STEM Innovation Engineer | Robotics and Embedded Systems Enthusiast

Specializing in ROS2, PX4, Autonomous Systems, Embedded Electronics, and AI Robotics.
