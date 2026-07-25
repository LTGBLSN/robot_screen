# 奥比中光 DaBai DC1 使用说明

本文档适用于通过 `OrbbecSDK_ROS2` 在 Ubuntu 22.04 和 ROS 2 Humble 中使用奥比中光 `DaBai DC1` 深度相机。

本文档对应的工作空间路径为：

```text
~/ros2_ws
```

本机已验证的设备信息：

| 项目 | 值 |
| --- | --- |
| 相机型号 | DaBai DC1 |
| USB Vendor ID | `0x2bc5` |
| 深度设备 Product ID | `0x0657` |
| 彩色设备 Product ID | `0x0557` |
| 序列号 | `CC1N16200H8` |
| ROS 2 | Humble |
| Ubuntu | 22.04 |
| 当前连接方式 | USB 2.0 |

## 1. 加载 ROS 2 环境

每个新终端都需要加载 ROS 2 和工作空间环境：

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

当前用户的 `~/.bashrc` 已经配置了这两行。若新终端中仍然找不到
`orbbec_camera`，手动执行上面的命令即可。

确认驱动包已经安装：

```bash
ros2 pkg list | grep -E '^(orbbec_camera|orbbec_camera_msgs|orbbec_description)$'
```

预期输出：

```text
orbbec_camera
orbbec_camera_msgs
orbbec_description
```

## 2. 检查相机连接

### 2.1 使用 USB 检查相机

```bash
lsusb | grep -i -E 'orbbec|2bc5|dabai'
```

应该能看到类似以下设备：

```text
2bc5:0657 Orbbec 3D Technology International, Inc ORBBEC Depth Sensor
2bc5:0557 Orbbec 3D Technology International, Inc Dabai DC1
```

### 2.2 使用 Orbbec SDK 枚举设备

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run orbbec_camera list_devices_node
```

正常情况下会显示相机名称、序列号和 USB 端口，例如：

```text
Name: DaBai DC1
serial: CC1N16200H8
usb port: 3-1.4
```

### 2.3 检查 udev 设备链接

安装 udev 规则后，系统会创建以下链接：

```bash
ls -l /dev/dabai_dc1 /dev/dabai_dc1_rgb
```

如果链接不存在，重新安装规则并重新插拔相机：

```bash
cd ~/ros2_ws/src/OrbbecSDK_ROS2/orbbec_camera/scripts
sudo bash install_udev_rules.sh
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## 3. 启动 DaBai DC1

### 3.1 默认启动

在终端一执行：

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 launch orbbec_camera dabai.launch.py
```

该启动文件默认开启：

- 彩色图像
- 深度图像
- 红外图像
- 深度点云
- TF 静态变换

正常启动后，日志中应出现类似内容：

```text
Device DaBai DC1 connected
Serial number: CC1N16200H8
usb connect type: USB2.0
```

### 3.2 按序列号启动指定相机

连接多台奥比中光相机时，可以使用序列号选择设备：

```bash
ros2 launch orbbec_camera dabai.launch.py \
  serial_number:=CC1N16200H8
```

也可以按照 USB 端口选择：

```bash
ros2 launch orbbec_camera dabai.launch.py \
  usb_port:=3-1.4
```

默认自动选择设备即可。只有连接多台相机或设备选择不正确时，才需要指定
`serial_number` 或 `usb_port`。

### 3.3 停止相机

在运行启动命令的终端按：

```text
Ctrl+C
```

停止相机节点。启动新的相机节点前，先确认旧的节点已经退出，避免设备被占用。

## 4. 查看 ROS 2 话题

相机启动后，在另一个终端加载环境：

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

查看所有相机话题：

```bash
ros2 topic list | grep '^/camera/'
```

DaBai DC1 常用话题如下：

| 话题 | 类型 | 用途 |
| --- | --- | --- |
| `/camera/color/image_raw` | `sensor_msgs/msg/Image` | 彩色图像 |
| `/camera/color/camera_info` | `sensor_msgs/msg/CameraInfo` | 彩色相机内参 |
| `/camera/depth/image_raw` | `sensor_msgs/msg/Image` | 深度图像 |
| `/camera/depth/camera_info` | `sensor_msgs/msg/CameraInfo` | 深度相机内参 |
| `/camera/ir/image_raw` | `sensor_msgs/msg/Image` | 红外图像 |
| `/camera/ir/camera_info` | `sensor_msgs/msg/CameraInfo` | 红外相机内参 |
| `/camera/depth/points` | `sensor_msgs/msg/PointCloud2` | 深度点云 |
| `/camera/depth_to_color` | `orbbec_camera_msgs/msg/Extrinsics` | 深度到彩色外参 |
| `/camera/depth_to_ir` | `orbbec_camera_msgs/msg/Extrinsics` | 深度到红外外参 |
| `/camera/depth_filter_status` | `orbbec_camera_msgs/msg/DepthFiltersStatus` | 深度滤波状态 |

彩色、深度和红外图像还会自动生成压缩传输话题，例如：

```text
/camera/color/image_raw/compressed
/camera/depth/image_raw/compressed
/camera/ir/image_raw/compressed
```

## 5. 检查图像是否正常发布

检查深度图帧率：

```bash
ros2 topic hz /camera/depth/image_raw
```

检查彩色图帧率：

```bash
ros2 topic hz /camera/color/image_raw
```

检查话题类型和发布节点：

```bash
ros2 topic info /camera/color/image_raw
ros2 topic info /camera/depth/image_raw
ros2 topic info /camera/depth/points
```

查看一帧图像的消息头和分辨率：

```bash
ros2 topic echo /camera/color/image_raw --once
ros2 topic echo /camera/depth/image_raw --once
```

默认配置为：

| 数据流 | 分辨率 | 帧率 | 格式 |
| --- | ---: | ---: | --- |
| 彩色 | 640 x 480 | 30 | MJPG |
| 深度 | 640 x 400 | 30 | Y11 |
| 红外 | 640 x 400 | 30 | Y10 |

## 6. 使用 RViz2 查看图像和点云

启动相机后，在另一个终端执行：

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
rviz2
```

在 RViz2 中：

1. 将 `Fixed Frame` 设置为相机发布的有效坐标系。
2. 点击 `Add`，添加 `Image`，选择：
   `/camera/color/image_raw`。
3. 再添加一个 `Image`，选择：
   `/camera/depth/image_raw`。
4. 查看点云时添加 `PointCloud2`，选择：
   `/camera/depth/points`。
5. 如果点云没有显示，先检查 `PointCloud2` 的 `Style`、`Size` 和
   `Fixed Frame` 设置。

查看点云消息使用的坐标系：

```bash
ros2 topic echo /camera/depth/points --once | grep frame_id
```

## 7. 常用启动参数

### 7.1 关闭点云以降低 CPU 和 USB 负载

```bash
ros2 launch orbbec_camera dabai.launch.py \
  enable_point_cloud:=false
```

### 7.2 只启用彩色和深度流

```bash
ros2 launch orbbec_camera dabai.launch.py \
  enable_ir:=false
```

### 7.3 启用彩色点云

```bash
ros2 launch orbbec_camera dabai.launch.py \
  enable_colored_point_cloud:=true
```

启动后检查彩色点云话题：

```bash
ros2 topic list | grep -E 'depth_registered/points|depth/points'
```

### 7.4 启用深度和彩色对齐

```bash
ros2 launch orbbec_camera dabai.launch.py \
  depth_registration:=true
```

如果对齐后无法启动，恢复默认配置：

```bash
ros2 launch orbbec_camera dabai.launch.py \
  depth_registration:=false
```

### 7.5 调整图像分辨率和帧率

例如降低分辨率以减少 USB 负载：

```bash
ros2 launch orbbec_camera dabai.launch.py \
  color_width:=640 \
  color_height:=480 \
  color_fps:=15 \
  depth_width:=640 \
  depth_height:=400 \
  depth_fps:=15
```

DaBai DC1 支持的具体配置应以相机实际返回的 profile 为准。配置不支持
时，节点会在启动日志中报告失败。

## 8. 常用设备控制服务

查看设备信息：

```bash
ros2 service call /camera/get_device_info \
  orbbec_camera_msgs/srv/GetDeviceInfo '{}'
```

查看 SDK 版本：

```bash
ros2 service call /camera/get_sdk_version \
  orbbec_camera_msgs/srv/GetString '{}'
```

查看当前可用服务：

```bash
ros2 service list | grep '^/camera/'
```

常用控制服务包括：

```text
/camera/get_device_info
/camera/get_sdk_version
/camera/get_color_exposure
/camera/get_color_gain
/camera/get_depth_exposure
/camera/get_depth_gain
/camera/get_ir_exposure
/camera/get_ir_gain
/camera/set_color_exposure
/camera/set_color_gain
/camera/set_depth_exposure
/camera/set_depth_gain
/camera/set_ir_exposure
/camera/set_ir_gain
/camera/toggle_color
/camera/toggle_depth
/camera/toggle_ir
```

部分服务只有在对应数据流启用时才可用。

## 9. 常见问题

### 9.1 `ros2 launch` 找不到 `orbbec_camera`

重新加载环境：

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

如果仍然找不到，确认包是否已经构建：

```bash
cd ~/ros2_ws
colcon build --symlink-install \
  --packages-select orbbec_camera_msgs orbbec_camera orbbec_description \
  --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

### 9.2 找不到相机

按以下顺序检查：

```bash
lsusb | grep -i -E 'orbbec|2bc5|dabai'
ros2 run orbbec_camera list_devices_node
ls -l /dev/dabai_dc1 /dev/dabai_dc1_rgb
```

如果 USB 中没有相机：

- 重新插拔 USB 线。
- 更换 USB 接口和数据线。
- 不要通过供电不足的 USB Hub 连接。

如果 USB 能看到相机，但 SDK 枚举不到设备：

```bash
cd ~/ros2_ws/src/OrbbecSDK_ROS2/orbbec_camera/scripts
sudo bash install_udev_rules.sh
sudo udevadm control --reload-rules
sudo udevadm trigger
```

执行后重新插拔相机，再运行 `list_devices_node`。

### 9.3 启动时报设备被占用

检查是否有其他相机节点或图像程序正在运行：

```bash
pgrep -af 'orbbec_camera|component_container|rqt_image_view|rviz2'
```

关闭旧的 ROS 2 节点、RViz2、V4L2 查看工具后重新启动。

### 9.4 USB 2.0 带宽不足

当前这台相机被 SDK 识别为 USB 2.0。USB 2.0 下同时发布彩色、深度和红外
数据时，彩色流实际帧率可能低于配置值。

可以先降低负载：

```bash
ros2 launch orbbec_camera dabai.launch.py \
  enable_ir:=false \
  enable_point_cloud:=false \
  color_fps:=15
```

如果主机和相机都支持 USB 3.x，使用主机的 USB 3.x 接口和合适的数据线，
然后重新执行：

```bash
lsusb -t
ros2 run orbbec_camera list_devices_node
```

### 9.5 日志中出现 `Failed to set 2035 to 1`

DaBai DC1 启动时可能出现关于属性 `2035` 的 SDK 警告。只要后续日志出现
`Device DaBai DC1 connected`，并且 `/camera/depth/image_raw` 等话题持续发布，
该警告通常不影响基本使用。

需要进一步排查时，使用 debug 日志启动：

```bash
ros2 launch orbbec_camera dabai.launch.py \
  log_level:=debug
```

ROS 2 日志默认保存在：

```text
~/.ros/log/
```

## 10. 驱动包和配置文件位置

源码包：

```text
~/ros2_ws/src/OrbbecSDK_ROS2
```

相机 ROS 2 包：

```text
~/ros2_ws/src/OrbbecSDK_ROS2/orbbec_camera
```

DaBai 启动文件：

```text
~/ros2_ws/src/OrbbecSDK_ROS2/orbbec_camera/launch/dabai.launch.py
```

SDK 配置文件：

```text
~/ros2_ws/src/OrbbecSDK_ROS2/orbbec_camera/config/OrbbecSDKConfig_v1.0.xml
```

udev 规则源文件：

```text
~/ros2_ws/src/OrbbecSDK_ROS2/orbbec_camera/scripts/99-obsensor-libusb.rules
```

## 11. 最常用的启动流程

以后日常使用时，只需要：

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch orbbec_camera dabai.launch.py
```

另开一个终端查看数据：

```bash
source ~/ros2_ws/install/setup.bash
ros2 topic list | grep '^/camera/'
ros2 topic hz /camera/depth/image_raw
```

使用结束后，在启动相机的终端按 `Ctrl+C` 停止节点。
