# 启动启动启动还有这个！

1.相机驱动
ros2 launch orbbec_camera dabai.launch.py depth_registration:=true

2.人脸关键点检测
ros2 launch orbbec_face_landmark orbbec_face_landmark.launch.py

2.1可视化
rviz2 
config里面3D_fase配置文件

3.机械臂部分使能can通讯
先进入到ros2_ws/src/advx/piper_ros
然后
bash can_activate.sh can0 1000000

4.启动机械臂节点
ros2 run piper piper_single_ctrl --ros-args -p can_port:=can0 -p auto_enable:=true -p gripper_exist:=false

4.1滑块测试机械臂是否可用（需要退出示教模式）
方式1（标准）：ros2 topic pub /pos_cmd piper_msgs/msg/PosCmd "{x: 0.25, y: 0.00, z: 0.4, roll: 0.00, pitch: 1.57, yaw: 0.00, gripper: 0.0, mode1: 0, mode2: 0}"
方式2（自定义滑块）：ros2 run piper_pos_slider piper_pos_slider

4.2检查机械臂末端位置
ros2 topic echo /end_pose

5.启动关节数据反馈
ros2 run piper piper_read_slave_joint --ros-args -p can_port:=can0 -p gripper_exist:=false

6.启动URDF并得到完整TF链（带RVIZ）
ros2 launch piper_description  display_no_gripper_urdf_follow.launch.py

6.1查看当前TF状态
rqt就行

7.将camera_link绑定到link6(此处给的是直接相等)
ros2 run tf2_ros static_transform_publisher   0 0 0 0 -1.57 0   link6 camera_link


8.目标位姿节点(todo)
待测试）：ros2 launch face_tf_2_piper_link6 face_tf_2_piper_link6.launch.py

9.机械臂控制节点(finish)
ros2 launch piper_tf_control piper_tf_control.launch.py




At time 1784971454.105523968
- Translation: [-0.233, -0.042, -0.458]
- Rotation: in Quaternion (xyzw) [-0.217, 0.127, -0.777, 0.577]
- Rotation: in RPY (radian) [-0.473, -0.191, -1.818]
- Rotation: in RPY (degree) [-27.080, -10.966, -104.153]
- Matrix:
 -0.240  0.842  0.483 -0.233
 -0.952 -0.302  0.053 -0.042
  0.190 -0.447  0.874 -0.458
  0.000  0.000  0.000  1.000
At time 1784971455.141591040
- Translation: [-0.238, -0.048, -0.455]
- Rotation: in Quaternion (xyzw) [-0.225, 0.123, -0.781, 0.569]
- Rotation: in RPY (radian) [-0.477, -0.213, -1.829]
- Rotation: in RPY (degree) [-27.335, -12.219, -104.814]
- Matrix:
 -0.250  0.834  0.492 -0.238
 -0.945 -0.321  0.064 -0.048
  0.212 -0.449  0.868 -0.455
  0.000  0.000  0.000  1.000
At time 1784971456.177666048
- Translation: [-0.189, -0.044, -0.478]
- Rotation: in Quaternion (xyzw) [-0.180, 0.102, -0.769, 0.605]
- Rotation: in RPY (radian) [-0.389, -0.155, -1.777]
- Rotation: in RPY (degree) [-22.292, -8.877, -101.822]
- Matrix:
 -0.202  0.894  0.401 -0.189
 -0.967 -0.247  0.062 -0.044
  0.154 -0.375  0.914 -0.478
  0.000  0.000  0.000  1.000
