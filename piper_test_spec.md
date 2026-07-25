# piper机械臂测试节点spec

前置已完成：
机械臂驱动，已经可以通过如下方式给末端点让机械臂运动到想要的位置：ros2 topic pub /pos_cmd piper_msgs/msg/PosCmd "{x: 0.25, y: 0.00, z: 0.27, roll: 0.00, pitch: 1.57, yaw: 0.00, gripper: 0.0, mode1: 0, mode2: 0}"

需求：新建一个节点来平替掉ros2 topic pub /pos_cmd piper_msgs/msg/PosCmd "{x: 0.25, y: 0.00, z: 0.27, roll: 0.00, pitch: 1.57, yaw: 0.00, gripper: 0.0, mode1: 0, mode2: 0}"这种方式去控制末端位置，控制方式如下：需要6个拖动滑块，滑块1~3为控制末端位置的xyz坐标，4~6为末端欧拉角(roll、yaw、pitch)发布频率20hz

初始化位置（注意这里不是给的欧拉角，给的是四元数）：
position:
  x: 0.055142
  y: -0.002581
  z: 0.217777
orientation:
  x: 0.0009725635585632747
  y: 0.6530434831441294
  z: -0.029481433351552566
  w: 0.7567457355880148
