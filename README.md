# ROS2 drivers to interface with Segway RMP 200/400 mobile robots
This repository provides a native ROS2 driver based on libsegwayrmp to control the base and receive sensor feedback from the robot. It enables teleoperation, integration with navigation stacks (e.g., Nav2), and use in research & autonomy applications. This package is derived from <code>segway_rmp</code> (https://github.com/utexas-bwi/segway_rmp.git)

🔍 Features

- 🦾 ROS2 C++ driver node for Segway RMP platforms

- 📡 Publishes odometry, IMU, and status topics

- 🕹️ Teleoperation support via standard ROS2 interfaces

- 📦 Compatible with ROS2 Humble+

- 🚀 Easy to launch and integrate into robot systems

## Dependencies
### serial
- put the serial folder from the following repo in parallel to your colcon workspace
- <code>git clone https://github.com/utexas-bwi/serial_for_ros2.git</code>

### libsegwayrmp-ros2
- <code>cd [ROS_WS]/src</code>
- <code>git clone https://github.com/utexas-bwi/libsegwayrmp_ros2.git</code>
- use <code>libsegwayrmp</code> as a main package of ros2 humble and not <code>libsegwayrmp_ros2</code>

## Launch
- <code>ros2 launch segway_rmp_ros2 segway_rmp_ros2.launch.py</code>
  
## Test
- <code>ros2 run teleop_twist_keyboard teleop_twist_keyboard</code>
- You should be able to operate the segway with teleop commands now
