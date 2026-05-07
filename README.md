# Segway RMP ROS 2 Driver

[![Ubuntu 22.04](https://img.shields.io/badge/Ubuntu-22.04%20LTS-orange)](https://releases.ubuntu.com/22.04/)
[![ROS 2 Humble](https://img.shields.io/badge/ROS%202-Humble%20Hawksbill-blue)](https://docs.ros.org/en/humble/index.html)
[![C++](https://img.shields.io/badge/Language-C%2B%2B-blue.svg)](https://isocpp.org/)
[![Calibrated For](https://img.shields.io/badge/Calibrated%20For-RMP50%20%2F%20RMP100-9cf)](https://www.segway.com/)

This repository provides a native ROS2 driver based on libsegwayrmp to control the base and receive sensor feedback from the robot. It enables teleoperation, integration with navigation stacks (e.g., Nav2), and use in research & autonomy applications.

🔍 Features

- 🦾 ROS2 C++ driver node for Segway RMP platforms

- 📡 Publishes odometry, IMU, and status topics

- 🕹️ Teleoperation support via standard ROS2 interfaces

- 📦 Compatible with ROS2 Humble+

- 🚀 Easy to launch and integrate into robot systems

## Dependencies

* **ROS 2 Humble Hawksbill:** Not a dependency, but it is the only version we tested on (Desktop Install recommended).
* **Ubuntu 22.04 LTS**
* **`serial_for_ros2`:** https://github.com/utexas-bwi/segway_rmp_ros2.
* **`libsegwayrmp`:** https://github.com/utexas-bwi/libsegwayrmp_ros2.

## Installation

1.  **Set up your ROS 2 Workspace:**
    If you don't have one, create a workspace (e.g., `~/ros2_ws`).
    ```bash
    mkdir -p ~/ros2_ws/src
    cd ~/ros2_ws
    ```

2.  **Clone Dependencies:**
    Clone the `serial_for_ros2` driver and the `libsegwayrmp_ros2` library into your workspace's `src` directory. Make sure to remove any existing versions first.
    ```bash
    cd ~/ros2_ws/src
    rm -rf serial_for_ros2 libsegwayrmp libsegwayrmp_ros2 segway_rmp_ros2 # Clean old versions
    git clone https://github.com/utexas-bwi/serial_for_ros2.git
    git clone https://github.com/utexas-bwi/libsegwayrmp_ros2.git
    ```

3.  **Clone this Repository:**
    Clone this `segway_rmp_ros2` driver package into your `src` directory.
    ```bash
    cd ~/ros2_ws/src
    git clone https://github.com/JPLDevMaster/segway_rmp_ros2.git
    ```

4.  **Install Dependencies:**
    Use `rosdep` to install any missing system dependencies.
    ```bash
    cd ~/ros2_ws
    sudo apt update
    rosdep update
    rosdep install --from-paths src --ignore-src -r -y
    ```

5.  **Build the Workspace:**
    Build all the packages.
    ```bash
    cd ~/ros2_ws
    colcon build --symlink-install
    ```

6.  **Source the Workspace:**
    Remember to source your workspace in every new terminal or add it to your `.bashrc`.
    ```bash
    source ~/ros2_ws/install/setup.bash
    ```

## Usage

Launch the driver using the provided launch file. You can override parameters like the serial port or log level via the command line.

```bash
# Basic launch with default parameters (log level = info)
ros2 launch segway_rmp_ros2 segway_rmp_ros2.launch.py

# Launch with debug logging enabled
ros2 launch segway_rmp_ros2 segway_rmp_ros2.launch.py log_level:=debug

# Launch specifying a different serial port
ros2 launch segway_rmp_ros2 segway_rmp_ros2.launch.py serial_port:=/dev/ttyUSB1
```

Important: Because the low-level libsegwayrmp library is calibrated, ensure that the linear_odom_scale parameter in the launch file is set to 1.0, or just ignore it and let it use the default value.

```bash
<param name="linear_odom_scale" value="1.0" />
```

## Key Improvements Detailed

* **Velocity Logging:** When the `log_level` parameter is set to `debug`, the node now prints detailed comparisons:
    * `Cmd`: The velocity received on the `/cmd_vel` topic (after applying limits).
    * `Sent`: The velocity sent to the hardware (after applying acceleration limits and potential `linear_vel_scale`).
    * `Measured`: The velocity calculated from the robot's odometry feedback.
    * `% Diff`: Percentage differences between these values, helping to quickly identify discrepancies.
* **`TwistStamped` Input:** Accepts `geometry_msgs/msg/TwistStamped` for better timestamp handling and compatibility, especially with Nav2.
* **Parameterized Logging:** Internal debug messages (like raw commands sent) are now conditional based on the `log_level` parameter, reducing console spam during normal operation.

## Parameters

This node accepts various parameters to configure its behavior. Key parameters include:

| Parameter Name                | Default Value                | Description                                                                                         |
|-------------------------------|------------------------------|-----------------------------------------------------------------------------------------------------|
| `interface_type`              | `"serial"`                   | Communication interface (`serial` or `usb`).                                                        |
| `serial_port`                 | `"/dev/ttyUSB0"`             | Device path for the serial port (used if `interface_type` is `serial`).                             |
| `usb_selector`                | `"index"`                    | Method to select USB device (`index`, `serial_number`, `description`) (if `interface_type` is `usb`). |
| `usb_index`                   | `0`                          | Index of the USB device (if `usb_selector` is `index`).                                             |
| `usb_serial_number`           | `"00000000"`                 | Serial number of the USB device (if `usb_selector` is `serial_number`).                             |
| `usb_description`             | `"Robotic Mobile Platform"`  | Description string of the USB device (if `usb_selector` is `description`).                        |
| `motor_timeout`               | `0.5`                        | Duration (s) without `/cmd_vel` before stopping motors.                                           |
| `frame_id`                    | `"base_link"`                | Robot's base TF frame ID. Used as `child_frame_id` for odometry TF.                                 |
| `odom_frame_id`               | `"odom"`                     | Odometry TF frame ID. Used as `header.frame_id` for odometry TF and messages.                       |
| `invert_linear_vel_cmds`      | `false`                      | Invert sign of incoming linear velocity commands if `true`.                                         |
| `invert_angular_vel_cmds`     | `false`                      | Invert sign of incoming angular velocity commands if `true`.                                        |
| `broadcast_tf`                | `true`                       | Publish the odometry transform (`odom` -> `base_link`) if `true`.                                   |
| `rmp_type`                    | `"50/100"`                   | Segway model (`50/100` or `200/400`) for loading calibration constants in `libsegwayrmp`.           |
| `linear_pos_accel_limit`      | `0.0`                        | Max linear acceleration (m/s²). `0.0` means no limit.                                               |
| `linear_neg_accel_limit`      | `0.0`                        | Max linear deceleration (m/s²). `0.0` means no limit.                                               |
| `angular_pos_accel_limit`     | `0.0`                        | Max angular acceleration (deg/s²). `0.0` means no limit.                                              |
| `angular_neg_accel_limit`     | `0.0`                        | Max angular deceleration (deg/s²). `0.0` means no limit.                                              |
| `max_linear_vel`              | `0.0`                        | Max allowed linear velocity command (m/s). `0.0` means no limit.                                    |
| `max_angular_vel`             | `0.0`                        | Max allowed angular velocity command (rad/s). `0.0` means no limit.                                 |
| `linear_odom_scale`           | `1.0`                        | Scale factor applied to the linear component of calculated odometry.                              |
| `angular_odom_scale`          | `1.0`                        | Scale factor applied to the angular component of calculated odometry.                             |
| `reset_odometry`              | `false`                      | Attempt hardware reset of odometry integrators on startup if `true`.                                |
| `odometry_reset_duration`     | `1.0`                        | Timeout (s) for waiting for hardware odometry reset.                                              |
| `linear_vel_scale`            | `true`                       | Apply velocity-dependent scaling to linear velocity sent to motors if `true`.                     |
| `angular_vel_scale`           | `true`                       | Apply scaling factor to angular velocity sent to motors if `true`.                                |
| `log_level`                   | `"info"`                     | Controls custom debug output (`debug`, `info`, `error`). `debug` prints detailed velocity info. |

## Testing

Once the node is launched, you can test basic motion control using standard ROS 2 tools:

* **Teleoperation:**
    ```bash
    ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -p stamped:=true
    ```
    *(Note the `stamped:=true` so that the teleop node published TwistStamped messages)*

* **Manual Publishing:**
    ```bash
    # Publish a single command.
    ros2 topic pub -r 20 /cmd_vel geometry_msgs/msg/TwistStamped '{header: {frame_id: "base_link"}, twist: {linear: {x: 0.2, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}}' -1
    ```
    *(Note the `-r 20` argument, the publishing rate in Hz should be more or equal to the operating frequency of the segway library (20 Hz))*

You should see the robot move and observe accurate velocity reporting if the `log_level` is set to `debug`.
