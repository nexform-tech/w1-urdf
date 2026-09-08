# W1 轮式移动机器人 — URDF 描述

[English](README.md) | **简体中文**

> W1 轮式移动机器人的 URDF 描述包。ROS 包名与模型名均为 **`yudao_urdf`**
> （本仓库中也以 *yudao* 指代该平台）。

本仓库提供了 W1 机器人平台的描述文件（URDF）、三维网格、launch 文件和控制器配置，
可用于 RViz 可视化、Gazebo 仿真以及运动规划 / 控制相关的研究开发。

## 概述

W1 是一款带升降躯干与双臂的轮式移动机器人：

- **轮式移动底盘**（`XC_link`）——机器人底部的移动底座。
- **三段升降柱**（`LZ2_Link` → `LZ3_Link` → `LZ4_Link`）——三个嵌套的移动（prismatic）
  关节，每个行程为 `0 … 0.314 m`，负责躯干的垂直升降。
- **腰部 / 肩部连接平台**（`YBZC_Link`）——由回转关节（`YBZC_Joint`）驱动，实现上半身
  整体旋转（约 ±160° … 180°）。
- **双臂（各 7 自由度）**——一侧为 `j2`–`J8`（J2…J8），另一侧为对应的镜像手臂
  `j2y`–`j8y`，末端各带一个固定的连接器工具（`Ljq_Link` / `LJQy_Link`）。
- **头部 / 云台机构**（`TBYD2` → `tbyd3` → `tbyd4`）——躯干顶部用于搭载传感器
  （如相机）的三关节颈部 / 云台。

模型由 SolidWorks 通过 **SW2URDF Exporter v1.6.0** 导出，之后按 Z-up 世界坐标系进行了
适配（见 [坐标系约定](#坐标系约定z-up)）。

## 模型概要

| 项目                      | 数值                                          |
| ------------------------- | --------------------------------------------- |
| 模型 / 包名               | `yudao_urdf` / 机器人 `yudao_urdf_z_up`       |
| link 数量                 | 25                                            |
| joint 数量                | 24（3 个固定，3 个移动，18 个回转）            |
| 可驱动自由度              | 21                                            |
| 总质量（近似）            | ≈ 25 kg                                       |
| 轮式底盘                  | 有（`XC_link`）                               |
| 双臂                      | 2 × 7 自由度，镜像对称                         |
| 模型来源                  | SolidWorks → SW2URDF 导出 → 修改适配           |

## 坐标系约定（Z-up）

SolidWorks 导出的模型遵循 Y-up 约定。为符合机器人领域通用的坐标系约定（Z 轴**朝上**，
X 轴**向前**），本 URDF 在固定关节 `rot_fixed_joint` 中施加了一次绕 X 轴 +90° 的旋转，
将原始根 link `XC_link` 连接到新增加的根 link `base_link`：

```
base_link --(rot_fixed_joint, rpy = [π/2, 0, 0])--> XC_link --> LZ2 --> LZ3 --> LZ4 --> ...
```

转换过程中未修改任何其他数值。适配后，所有移动（prismatic）关节沿世界坐标 Z（垂直）
方向运动，机器人在 RViz / Gazebo 中呈直立姿态。

## 关节列表

| 关节                   | 类型      | 父 → 子 link           | 轴          | 限位（近似）         | 说明                       |
| ---------------------- | --------- | ---------------------- | ----------- | -------------------- | -------------------------- |
| `rot_fixed_joint`      | fixed     | base_link → XC_link    | —           | —                    | Z-up 坐标适配              |
| `LZ2_Joint`            | prismatic | XC_link → LZ2_Link     | (0, 1, 0)   | 0 … 0.314 m          | 升降第 1 级                |
| `LZ3_Joint`            | prismatic | LZ2_Link → LZ3_Link    | (0, 1, 0)   | 0 … 0.314 m          | 升降第 2 级                |
| `LZ4_Joint`            | prismatic | LZ3_Link → LZ4_Link    | (0, 1, 0)   | 0 … 0.314 m          | 升降第 3 级                |
| `YBZC_Joint`           | revolute  | LZ4_Link → YBZC_Link   | (0,-1, 0)   | -2.79 … 3.14 rad     | 腰部 / 躯干旋转             |
| `j2_Joint` … `J8_Joint`| revolute, 7× | YBZC → j2 → … → J8  | —           | 每关节 ±2.79 rad     | 一侧机械臂（J2 … J8）      |
| `ee_Ljq_Joint`         | fixed     | J8_Link → Ljq_Link     | —           | —                    | 该侧末端工具               |
| `j2y_Joint` … `j8y_Joint` | revolute, 7× | YBZC → j2y → … → j8y | —       | 每关节 ±2.79 rad     | 另一侧机械臂（镜像）        |
| `ee_LJQy_Joint`        | fixed     | j8y_Link → LJQy_Link   | —           | —                    | 该侧末端工具               |
| `TBYD2_Joint`          | revolute  | YBZC_Link → TBYD2_Link | (0, 1, 0)  | ±1.57 rad            | 云台 — 偏航                |
| `tbyd3_Joint`          | revolute  | TBYD2 → tbyd3_Link     | (0, 0,-1)   | ±0.70 rad            | 云台 — 俯仰                |
| `tbyd4_Joint`          | revolute  | tbyd3 → tbyd4_Link     | (-1, 0, 0)  | ±0.70 rad            | 云台 — 横滚                |

> 说明：移动关节的限位单位为米；上表中的数值对应 Z-up 适配后的模型。
> limit 中的 effort / velocity（`100` / `1`）为导出时的默认占位值。

面向控制器的可驱动关节名称清单见 [config/joint_names.yaml](config/joint_names.yaml)。

## 目录结构

```
w1-urdf/
├── CMakeLists.txt                 # catkin 构建 / 安装规则
├── package.xml                    # ROS 包描述（yudao_urdf）
├── config/
│   └── joint_names.yaml           # 控制器使用的关节名称顺序列表
├── launch/
│   ├── display.launch             # RViz 可视化
│   └── gazebo.launch              # Gazebo 仿真
├── meshes/
│   └── *.STL                      # 各 link 的三维网格
└── urdf/
    ├── yudao.urdf                 # 主 URDF（Z-up 适配版）
    └── yudao.csv                  # SolidWorks 导出数据（惯量 / 关节表）
```

## 环境要求

- Ubuntu 20.04 + **ROS 1（Noetic）**，使用 `catkin` 构建
- `robot_state_publisher`
- `rviz`
- `joint_state_publisher_gui`
- `gazebo` / `gazebo_ros`

如缺少依赖，可执行：

```bash
sudo apt install ros-noetic-robot-state-publisher ros-noetic-rviz \
                 ros-noetic-joint-state-publisher-gui ros-noetic-gazebo-ros
```

## 使用方法

### 1. 构建软件包

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone git@github.com:nexform-tech/w1-urdf.git
cd ~/catkin_ws && catkin_make
source ~/catkin_ws/devel/setup.bash
```

> ROS 通过 `package.xml` 中的 `<name>` 字段定位软件包，目录名无需与包名一致，
> 但建议以包名命名目录。

### 2. 在 RViz 中查看模型

```bash
roslaunch yudao_urdf display.launch
```

### 3. 在 Gazebo 中仿真

```bash
roslaunch yudao_urdf gazebo.launch
```

配置 `joint_state_controller` 或真机驱动时，请使用
[config/joint_names.yaml](config/joint_names.yaml) 中的关节列表。

## 已知问题

- `launch/gazebo.launch` 中引用的还是 `urdf/yudao_urdf.urdf`，而实际文件名为
  `urdf/yudao.urdf`；若 spawn 步骤失败，请更新该路径。
- `launch/display.launch` 使用了 `-d $(find yudao_urdf)/urdf.rviz`，但仓库中并未附带
  `urdf.rviz` 文件，RViz 将以默认配置打开。

## 许可证

[BSD](package.xml)（详见 `package.xml`）。

---

由 [nexform-tech](https://github.com/nexform-tech) 维护，欢迎贡献。