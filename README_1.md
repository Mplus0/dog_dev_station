# dog_dev_station

Developer-side tools for viewing and debugging the quadruped localization stack.
The robot should run the VINS/bridge stack headlessly; RViz runs here.

## RViz Workspace

Build once inside the Ubuntu/ROS Noetic VM:

```bash
cd ~/dog_dev_station/rviz_ws
catkin_make
source devel/setup.bash
```

Point the VM at the robot ROS master. Prefer bridged networking for the VM so
the robot can connect back to the VM directly. Replace the IPs with the current
network:

```bash
export ROS_MASTER_URI=http://192.168.137.144:11311
export ROS_IP=192.168.137.xxx
```

Check connectivity:

```bash
rostopic list
rostopic hz /dog_vins/vins_estimator/odometry
rostopic hz /leg_odom2
```

Start RViz on the developer station:

```bash
roslaunch dog_dev_rviz dog_vins_remote_rviz.launch
```

If the robot is not publishing helper TFs and you only need a visual comparison,
you can publish them from the developer station:

```bash
roslaunch dog_dev_rviz dog_vins_remote_rviz.launch publish_world_odom_tf:=true publish_camera_base_tf:=true
```

The RViz profile is at:

```text
rviz_ws/src/dog_dev_rviz/config/dog_vins_localization.rviz
```

## comp2026_ws RViz Migration Notes

Current `dog_dev_rviz` is intentionally display-only by default. It does not
start RealSense, VINS, robot bridges, navigation, `robot_state_publisher`, or
task nodes.

Safe to migrate later:

```text
comp2026_ws/src/allmovebase/launch/rviz_nav.launch
comp2026_ws/src/allmovebase/rviz/nav_debug.rviz
comp2026_ws/src/lite3_description/rviz/lite3_model_check.rviz
```

Keep these on the robot unless doing offline bag/model debugging:

```text
comp2026_ws/src/allmovebase/launch/camera2base_tf.launch
comp2026_ws/src/allmovebase/scripts/camera_static_tf_from_yaml.py
comp2026_ws/src/lite3_description/launch/lite3_model_check.launch
comp2026_ws/src/lite3_description/launch/lite3_map_amcl_debug.launch
```

Reason: those launch files can publish TF, `robot_description`, joint states,
map/amcl, or `robot_state_publisher`. Duplicating them from the developer
station while the robot main stack is running can create duplicate frames,
duplicate node names, or conflicting `/use_sim_time`.

## comp2026_ws Navigation RViz

This developer-station RViz entry is for the national-task navigation/debug stack in `comp2026_ws`.
The robot should run the ROS master and the headless navigation/task launch. RViz runs in the VM.

Typical VM environment, using the AX3600 debug LAN after the robot IP is confirmed:

```bash
export ROS_MASTER_URI=http://<robot_ip>:11311
export ROS_IP=<vm_ip>
```

Current planned AX3600 addresses:

```text
router:     192.168.31.1
host PC:    192.168.31.10
VM:         192.168.31.11
robot:      TBD after field test
arm RDK X5: TBD after field test
```

Check connectivity from the VM:

```bash
rostopic list
rostopic hz /scan
rostopic hz /amcl_pose
rostopic echo -n 1 /move_base/status
```

Start remote RViz:

```bash
cd ~/dog_dev_station/rviz_ws
source devel/setup.bash
roslaunch dog_dev_rviz dog_comp2026_nav_remote_rviz.launch
```

The RViz profile is at:

```text
rviz_ws/src/dog_dev_rviz/config/comp2026_nav_debug.rviz
```

This launch only starts RViz. It must not start camera, AMCL, move_base, task nodes, `robot_state_publisher`, or TF publishers on the VM.
Keep model-check launches on the robot or in a separate offline workspace unless explicitly debugging URDF/bag data.

## comp2026_ws Lite3 Model RViz

`lite3_model_check.launch` from `comp2026_ws` was split for developer-station use because the original launch does more than open RViz:

```text
- loads robot_description from Lite3 URDF
- starts joint_state_publisher_gui or joint_state_publisher
- starts robot_state_publisher
- publishes static base_link -> TORSO TF
- starts RViz
```

For a live robot session, use the display-only remote RViz entry. It will not publish TF or robot model data from the VM:

```bash
cd ~/dog_dev_station/rviz_ws
source devel/setup.bash
roslaunch dog_dev_rviz dog_comp2026_lite3_model_remote_rviz.launch
```

If `/robot_description` and model TF are not published by the robot stack, the RobotModel display may stay empty. That is expected for the safe remote mode.

For offline URDF/model checking on the VM, use the offline launch. This starts the local copied `dog_dev_lite3_description` package, joint state publisher, robot state publisher, static TF, and RViz:

```bash
cd ~/dog_dev_station/rviz_ws
source devel/setup.bash
roslaunch dog_dev_rviz dog_comp2026_lite3_model_offline_check.launch
```

Do not run the offline model launch against the live robot main ROS master unless duplicate `robot_description` and TF publishers are intentional.

Copied model package, renamed to avoid rospack conflicts with comp2026_ws:

```text
rviz_ws/src/dog_dev_lite3_description
```

Developer-station RViz profile:

```text
rviz_ws/src/dog_dev_rviz/config/comp2026_lite3_model_check.rviz
```
