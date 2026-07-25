# Piper 单臂无夹爪使用手册

本文只覆盖 `humble` 分支下的单机械臂、无夹爪场景，重点是三类操作：

1. 使能 / 失能
2. 末端位姿直接控制
3. 关节角度直接控制

不包含 MoveIt、仿真和夹爪控制。

## 1. 前置条件

- 已安装 `piper_ros` 的 `humble` 分支
- 已安装 Python 依赖和系统依赖
- USB 转 CAN 模块已经插好
- CAN 口已经激活，波特率为 `1000000`

如果还没有激活 CAN，先执行：

```bash
bash can_activate.sh can0 1000000
```

如果系统里不是 `can0`，把命令里的端口名改成实际名称。

## 2. 启动方式

无夹爪场景建议直接启动单臂节点，并显式关闭夹爪控制：

```bash
cd /home/grubaxu/ros2_ws/src/advx/piper_ros
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch piper start_single_piper.launch.py can_port:=can0 auto_enable:=true gripper_exist:=false
```

如果你想直接跑节点，也可以用：

```bash
ros2 run piper piper_single_ctrl --ros-args -p can_port:=can0 -p auto_enable:=true -p gripper_exist:=false
```

说明：

- `auto_enable:=true` 表示先自动使能
- `gripper_exist:=false` 表示没有夹爪，节点不会走夹爪控制分支
- `start_single_piper.launch.py` 会把 `/joint_ctrl_single` 重映射到 `/joint_states`

## 3. 你需要记住的接口

### 3.1 服务

- `/enable_srv`
- 类型：`piper_msgs/srv/Enable`

服务定义：

```text
bool enable_request
---
bool enable_response
```

### 3.2 控制话题

- `/joint_ctrl_single`
- `/pos_cmd`

如果你是用 `ros2 launch piper start_single_piper.launch.py` 启动的节点，`/joint_ctrl_single` 会被重映射为 `/joint_states`，所以发关节角时要发到 `/joint_states`。

### 3.3 状态话题

- `/arm_status`
- `/joint_states_single`
- `/joint_ctrl`
- `/end_pose`
- `/end_pose_stamped`

其中：

- `/joint_states_single` 是关节反馈
- `/end_pose` 和 `/end_pose_stamped` 是末端位姿反馈
- `/arm_status` 是机械臂状态和错误码

## 4. 使能和失能

### 4.1 推荐方式：服务调用

使能：

```bash
ros2 service call /enable_srv piper_msgs/srv/Enable "{enable_request: true}"
```

失能：

```bash
ros2 service call /enable_srv piper_msgs/srv/Enable "{enable_request: false}"
```

服务返回 `enable_response: true` 时，表示节点已经确认到目标状态。

### 4.2 备选方式：发布 topic

使能：

```bash
ros2 topic pub /enable_flag std_msgs/msg/Bool "{data: true}"
```

失能：

```bash
ros2 topic pub /enable_flag std_msgs/msg/Bool "{data: false}"
```

### 4.3 建议的检查方式

使能后看状态：

```bash
ros2 topic echo /arm_status
```

关注这些字段：

- `err_code`
- `ctrl_mode`
- `arm_status`
- `motion_status`

一般情况下，`err_code` 应该为 `0`。

### 4.4 运行习惯

- 先使能，再发控制命令
- 调试结束后记得失能
- 如果失能后继续发控制命令，节点会忽略这些控制输入

## 5. 末端位姿直接控制

### 5.1 话题和消息

- 话题：`/pos_cmd`
- 消息：`piper_msgs/msg/PosCmd`

消息字段：

```text
float64 x
float64 y
float64 z
float64 roll
float64 pitch
float64 yaw
float64 gripper
int32 mode1
int32 mode2
```

### 5.2 单位

按当前代码实现，建议按下面的单位来发：

- `x / y / z`：米
- `roll / pitch / yaw`：弧度
- `gripper`：无夹爪场景下设为 `0`
- `mode1 / mode2`：当前代码里仅打印，不参与控制，保持 `0` 即可

### 5.3 示例

下面示例表示把末端移动到一个指定位姿：

```bash
ros2 topic pub /pos_cmd piper_msgs/msg/PosCmd "{x: 0.25, y: 0.00, z: 0.27, roll: 0.00, pitch: 1.57, yaw: 0.00, gripper: 0.0, mode1: 0, mode2: 0}"
```

### 5.4 说明

- 这个控制方式是“末端空间控制”
- 节点收到后会把位姿转换成内部控制量再发给机械臂
- 发布一次通常就够了，不需要一直高频重复发

### 5.5 查看末端反馈

如果想确认当前末端位姿，可以看：

```bash
ros2 topic echo /end_pose
```

或者带时间戳的版本：

```bash
ros2 topic echo /end_pose_stamped
```

## 6. 关节角度直接控制

### 6.1 话题

如果你是直接运行节点：

- 话题：`/joint_ctrl_single`

如果你是用 `start_single_piper.launch.py` 启动：

- 话题：`/joint_states`

这两个在 launch 场景下是等价的，因为 launch 里做了重映射。

### 6.2 控制方式

节点会把 `JointState.name` 里的：

- `joint1`
- `joint2`
- `joint3`
- `joint4`
- `joint5`
- `joint6`

映射为机械臂 6 个关节目标角。

无夹爪场景建议只发 6 个关节，不要带第 7 个夹爪关节。

### 6.3 单位

- `position`：弧度
- `velocity`：可选
- `effort`：无夹爪场景可以全部设为 `0`

当前实现里，如果 `position` 里任何关节绝对值超过 `3.5`，节点会给出告警，不建议超这个范围。

### 6.4 示例

直接发 6 关节角：

```bash
ros2 topic pub /joint_ctrl_single sensor_msgs/msg/JointState "{header: {stamp: {sec: 0, nanosec: 0}, frame_id: 'piper_single'}, name: ['joint1', 'joint2', 'joint3', 'joint4', 'joint5', 'joint6'], position: [0.10, 0.20, -0.15, 0.30, -0.20, 0.40], velocity: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], effort: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]}"
```

如果你是通过 launch 启动节点，就改成：

```bash
ros2 topic pub /joint_states sensor_msgs/msg/JointState "{header: {stamp: {sec: 0, nanosec: 0}, frame_id: 'piper_single'}, name: ['joint1', 'joint2', 'joint3', 'joint4', 'joint5', 'joint6'], position: [0.10, 0.20, -0.15, 0.30, -0.20, 0.40], velocity: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], effort: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]}"
```

### 6.5 说明

- 这是“关节空间控制”
- 适合做姿态摆位、回零、重复定位
- 如果你只想保持当前姿态，可以先读 `/joint_states_single`，再把读到的 6 个关节角原样发回去

## 7. 推荐的最小工作流

1. 启动节点
2. 调用 `/enable_srv` 使能
3. 用 `/joint_ctrl_single` 或 `/pos_cmd` 发控制命令
4. 观察 `/arm_status`、`/joint_states_single`、`/end_pose`
5. 结束后调用 `/enable_srv` 失能

## 8. 常见问题

### 8.1 服务一直等不到返回

先确认：

- CAN 口已激活
- `can_port` 参数和实际接口名一致
- 节点正在运行

### 8.2 发了命令但机械臂没动

先检查：

- 是否已经使能
- 控制话题是否发对了
- `start_single_piper.launch.py` 场景下是否发到了 `/joint_states`
- `JointState.name` 是否包含 `joint1` 到 `joint6`

### 8.3 失能后机械臂为什么还保持姿态

这是正常现象。失能只是关闭驱动，不会自动把机械臂拉回原点。
