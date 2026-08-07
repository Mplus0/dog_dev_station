# dog_dev_station

用于查看和调试四足机器人定位系统的开发机端工具。
机器人端应以无界面方式运行 VINS/桥接链路，RViz 则在开发机上运行。

比赛现场更换 WiFi 或 IP 地址变化时，请参阅 [`README_FIELD_NETWORK.md`](README_FIELD_NETWORK.md)。

## RViz 工作空间

在 Ubuntu/ROS Noetic 虚拟机中完成一次编译：

```bash
cd ~/dog_dev_station/rviz_ws
catkin_make
source devel/setup.bash
```

将虚拟机指向机器人端的 ROS Master。建议虚拟机使用桥接网络，使机器人能够直接反向连接虚拟机。请将以下 IP 替换为当前网络中的实际地址：

```bash
export ROS_MASTER_URI=http://192.168.137.144:11311
export ROS_IP=192.168.137.xxx
```

检查连接：

```bash
rostopic list
rostopic hz /dog_vins/vins_estimator/odometry
rostopic hz /leg_odom2
```

在开发机上启动 RViz：

```bash
roslaunch dog_dev_rviz dog_vins_remote_rviz.launch
```

如果机器人没有发布辅助 TF，而你只需要进行可视化对比，可以从开发机发布这些 TF：

```bash
roslaunch dog_dev_rviz dog_vins_remote_rviz.launch publish_world_odom_tf:=true publish_camera_base_tf:=true
```

RViz 配置文件位于：

```text
rviz_ws/src/dog_dev_rviz/config/dog_vins_localization.rviz
```

## comp2026_ws RViz 迁移说明

当前的 `dog_dev_rviz` 默认被设计为仅用于显示。它不会启动 RealSense、VINS、机器人桥接、导航、`robot_state_publisher` 或任务节点。

后续可以安全迁移的内容：

```text
comp2026_ws/src/allmovebase/launch/rviz_nav.launch
comp2026_ws/src/allmovebase/rviz/nav_debug.rviz
comp2026_ws/src/lite3_description/rviz/lite3_model_check.rviz
```

除非正在进行离线 rosbag 或模型调试，否则应将以下内容保留在机器人端运行：

```text
comp2026_ws/src/allmovebase/launch/camera2base_tf.launch
comp2026_ws/src/allmovebase/scripts/camera_static_tf_from_yaml.py
comp2026_ws/src/lite3_description/launch/lite3_model_check.launch
comp2026_ws/src/lite3_description/launch/lite3_map_amcl_debug.launch
```

原因是这些 launch 文件可能会发布 TF、`robot_description`、关节状态，或者启动地图、AMCL、`robot_state_publisher`。如果在机器人主链路运行期间从开发机重复启动它们，可能产生重复坐标系、重复节点名称或冲突的 `/use_sim_time` 设置。

## comp2026_ws 导航 RViz

该开发机端 RViz 入口用于调试 `comp2026_ws` 中的国赛导航系统。机器人端应运行 ROS Master 和无界面的导航/任务 launch，RViz 则在虚拟机中运行。

使用 AX3600 调试局域网时，确认机器人 IP 后，虚拟机中的典型环境配置如下：

```bash
export ROS_MASTER_URI=http://<robot_ip>:11311
export ROS_IP=<vm_ip>
```

当前规划的 AX3600 地址：

```text
路由器：     192.168.31.1
宿主机：     192.168.31.10
虚拟机：     192.168.31.11
机器人：     现场测试后确定
机械臂 RDK X5：现场测试后确定
```

从虚拟机检查连接：

```bash
rostopic list
rostopic hz /scan
rostopic hz /amcl_pose
rostopic echo -n 1 /move_base/status
```

启动远程 RViz：

```bash
cd ~/dog_dev_station/rviz_ws
source devel/setup.bash
roslaunch dog_dev_rviz dog_comp2026_nav_remote_rviz.launch
```

RViz 配置文件位于：

```text
rviz_ws/src/dog_dev_rviz/config/comp2026_nav_debug.rviz
```

该 launch 只启动 RViz。它不得在虚拟机上启动相机、AMCL、`move_base`、任务节点、`robot_state_publisher` 或 TF 发布器。
除非明确要调试 URDF 或 rosbag 数据，否则应将模型检查 launch 保留在机器人端，或放在单独的离线工作空间中运行。

## comp2026_ws Lite3 模型 RViz

由于 `comp2026_ws` 中原有的 `lite3_model_check.launch` 不仅会打开 RViz，还会执行其他操作，因此针对开发机端用途对其进行了拆分：

```text
- 从 Lite3 URDF 加载 robot_description
- 启动 joint_state_publisher_gui 或 joint_state_publisher
- 启动 robot_state_publisher
- 发布静态 base_link -> TORSO TF
- 启动 RViz
```

在机器人实时运行期间，请使用仅显示模式的远程 RViz 入口。它不会从虚拟机发布 TF 或机器人模型数据：

```bash
cd ~/dog_dev_station/rviz_ws
source devel/setup.bash
roslaunch dog_dev_rviz dog_comp2026_lite3_model_remote_rviz.launch
```

如果机器人端没有发布 `/robot_description` 和模型 TF，RobotModel 显示可能会保持为空。对于安全的远程显示模式，这是预期现象。

如需在虚拟机中离线检查 URDF/模型，请使用离线 launch。该入口会启动本地复制的 `dog_dev_lite3_description` 包、关节状态发布器、机器人状态发布器、静态 TF 和 RViz：

```bash
cd ~/dog_dev_station/rviz_ws
source devel/setup.bash
roslaunch dog_dev_rviz dog_comp2026_lite3_model_offline_check.launch
```

除非确实需要重复发布 `robot_description` 和 TF，否则不要在连接机器人实时 ROS Master 的情况下运行离线模型 launch。

为避免与 `comp2026_ws` 中的软件包发生 rospack 冲突，复制后的模型包已重命名为：

```text
rviz_ws/src/dog_dev_lite3_description
```

开发机端 RViz 配置文件：

```text
rviz_ws/src/dog_dev_rviz/config/comp2026_lite3_model_check.rviz
```
