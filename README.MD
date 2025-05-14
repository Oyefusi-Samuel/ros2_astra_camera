#ros2_astra_camera

A ROS 2 Humble driver for Orbbec 3D cameras

Install dependencies
--------------------
ROS 2

● Please refer to the official ROS 2 Humble installation guide

Other dependencies

● Install system and ROS 2 dependencies:

# Make sure you have sourced the ROS 2 Humble environment
sudo apt install libgflags-dev \
  ros-humble-image-geometry \
  ros-humble-camera-info-manager \
  ros-humble-image-transport \
  ros-humble-image-publisher \
  libgoogle-glog-dev \
  libusb-1.0-0-dev \
  libeigen3-dev

Install libuvc
--------------
git clone https://github.com/libuvc/libuvc.git
cd libuvc
mkdir build && cd build
cmake .. && make -j4
sudo make install
sudo ldconfig

Getting Started
---------------
Use Case

● Create a new ROS 2 workspace:
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

● Clone the driver:
git clone https://github.com/Oyefusi-Samuel/ros2_astra_camera.git

● Install udev rules:
cd ~/ros2_ws/src/ros2_astra_camera/astra_camera/scripts
sudo bash install.sh
sudo udevadm control --reload-rules && sudo udevadm trigger

Build the workspace
-------------------
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
colcon build --event-handlers console_direct+ --cmake-args -DCMAKE_BUILD_TYPE=Release

Launch the camera
-----------------
Terminal 1:
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 launch astra_camera astra.launch.xml

Terminal 2:
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
rviz2

Useful ROS 2 commands
---------------------
source /opt/ros/humble/setup.bash

ros2 topic list
ros2 service list
ros2 param list

Service Usage Examples
----------------------
Camera Info:
ros2 service call /camera/get_camera_info astra_camera_msgs/srv/GetCameraInfo '{}'
ros2 topic echo /camera/depth/camera_info
ros2 topic echo /camera/color/camera_info

Device Info:
ros2 service call /camera/get_device_info astra_camera_msgs/srv/GetDeviceInfo '{}'
ros2 service call /camera/get_sdk_version astra_camera_msgs/srv/GetString "{}"

Exposure Settings:
# Disable auto exposure first
ros2 service call /camera/set_color_auto_exposure std_srvs/srv/SetBool '{data: false}'
ros2 service call /camera/set_uvc_auto_exposure std_srvs/srv/SetBool '{data: false}'
ros2 service call /camera/set_ir_auto_exposure std_srvs/srv/SetBool '{data: false}'

# Set exposure
ros2 service call /camera/set_color_exposure astra_camera_msgs/srv/SetInt32 '{data: 2000}'
ros2 service call /camera/set_uvc_exposure astra_camera_msgs/srv/SetInt32 '{data: 2000}'
ros2 service call /camera/set_ir_exposure astra_camera_msgs/srv/SetInt32 '{data: 2000}'

# Get exposure
ros2 service call /camera/get_color_exposure astra_camera_msgs/srv/GetInt32 '{}'
ros2 service call /camera/get_uvc_exposure astra_camera_msgs/srv/GetInt32 '{}'
ros2 service call /camera/get_ir_exposure astra_camera_msgs/srv/GetInt32 '{}'

Gain Settings:
# Set gain
ros2 service call /camera/set_color_gain astra_camera_msgs/srv/SetInt32 '{data: 200}'
ros2 service call /camera/set_ir_gain astra_camera_msgs/srv/SetInt32 '{data: 200}'
ros2 service call /camera/set_uvc_gain astra_camera_msgs/srv/SetInt32 '{data: 200}'

# Get gain
ros2 service call /camera/get_color_gain astra_camera_msgs/srv/GetInt32 '{}'
ros2 service call /camera/get_ir_gain astra_camera_msgs/srv/GetInt32 '{}'
ros2 service call /camera/get_uvc_gain astra_camera_msgs/srv/GetInt32 '{}'

Misc Controls
-------------
Mirror Mode:
ros2 service call /camera/set_color_mirror std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/set_ir_mirror std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/set_depth_mirror std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/set_uvc_color_mirror std_srvs/srv/SetBool '{data: true}'

White Balance:
ros2 service call /camera/get_color_auto_white_balance astra_camera_msgs/srv/GetInt32 '{}'
ros2 service call /camera/set_color_auto_white_balance std_srvs/srv/SetBool '{data: false}'

Enable/Disable Features:
# Laser
ros2 service call /camera/set_laser_enable std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/set_laser_enable std_srvs/srv/SetBool '{data: false}'

# Fan
ros2 service call /camera/set_fan_mode std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/set_fan_mode std_srvs/srv/SetBool '{data: false}'

# LDP
ros2 service call /camera/set_ldp_enable std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/set_ldp_enable std_srvs/srv/SetBool '{data: false}'

# Toggle sensors
ros2 service call /camera/toggle_ir std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/toggle_color std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/toggle_depth std_srvs/srv/SetBool '{data: true}'
ros2 service call /camera/toggle_uvc_camera std_srvs/srv/SetBool '{data: true}'

Multi-Camera Setup
------------------
Get serial numbers:
ros2 run astra_camera list_devices_node

Edit the multi_astra.launch.xml file and update the serial numbers accordingly.

Clear shared memory (important!):
ros2 run astra_camera cleanup_shm_node

Launch multi-camera:
ros2 launch astra_camera multi_astra.launch.xml

Use Calibration Camera Parameters
---------------------------------
Update the launch file:
<arg name="ir_info_url" default="file:///your_ir_camera_calib_path/depth_camera.yaml"/>
<arg name="color_info_url" default="file:///your_color_camera_calib_path/rgb_camera.yaml"/>

Sample YAML format:
image_width: 640
image_height: 480
camera_name: rgb_camera
camera_matrix:
  rows: 3
  cols: 3
  data: [517.301, 0, 326.785, 0, 519.291, 244.563, 0, 0, 1]
distortion_model: plumb_bob
distortion_coefficients:
  rows: 1
  cols: 5
  data: [0.1, -0.25, 0.001, 0.0005, 0.1]

* Calibration file should be like

> Be careful the camera name The color camera is rgb_camera, the depth/IR camera name is ir_camera

```yaml
image_width: 640
image_height: 480
# The camera name is fixed. The color camera is rgb_camera, the depth/IR camera name is ir_camera
camera_name: rgb_camera
camera_matrix:
  rows: 3
  cols: 3
  data: [517.301, 0, 326.785, 0, 519.291, 244.563, 0, 0, 1]
distortion_model: plumb_bob
distortion_coefficients:
  rows: 1
  cols: 5
  data: [-0.41527, 0.31874, -0.00197, 0.00071, 0]
rectification_matrix:
  rows: 3
  cols: 3
  data: [0.999973, 0.00612598, -0.00406652, -0.00610201, 0.999964, 0.00588094, 0.0041024, -0.00585596, 0.999974 ]
projection_matrix:
  rows: 3
  cols: 4
  data: [517.301, 0, 326.785, -25.3167, 0, 519.291, 244.563, 0.282065, 0, 0, 1, 0.0777703]

```

## Launch parameters

* `connection_delay`, The delay time for reopening the device in seconds. Some devices would take longer time to
  initialize, such as Astra mini, so reopening the device immediately would causes firmware crashes when hot plug.

* `enable_point_cloud`, Whether to enable point cloud.
* `enable_colored_point_cloud`, Whether to enable RGB point cloud.
* `point_cloud_qos`, `[color|depth|ir]_qos`,`[color|depth|ir]_camera_info_qos`, ROS2 Message QoS,
  can be set to `SYSTEM_DEFAULT`, `DEFAULT`,`PARAMETER_EVENTS` ,`SERVICES_DEFAULT`, `PARAMETERS`, `SENSOR_DATA`,
  corresponds to `rmw_qos_profile_system_default`, `rmw_qos_profile_default`, `rmw_qos_profile_parameter_events`
 `rmw_qos_profile_services_default`,`rmw_qos_profile_parameters`, `SENSOR_DATA`,Case-insensitive.
* `enable_d2c_viewer`, Publish D2C overlay image (For Test only).
* `device_num`,The number of devices, You need to fill in the number of devices when you need multiple cameras.
* `color_width`， `color_height`， `color_fps`， color stream resolution and frame rate.
* `ir_width`， `ir_height`， `ir_fps`，IR stream resolution and frame rate
* `depth_width`， `depth_height`， `depth_fps` depth stream resolution and frame rate
* `enable_color`， Whether to enable RGB camera, this parameter has no effect when the RGB camera is UVC protocol
* `enable_depth` , Whether to enable depth camera
* `enable_ir`, Whether to enable IR camera
* `depth_registration`, Enables hardware depth to color alignment, requires RGB point cloud to open
* `depth_scale`, Depth image zoom scale, e.g. set to 2 means aligning depth 320x240 to RGB 640x480
* `color_roi_x`， `color_roi_y`， `color_roi_width`， `color_roi_height`, Whether to crop RGB images, the default is -1,
  which is only used when the RGB resolution is greater than the depth resolution and needs to be aligned. For example,
  if you need to align the depth 640x400 to RGB 640x480, you need to set color_roi_x: 0, color_roi_y: 0,
  color_roi_width: 640, color_roi_height: 400. roi_height: 400, which will crop the top 400 pixels of the RGB with a
  corresponding depth ROI.
* `color_depth_synchronization`，Enable synchronization of RGB with depth
* `use_uvc_camera`，if the RGB camera is UVC protocol, setting as true, UVC is the protocol that currently includes
  dabai, dabai_dcw etc.
* `uvc_product_id`，pid of UVC camera
* `uvc_camera_format`，Image format for uvc camera
* `uvc_retry_count` sometimes the UVC protocol camera does not reconnect successfully when hot-plug, requiring many
  times to retry.
* `enable_publish_extrinsic` Enable publish camera extrinsic.
* `oni_log_level`, Log levels for OpenNI verbose/ info /warning/ error /none
* `oni_log_to_console`, Whether to output OpenNI logs to the console
* `oni_log_to_file`, Whether to output OpenNI logs to a file, by default it will save in Log folder under the path of
  the currently running program

## DDS Tuning

The default DDS settings (Galactic) may not be efficient for data transmission. Different DDS settings will have
different performance. Here we take CycloneDDS as an example. For detailed information, please refer
to[ROS DDS Tuning](https://docs.ros.org/en/humble/How-To-Guides/DDS-tuning.html)。

● Edit cyclonedds configuration file

```bash
sudo gedit /etc/cyclonedds/config.xml
```

Add

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="https://cdds.io/confighttps://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
    <Domain id="any">
        <General>
            <NetworkInterfaceAddress>lo</NetworkInterfaceAddress>
            <AllowMulticast>false</AllowMulticast>
        </General>
        <Internal>
            <MinimumSocketReceiveBufferSize>16MB</MinimumSocketReceiveBufferSize>
        </Internal>
        <Discovery>
            <ParticipantIndex>auto</ParticipantIndex>
            <MaxAutoParticipantIndex>30</MaxAutoParticipantIndex>
            <Peers>
                <Peer address="localhost"/>
            </Peers>
        </Discovery>
    </Domain>
</CycloneDDS>
```

● Set the environment variables, add to .zshrc or .bashrc

```bash
export ROS_DOMAIN_ID=42 # Numbers from 0 to 232
export ROS_LOCALHOST_ONLY=1
export CYCLONEDDS_URI=file:///etc/cyclonedds/config.xml
```

Tip：to understand why the maximum ROS_DOMAIN_ID is 232, please
visit [The ROS DOMAIN ID](https://docs.ros.org/en/humble/Concepts/About-Domain-ID.html)
● Increase UDP receive buffer size
Edit

```bash
/etc/sysctl.d/10-cyclone-max.conf
```

Add

```bash
net.core.rmem_max=2147483647
net.core.rmem_default=2147483647
```

## Frequently Asked Questions

● No point cloud or image displayed on Rviz2

* The default QoS for outgoing messages is sense_data, i.e. Relaiblity Policy = Best Effort, please check if the
  Relaiblity Policy is set to Best Effort on Rviz2, and also check if there is any problem with the QoS setting of the
  message via `ros2 topic info -v /xxx/topic`

● No picture from multiple cameras

* It is possible that the power supply is insufficient, do not connect all cameras to the same hub, and a powered hub is
  recommended

* It is also possible that the resolution is too high, try turning it lower
* If the color camera is an UVC camera, the serial number of the UVC may not be set

## License

Copyright 2023 Orbbec Ltd.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this project except in compliance with
the License. You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "
AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific
language governing permissions and limitations under the License.

**Other names and brands may be claimed as the property of others**
