需求：
在src/advx里面新建一个功能包，该节点读取一下arm_pose这个tf作为想要的末端位置，然后对接机械臂的控制节点中的pos_cmd话题让末端移动到arm_pose这个tf在base_link下的坐标位置（应该直接就是，或者读一下piper的驱动pos_cmd对应的是base_link下还是啥的）需要写中文README和config

测试：
我已经运行好了piper机械臂的驱动，并且已经可以通过ros2 topic pub /pos_cmd piper_msgs/msg/PosCmd "{x: 0.25, y: 0.00, z: 0.27, roll: 0.00, pitch: 1.57, yaw: 0.00, gripper: 0.0, mode1: 0, mode2: 0}"控制末端到指定位置
虚拟发布一个arm_pose的位置，然后控制机械臂移动，并通过/end_pose查看是否运动到位，注意测试仅允许机械臂在Z轴上由0.2~0.4这个范围进行移动，xyz及roll，yaw，pitch均不可动（防止空间干涉）
