#

需求：在src/advx里面新建一个功能包，该节点将face这个tf转换为机械臂末端目标位置arm_pose这个tf，也就是机械臂末端要达到的位置，他俩之间的位置关系这里有个例子，如下：
ros2 run tf2_ros tf2_echo face link6
At time 1784971456.177666048
- Translation: [-0.189, -0.044, -0.478]
- Rotation: in Quaternion (xyzw) [-0.180, 0.102, -0.769, 0.605]
- Rotation: in RPY (radian) [-0.389, -0.155, -1.777]
- Rotation: in RPY (degree) [-22.292, -8.877, -101.822]

ros2 run tf2_ros tf2_echo base_link link6
At time 1784972214.879953861
- Translation: [0.220, -0.014, 0.337]
- Rotation: in Quaternion (xyzw) [0.026, 0.648, 0.008, 0.761]
- Rotation: in RPY (radian) [0.306, 1.403, 0.280]
- Rotation: in RPY (degree) [17.537, 80.409, 16.018]
- Matrix:
  0.160  0.022  0.987  0.220
  0.046  0.998 -0.030 -0.014
 -0.986  0.050  0.159  0.337
  0.000  0.000  0.000  1.000



这里关注几个点：
1.机械臂最终需要达到的arm_pose的tf的roll、yaw、pitch需要在config里面可设置，默认值为link6在baselink下的位姿Rotation: in RPY (radian) [0.306, 1.403, 0.280]
2.机械臂最终要达到的arm_pose的tf的xyz由face这个tf得到，face的xyz经过偏移一定距离得到arm_pose的xyz，也要在config里面可以改这个，默认值可以用当前face到link6的
3.在上述中，face -> link6是用来测量默认合适偏差值的，这个状态下的link6就是默认偏差值下的arm_pose，最后只有机械臂实际运动的时候link6才会运动到arm_pose

总结：arm_pose的RPY为config定死，xyz由face经过一定偏移得到


测试方式：
我已经运行好了piper机械臂的驱动，tf链也正常，虚拟发布一个face的位置，然后经过新搞的这个节点得到arm_pose的位置，base_link-->face使用如下数据
- Translation: [0.721, -0.057, 0.438]
- Rotation: in Quaternion (xyzw) [0.685, 0.251, 0.495, 0.473]
- Rotation: in RPY (radian) [1.642, -0.456, 1.127]
- Rotation: in RPY (degree) [94.055, -26.124, 64.588]

