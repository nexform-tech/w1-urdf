# W1 轮式移动机器人 — 官方 URDF 描述文件

[English](README.md) | **简体中文**

| 文档版本           | 1.0                          |
| ------------------ | ---------------------------- |
| 发布日期           | 2026-09-08                   |
| 产品型号           | W1 轮式移动机器人             |
| 软件包名称         | `yudao_urdf`                 |
| 编制单位           | nexform-tech                 |

本文档为 **W1 轮式移动机器人**的官方 URDF（统一机器人描述格式，Unified Robot Description
Format）描述文件说明。本文档由 nexform-tech 编制并维护，供需要对本平台进行集成、仿真或
运动控制开发的相关客户使用。

---

## 目录

1. [产品说明](#1-产品说明)
2. [技术规格](#2-技术规格)
3. [机构组成](#3-机构组成)
4. [坐标系定义](#4-坐标系定义)
5. [关节命名与参数](#5-关节命名与参数)
6. [仓库文件结构](#6-仓库文件结构)
7. [系统要求](#7-系统要求)
8. [安装与使用](#8-安装与使用)
9. [注意事项与限制](#9-注意事项与限制)
10. [技术支持](#10-技术支持)
11. [许可与免责声明](#11-许可与免责声明)

---

## 1. 产品说明

W1 是一款配备垂直升降躯干与双臂的轮式移动机器人。本仓库提供仿真与控制开发所需的完整
机器人描述资源，包括：

- 机构 URDF 模型；
- 全部 link 的三维网格文件（STL）；
- 可视化（RViz）与物理仿真（Gazebo）的 launch 文件；
- 面向控制器配置的可驱动关节列表。

本模型基于 W1 平台的 SolidWorks 设计数据，通过 SW2URDF Exporter（v1.6.0）导出，并按
[第 4 节](#4-坐标系定义)所述适配为 Z-up 世界坐标系。

## 2. 技术规格

| 项目                       | 数值                                       |
| -------------------------- | ------------------------------------------ |
| 模型 / 软件包名称          | `yudao_urdf` / 机器人 `yudao_urdf_z_up`    |
| link（连杆）数量           | 25                                         |
| joint（关节）数量          | 24（3 个固定、3 个移动、18 个回转）          |
| 可驱动自由度               | 21                                         |
| 整机质量（近似值）         | ≈ 25 kg                                   |
| 移动底座                   | 轮式底盘（`XC_link`）                       |
| 升降立柱                   | 3 级平移结构，每级行程 0 … 0.314 m           |
| 机械臂                     | 2 × 7 自由度，镜像对称                        |
| 单位约定                   | 长度：m；质量：kg；角度：rad；惯量：kg·m²    |

上表数值均取自本包所包含的设计数据，仅作**参考值**使用，不作为交付产品的结构规格依据。

## 3. 机构组成

自基座至末端，本机构由以下模块串联组成：

| 模块         | 相关 link                                   | 说明                                       |
| ------------ | ------------------------------------------- | ------------------------------------------ |
| 移动底座     | `XC_link`                                   | 轮式底盘，机构的固定根节点。                |
| 升降立柱     | `LZ2_Link`、`LZ3_Link`、`LZ4_Link`          | 三个嵌套平移关节，实现躯干的垂直升降。      |
| 腰部 / 躯干  | `YBZC_Link`                                 | 回转关节，带动整个上半身旋转。              |
| 机械臂 A 侧  | `j2_Link` … `J8_Link`、`Ljq_Link`           | 7 自由度机械臂，末端带固定连接器。          |
| 机械臂 B 侧  | `j2y_Link` … `j8y_Link`、`LJQy_Link`        | 7 自由度机械臂，与 A 侧镜像对称，末端带固定连接器。 |
| 头部 / 云台  | `TBYD2_Link`、`tbyd3_Link`、`tbyd4_Link`    | 用于搭载传感器（如相机）的 3 关节云台。     |

## 4. 坐标系定义

W1 的 URDF 遵循通用的机器人坐标系约定，全部物理量采用国际单位制（SI）：

- **Z 轴**：竖直**向上**；
- **X 轴**：指向前方；
- 旋转方向遵循右手定则，单位为弧度（rad）。

SolidWorks 导出的原始模型遵循 Y-up 约定。为符合上述 Z-up 约定，本 URDF 新增根 link
`base_link`，并在固定关节 `rot_fixed_joint` 内施加一次性绕 X 轴 +90° 的旋转：

```text
base_link --(rot_fixed_joint, rpy = [π/2, 0, 0])--> XC_link --> LZ2_Link --> LZ3_Link --> LZ4_Link --> ...
```

本次转换未修改其他任何数值。适配完成后，在 `base_link` 坐标系下，所有移动关节均沿世界
坐标 Z（垂直）方向运动，机器人在 RViz 与 Gazebo 中呈直立姿态。

注意：`base_link` 为机器人定义的基准坐标系，其原点不代表实物上的某个安装点；安装尺寸请
以交付的机械图纸为准。

## 5. 关节命名与参数

关节名称以其所属模块的代号为前缀。

| 关节                       | 类型       | 父 → 子 link                        | 轴         | 限位（近似）      | 说明                  |
| -------------------------- | ---------- | ----------------------------------- | ---------- | ----------------- | --------------------- |
| `rot_fixed_joint`          | fixed      | base_link → XC_link                 | —          | —                 | Z-up 坐标适配         |
| `LZ2_Joint`                | prismatic  | XC_link → LZ2_Link                  | (0, 1, 0)  | 0 … 0.314 m       | 升降立柱第 1 级        |
| `LZ3_Joint`                | prismatic  | LZ2_Link → LZ3_Link                 | (0, 1, 0)  | 0 … 0.314 m       | 升降立柱第 2 级        |
| `LZ4_Joint`                | prismatic  | LZ3_Link → LZ4_Link                 | (0, 1, 0)  | 0 … 0.314 m       | 升降立柱第 3 级        |
| `YBZC_Joint`               | revolute   | LZ4_Link → YBZC_Link                | (0,-1, 0)  | -2.79 … 3.14 rad  | 腰部 / 躯干旋转        |
| `j2_Joint` … `J8_Joint`    | revolute，×7 | YBZC_Link → j2_Link → … → J8_Link | —          | 每关节 ±2.79 rad  | 机械臂 A 侧（J2 … J8） |
| `ee_Ljq_Joint`             | fixed      | J8_Link → Ljq_Link                  | —          | —                 | 末端连接器，A 侧       |
| `j2y_Joint` … `j8y_Joint`  | revolute，×7 | YBZC_Link → j2y_Link → … → j8y_Link | —       | 每关节 ±2.79 rad  | 机械臂 B 侧（镜像）    |
| `ee_LJQy_Joint`            | fixed      | j8y_Link → LJQy_Link                 | —          | —                 | 末端连接器，B 侧       |
| `TBYD2_Joint`              | revolute   | YBZC_Link → TBYD2_Link               | (0, 1, 0)  | ±1.57 rad         | 云台 — 偏航            |
| `tbyd3_Joint`              | revolute   | TBYD2_Link → tbyd3_Link              | (0, 0,-1)  | ±0.70 rad         | 云台 — 俯仰            |
| `tbyd4_Joint`              | revolute   | tbyd3_Link → tbyd4_Link              | (-1, 0, 0) | ±0.70 rad         | 云台 — 横滚            |

说明：

- 移动（prismatic）关节的限位单位为米。
- URDF 中随附的力 / 速度限位（`100` / `1`）为导出器生成的占位值，**在用于轨迹规划前
  必须替换为实际系统的电机控制器额定参数**。
- 面向控制器的可驱动关节名称清单见 [config/joint_names.yaml](config/joint_names.yaml)。

## 6. 仓库文件结构

```text
w1-urdf/
├── CMakeLists.txt                  # catkin 构建 / 安装规则
├── package.xml                     # ROS 包描述（yudao_urdf）
├── config/
│   └── joint_names.yaml            # 控制器使用的关节名称顺序列表
├── launch/
│   ├── display.launch              # RViz 可视化
│   └── gazebo.launch               # Gazebo 仿真
├── meshes/
│   └── *.STL                       # 各 link 的三维网格
├── rviz/
│   └── display.rviz                # RViz 配置（固定坐标系：base_link）
└── urdf/
    ├── yudao.urdf                  # 主 URDF 文件（Z-up 适配版）
    └── yudao.csv                   # SolidWorks 导出数据（惯量 / 关节表）
```

## 7. 系统要求

- Ubuntu 20.04 + **ROS 1（Noetic）**，使用 `catkin` 构建；
- `robot_state_publisher`；
- `rviz`；
- `joint_state_publisher_gui`；
- `gazebo` / `gazebo_ros`。

若依赖缺失，请执行：

```bash
sudo apt install ros-noetic-robot-state-publisher ros-noetic-rviz \
                 ros-noetic-joint-state-publisher-gui ros-noetic-gazebo-ros
```

## 8. 安装与使用

### 8.1 构建软件包

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone git@github.com:nexform-tech/w1-urdf.git
cd ~/catkin_ws && catkin_make
source ~/catkin_ws/devel/setup.bash
```

### 8.2 RViz 可视化

```bash
roslaunch yudao_urdf display.launch
```

该命令将模型加载至参数服务器，并使用 [rviz/display.rviz](rviz/display.rviz) 中的官方
配置打开 RViz。可通过 *Joint State Publisher* 图形面板对各个关节进行位姿调节。

### 8.3 Gazebo 物理仿真

```bash
roslaunch yudao_urdf gazebo.launch
```

### 8.4 控制器配置

配置 `joint_state_controller` 或真机驱动时，请使用
[config/joint_names.yaml](config/joint_names.yaml) 中的关节列表。

## 9. 注意事项与限制

- 本包中的惯量参数与关节限位源自设计数据，仅作参考值；最终数值以交付产品为准。
- 网格文件仅用于可视化与碰撞检测，不适用于生产制造。
- 双臂之间（A 侧 `j2`…`j8` 与 B 侧 `j2y`…`j8y`）的镜像对称关系由 URDF 中的镜像支链
  定义实现；“左 / 右”的分配由集成方自行确定。
- 云台模块（`TBYD2_Link`、`tbyd3_Link`、`tbyd4_Link`）用于搭载选配传感器，传感器载荷
  不属于本机器人描述文件的范畴。
- 力与速度限位应保持在物理控制器的额定范围内（见[第 5 节](#5-关节命名与参数)）。

## 10. 技术支持

如对本软件包有任何疑问，欢迎在本仓库中提交 Issue，或通过 GitHub 联系
nexform-tech（[nexform-tech](https://github.com/nexform-tech)）。

## 11. 许可与免责声明

- 本软件包依据 `package.xml` 中所声明的 **BSD** 条款授权使用。
- 本文档按“原样”（as is）提供，不作任何形式的担保。nexform-tech 不承担因使用本文档所
  载信息而产生的任何损失责任。
- nexform-tech 保留不经事先通知随时修订本文档的权利。
