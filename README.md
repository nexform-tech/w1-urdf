# W1 Wheeled Mobile Robot — Official URDF Description

**English** | [简体中文](README.zh-CN.md)

| Document revision | 1.0                         |
| ----------------- | --------------------------- |
| Release date      | 2026-09-08                  |
| Product           | W1 Wheeled Mobile Robot     |
| Package name      | `yudao_urdf`                |
| Issued by         | nexform-tech                |

This document is the official URDF (Unified Robot Description Format) description of the
**W1 Wheeled Mobile Robot**. It is prepared and maintained by nexform-tech for customers who
integrate, simulate, or plan the control of the W1 platform.

---

## Table of contents

1. [Product introduction](#1-product-introduction)
2. [Technical specifications](#2-technical-specifications)
3. [Kinematic architecture](#3-kinematic-architecture)
4. [Coordinate system](#4-coordinate-system)
5. [Joint nomenclature](#5-joint-nomenclature)
6. [Repository structure](#6-repository-structure)
7. [System requirements](#7-system-requirements)
8. [Installation and usage](#8-installation-and-usage)
9. [Notes and limitations](#9-notes-and-limitations)
10. [Technical support](#10-technical-support)
11. [License and disclaimer](#11-license-and-disclaimer)

---

## 1. Product introduction

The W1 is a wheeled mobile robot with a vertically lifting torso and dual manipulator arms.
This repository provides the complete robot description required for simulation and control
development, including:

- The URDF model of the mechanism;
- Three-dimensional mesh files (STL) of all links;
- Launch files for visualization (RViz) and physics simulation (Gazebo);
- Actuated joint list for controller configuration.

The model is derived from the SolidWorks design data of the W1 platform via the SW2URDF
Exporter (v1.6.0), and has been adapted to a Z-up world frame as described in
[Section 4](#4-coordinate-system).

## 2. Technical specifications

| Item                          | Value                                   |
| ----------------------------- | --------------------------------------- |
| Model / package name          | `yudao_urdf` / robot `yudao_urdf_z_up`  |
| Number of links               | 25                                      |
| Number of joints              | 24 (3 fixed, 3 prismatic, 18 revolute)  |
| Actuated degrees of freedom   | 21                                      |
| Approximate total mass        | ≈ 25 kg                                |
| Mobile base                   | Wheeled chassis (`XC_link`)             |
| Lifting column                | 3-stage prismatic, stroke 0 … 0.314 m per stage |
| Manipulator arms              | 2 × 7-DOF, mirror-symmetric             |
| Units                         | length: m, mass: kg, angle: rad, inertia: kg·m² |

All values above are taken from the design data included in this package. They are
**reference values and are not a substitute for the specifications of the shipped product**.

## 3. Kinematic architecture

The mechanism consists of the following modules in series from the base to the tip:

| Module              | Associated links                              | Description                                        |
| ------------------- | --------------------------------------------- | -------------------------------------------------- |
| Mobile base         | `XC_link`                                     | Wheeled chassis, the fixed root of the mechanism.  |
| Lifting column      | `LZ2_Link`, `LZ3_Link`, `LZ4_Link`            | Three nested prismatic joints; vertical translation of the torso. |
| Waist / torso       | `YBZC_Link`                                   | Revolute joint rotating the entire upper body.     |
| Arm (side A)        | `j2_Link` … `J8_Link`, `Ljq_Link`             | 7-DOF manipulator with a fixed end-effector connector. |
| Arm (side B)        | `j2y_Link` … `j8y_Link`, `LJQy_Link`          | 7-DOF manipulator, mirror-symmetric to side A, with a fixed end-effector connector. |
| Head / gimbal       | `TBYD2_Link`, `tbyd3_Link`, `tbyd4_Link`      | 3-joint gimbal for sensor mounting (e.g. cameras). |

## 4. Coordinate system

The W1 URDF follows the standard robotics frame convention, and units are expressed in the
International System of Units (SI):

- **Z axis**: pointing **upward**;
- **X axis**: pointing **forward**;
- Rotation: right-hand rule, unit radian.

The SolidWorks export follows a Y-up convention. To comply with the Z-up convention above,
the URDF inserts a new root link `base_link` and applies a single +90° rotation about the
X-axis inside the fixed joint `rot_fixed_joint`:

```text
base_link --(rot_fixed_joint, rpy = [π/2, 0, 0])--> XC_link --> LZ2_Link --> LZ3_Link --> LZ4_Link --> ...
```

No other numeric values were modified during this conversion. As a result, within the frame
of `base_link`, all prismatic joints translate along the world Z (vertical) axis and the
robot stands upright in RViz and Gazebo.

`base_link` is the frame in which the robot is defined; its origin does not correspond to a
physical mounting point. Refer to the shipped mechanical drawings for mounting dimensions.

## 5. Joint nomenclature

Joint names are prefixed by the module they belong to.

| Joint                    | Type      | Parent → Child            | Axis       | Limit (approx.)     | Description             |
| ------------------------ | --------- | ------------------------- | ---------- | ------------------- | ----------------------- |
| `rot_fixed_joint`        | fixed     | base_link → XC_link       | —          | —                   | Z-up frame adaptation   |
| `LZ2_Joint`              | prismatic | XC_link → LZ2_Link        | (0, 1, 0)  | 0 … 0.314 m         | Lifting column, stage 1 |
| `LZ3_Joint`              | prismatic | LZ2_Link → LZ3_Link       | (0, 1, 0)  | 0 … 0.314 m         | Lifting column, stage 2 |
| `LZ4_Joint`              | prismatic | LZ3_Link → LZ4_Link       | (0, 1, 0)  | 0 … 0.314 m         | Lifting column, stage 3 |
| `YBZC_Joint`             | revolute  | LZ4_Link → YBZC_Link      | (0,-1, 0)  | -2.79 … 3.14 rad    | Waist / torso rotation  |
| `j2_Joint` … `J8_Joint`  | revolute, ×7 | YBZC_Link → j2_Link → … → J8_Link | — | ±2.79 rad each | Arm, side A (J2 … J8) |
| `ee_Ljq_Joint`           | fixed     | J8_Link → Ljq_Link        | —          | —                   | End-effector, side A    |
| `j2y_Joint` … `j8y_Joint`| revolute, ×7 | YBZC_Link → j2y_Link → … → j8y_Link | — | ±2.79 rad each | Arm, side B (mirrored) |
| `ee_LJQy_Joint`          | fixed     | j8y_Link → LJQy_Link      | —          | —                   | End-effector, side B    |
| `TBYD2_Joint`            | revolute  | YBZC_Link → TBYD2_Link    | (0, 1, 0)  | ±1.57 rad           | Gimbal — yaw            |
| `tbyd3_Joint`            | revolute  | TBYD2_Link → tbyd3_Link   | (0, 0,-1)  | ±0.70 rad           | Gimbal — tilt           |
| `tbyd4_Joint`            | revolute  | tbyd3_Link → tbyd4_Link   | (-1, 0, 0) | ±0.70 rad           | Gimbal — roll           |

Notes:

- Prismatic joint limits are expressed in meters.
- The effort / velocity limits (`100` / `1`) included in the URDF are exporter placeholders
  and **must be replaced with the motor controller ratings of the actual system** before use
  in trajectory planning.
- A controller-ready list of actuated joint names is provided in
  [config/joint_names.yaml](config/joint_names.yaml).

## 6. Repository structure

```text
w1-urdf/
├── CMakeLists.txt                  # catkin build / install rules
├── package.xml                     # ROS package manifest (yudao_urdf)
├── config/
│   └── joint_names.yaml            # ordered list of actuated joints for controllers
├── launch/
│   ├── display.launch              # RViz visualization
│   └── gazebo.launch               # Gazebo simulation
├── meshes/
│   └── *.STL                       # mesh files of all links
├── rviz/
│   └── display.rviz                # RViz configuration (fixed frame: base_link)
└── urdf/
    ├── yudao.urdf                  # main URDF (Z-up, adapted)
    └── yudao.csv                   # SolidWorks export data (inertial / joint table)
```

## 7. System requirements

- Ubuntu 20.04 with **ROS 1 (Noetic)**, built with `catkin`;
- `robot_state_publisher`;
- `rviz`;
- `joint_state_publisher_gui`;
- `gazebo` / `gazebo_ros`.

Install the dependencies if they are not present:

```bash
sudo apt install ros-noetic-robot-state-publisher ros-noetic-rviz \
                 ros-noetic-joint-state-publisher-gui ros-noetic-gazebo-ros
```

## 8. Installation and usage

### 8.1 Build the package

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone git@github.com:nexform-tech/w1-urdf.git
cd ~/catkin_ws && catkin_make
source ~/catkin_ws/devel/setup.bash
```

### 8.2 Visualization in RViz

```bash
roslaunch yudao_urdf display.launch
```

This loads the model into the parameter server and opens RViz with the shipped configuration
in [rviz/display.rviz](rviz/display.rviz). Use the *Joint State Publisher* GUI panel to move
individual joints.

### 8.3 Physics simulation in Gazebo

```bash
roslaunch yudao_urdf gazebo.launch
```

### 8.4 Controller configuration

Use the joint list in [config/joint_names.yaml](config/joint_names.yaml) when configuring
`joint_state_controller` or the drivers of the physical robot.

## 9. Notes and limitations

- The inertial parameters and joint limits in this package are derived from the design data
  and are reference values only. Final values are subject to the shipped product.
- The mesh files are provided for visualization and collision checking. They are not
  intended for manufacturing.
- Mirror symmetry between the two arms (side A `j2`…`j8` and side B `j2y`…`j8y`) is achieved
  within the URDF by definition of the mirrored branch; assignment of "left / right" is left
  to the integrator.
- The gimbal module (`TBYD2_Link`, `tbyd3_Link`, `tbyd4_Link`) is provided for mounting
  optional sensors; the sensor payload is not part of the robot description.
- Keep effort and velocity limits within the physical controller ratings (see
  [Section 5](#5-joint-nomenclature)).

## 10. Technical support

For questions about this package, please open an issue in this repository or contact
nexform-tech through GitHub ([nexform-tech](https://github.com/nexform-tech)).

## 11. License and disclaimer

- The package is licensed under the **BSD** terms, as specified in `package.xml`.
- This document is provided "as is" without warranty of any kind. nexform-tech disclaims all
  liability for damages arising from the use of the information contained herein.
- nexform-tech reserves the right to revise this document at any time without prior notice.
