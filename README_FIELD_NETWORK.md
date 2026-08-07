# IP 地址变化处理

虚拟机通过 `ros_robot`连接机器狗 ROS Master，机器狗通过 `ros_master`读取本机 IP。更换 WiFi 或手机热点后，可使用临时传参或修改默认值两种方案。

## 方案一：临时传入新 IP

该方案不修改 `~/.bashrc`，适合比赛现场或临时网络。

先查看当前 IP：

```bash
# 机器狗端
ip -br -4 addr show wlan0

# 虚拟机端
ip -br -4 addr show ens33
```

假设新网络中的机器狗 IP 为 `192.168.43.120`，在虚拟机的每个新终端执行：

```bash
ros_robot 192.168.43.120
```

`ros_robot` 会自动读取虚拟机当前 IP。如果自动读取错误，可同时指定机器狗和虚拟机 IP：

```bash
ros_robot 192.168.43.120 192.168.43.135
```

在机器狗的每个新终端执行：

```bash
ros_master
```

`ros_master` 默认自动读取 `wlan0` 的 IP。如果开发 WiFi 改用其他网卡，传入新网卡名：

```bash
ros_master wlan1
```

也可手动指定网卡和机器狗 IP：

```bash
ros_master wlan0 192.168.43.120
```

## 方案二：修改默认配置

该方案适合长期使用同一网络，修改后可在虚拟机中直接执行 `ros_robot`。

在虚拟机编辑：

```bash
nano ~/.bashrc
```

在 `ros_robot()` 函数中找到：

```bash
local robot_ip="${1:-192.168.31.174}"
```

将其中的默认地址替换为新的机器狗 IP，例如：

```bash
local robot_ip="${1:-192.168.43.120}"
```

如果机器狗长期改用其他网卡，在机器狗 `~/.bashrc` 的 `ros_master()` 函数中找到：

```bash
local interface_name="${1:-wlan0}"
```

将 `wlan0` 替换为新的默认网卡名。

修改完成后，在已打开的终端重新加载：

```bash
source ~/.bashrc
```

新打开的 Bash 终端会自动加载函数定义，但不会自动执行函数。每个新终端仍需执行：

```bash
# 机器狗端
ros_master

# 虚拟机端
ros_robot
```

## IP 变化后的注意事项

- 修改函数只会影响当前 shell 和之后启动的进程。
- 已经运行的 ROS Master、节点和 RViz 不会自动更新 IP。
- IP 变化后，应停止现有 ROS 程序，重新调用 `ros_master` 和 `ros_robot`，然后重启 ROS 链路。
- 重启前建议确认机器狗和虚拟机可以双向 `ping`。
