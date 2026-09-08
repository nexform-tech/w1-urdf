# W1 Wheeled Mobile Robot — URDF Description

**English** | [简体中文](README.zh-CN.md)

> URDF description package for the **W1 wheeled mobile robot**. The ROS package and model are
> named **`yudao_urdf`** (the platform is also referred to as *yudao* in this repository).

This repository provides the robot description (URDF), 3D meshes, launch files and control
configuration for the W1 robotic platform, ready for visualization in RViz, simulation in
Gazebo, and motion-planning / control research.

## Overview

The W1 is a wheeled mobile robot with a lifting torso and dual arms:

- **Wheeled mobile base** (`XC_link`) — the fixed-wheel chassis at the bottom of the robot.
- **3-DOF lifting column** (`LZ2_Link` → `LZ3_Link` → `LZ4_Link`) — three nested prismatic
  joints (each with a stroke of `0 … 0.314 m`) that translate the torso vertically.
- **Waist / shoulder mount** (`YBZC_Link`) — a revolute joint (`YBZC_Joint`) that rotates the
  upper body (±160° … 180°).
- **Dual 7-DOF arms** — `j2`–`J8` (J2…J8) on one side and the mirrored `j2y`–`j8y` on the
  other, each ending in a fixed connector tool (`Ljq_Link` / `LJQy_Link`).
- **Head / gimbal assembly** (`TBYD2` → `tbyd3` → `tbyd4`) — a 3-joint neck/gimbal on top of
  the torso for mounting sensors (e.g. cameras).

The model was exported from SolidWorks using the **SW2URDF Exporter v1.6.0** and afterwards
adapted for a Z-up world frame (see [Coordinate Frames](#coordinate-frames-z-up)).

## Model summary

| Item                        | Value                                        |
| --------------------------- | -------------------------------------------- |
| Model / package name        | `yudao_urdf` / robot `yudao_urdf_z_up`       |
| Number of links             | 25                                           |
| Number of joints            | 24 (3 fixed, 3 prismatic, 18 revolute)       |
| Actuated DOF                | 21                                           |
| Approximate total mass      | ≈ 25 kg                                      |
| Wheeled chassis             | yes (`XC_link`)                              |
| Dual arms                   | 2 × 7-DOF, mirrored                          |
| Origin file                 | SolidWorks → SW2URDF exporter → modified     |

## Coordinate frames (Z-up)

The SolidWorks export produced a model in the Y-up convention. To match the standard
robotics convention (Z-axis pointing **up**, X-axis **forward**), the URDF applies a single
+90° rotation about the X-axis inside the fixed joint `rot_fixed_joint`, which attaches the
original root link `XC_link` to a new root `base_link`:

```
base_link --(rot_fixed_joint, rpy = [π/2, 0, 0])--> XC_link --> LZ2 --> LZ3 --> LZ4 --> ...
```

No other numeric values were changed during this conversion. In the resulting frame,
all prismatic joints translate along the world Z (vertical) axis, and the robot stands
upright in RViz/Gazebo.

## Joints

| Joint            | Type      | Parent → Child      | Axis        | Limit (approx.)       | Role                    |
| ---------------- | --------- | ------------------- | ----------- | --------------------- | ----------------------- |
| `rot_fixed_joint`| fixed     | base_link → XC_link | —           | —                     | Z-up frame fix          |
| `LZ2_Joint`      | prismatic | XC_link → LZ2_Link  | (0, 1, 0)   | 0 … 0.314 m           | lift stage 1            |
| `LZ3_Joint`      | prismatic | LZ2_Link → LZ3_Link | (0, 1, 0)   | 0 … 0.314 m           | lift stage 2            |
| `LZ4_Joint`      | prismatic | LZ3_Link → LZ4_Link | (0, 1, 0)   | 0 … 0.314 m           | lift stage 3            |
| `YBZC_Joint`     | revolute  | LZ4_Link → YBZC_Link| (0,-1, 0)   | -2.79 … 3.14 rad      | waist / torso rotation  |
| `j2_Joint` … `J8_Joint` | revolute, 7× | YBZC → j2 → … → J8 | —      | ±2.79 rad (per joint) | right arm (J2 … J8)     |
| `ee_Ljq_Joint`   | fixed     | J8_Link → Ljq_Link  | —           | —                     | right end-effector      |
| `j2y_Joint` … `j8y_Joint` | revolute, 7× | YBZC → j2y → … → j8y | —  | ±2.79 rad (per joint) | left arm (mirrored)     |
| `ee_LJQy_Joint`  | fixed     | j8y_Link → LJQy_Link| —           | —                     | left end-effector       |
| `TBYD2_Joint`    | revolute  | YBZC_Link → TBYD2_Link | (0, 1, 0) | ±1.57 rad          | gimbal — yaw            |
| `tbyd3_Joint`    | revolute  | TBYD2 → tbyd3_Link  | (0, 0,-1)   | ±0.70 rad             | gimbal — tilt           |
| `tbyd4_Joint`    | revolute  | tbyd3 → tbyd4_Link  | (-1, 0, 0)  | ±0.70 rad             | gimbal — roll           |

> Note: prismatic joint limits are given in meters; the values above are for the Z-up model.
> Limit effort/velocity (`100` / `1`) are placeholder values exported by default.

A controller-friendly list of actuated joint names is provided in
[config/joint_names.yaml](config/joint_names.yaml).

## Repository layout

```
w1-urdf/
├── CMakeLists.txt              # catkin build/install rules
├── package.xml                 # ROS package manifest (yudao_urdf)
├── config/
│   └── joint_names.yaml        # ordered list of actuated joints for controllers
├── launch/
│   ├── display.launch          # RViz visualization
│   └── gazebo.launch           # Gazebo simulation
├── meshes/
│   └── *.STL                   # mesh files of all links
└── urdf/
    ├── yudao.urdf              # main URDF (Z-up, adapted)
    └── yudao.csv               # SolidWorks export data (inertia / joint table)
```

## Requirements

- Ubuntu 20.04 + **ROS 1 (Noetic)**, built with `catkin`
- `robot_state_publisher`
- `rviz`
- `joint_state_publisher_gui`
- `gazebo` / `gazebo_ros`

Install them if missing:

```bash
sudo apt install ros-noetic-robot-state-publisher ros-noetic-rviz \
                 ros-noetic-joint-state-publisher-gui ros-noetic-gazebo-ros
```

## Usage

### 1. Build the package

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone git@github.com:nexform-tech/w1-urdf.git
cd ~/catkin_ws && catkin_make
source ~/catkin_ws/devel/setup.bash
```

> ROS locates the package through the `<name>` field in `package.xml`, so the
> directory name does not need to match — but naming it after the package is recommended.

### 2. View in RViz

```bash
roslaunch yudao_urdf display.launch
```

### 3. Simulate in Gazebo

```bash
roslaunch yudao_urdf gazebo.launch
```

Use the [config/joint_names.yaml](config/joint_names.yaml) list when configuring
`joint_state_controller` / drivers for the physical robot.

## Known issues

- `launch/gazebo.launch` still references `urdf/yudao_urdf.urdf`, while the actual file is
  `urdf/yudao.urdf` — update the path if the spawn step fails.
- `launch/display.launch` passes `-d $(find yudao_urdf)/urdf.rviz`, but no `urdf.rviz` file
  is shipped; RViz will open with default settings.

## License

[BSD](package.xml) (see `package.xml`).

---

Maintained by [nexform-tech](https://github.com/nexform-tech). Contributions welcome.
