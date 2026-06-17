# Code Directory

This directory contains all ROS packages, configuration files, launch files, and scripts for the Autonomous Mobile Robot Navigation system.

---

## Package Overview

```text
code/
├── robot_navigation/       # Core navigation — move_base, AMCL, costmap configs
├── map_manager/            # Multi-map transition and re-localization node
├── velocity_smoother/      # Custom cmd_vel smoothing node
└── sensor_fusion/          # PointCloud processing and sensor pipeline
```

---

## navigation_stack

Core ROS navigation configuration for the Novus AMR 500.

### Key Files

| File | Purpose |
|------|---------|
| `launch/navigation_amcl.launch` | Main navigation launch file |
| `config/amcl_params.yaml` | AMCL localization parameters |
| `config/teb_local_planner_params.yaml` | TEB planner parameters |
| `config/global_costmap_params.yaml` | Global costmap configuration |
| `config/local_costmap_params.yaml` | Local costmap (dynamic window) config |
| `config/velocity_smoother_params.yaml` | Acceleration limit parameters |

### AMCL Key Parameters

```yaml
min_particles: 500
max_particles: 2000
kld_err: 0.05
update_min_d: 0.1       # m — update on motion
update_min_a: 0.2       # rad
resample_interval: 1
```

### TEB Key Parameters

```yaml
max_vel_x: 0.5          # m/s
max_vel_theta: 0.8      # rad/s
acc_lim_x: 0.3          # m/s²
min_obstacle_dist: 0.3  # m
inflation_dist: 0.6     # m
```

---

## map_manager

Topological multi-map navigation node.

### Architecture

The `map_manager.py` node:
1. Subscribes to `/amcl_pose` at 10 Hz
2. Checks if the robot's position falls inside any configured **trigger polygon** (quadrilateral in map frame)
3. On trigger detection:
   - Cancels the active `move_base` goal
   - Calls `/change_map` service to swap map server
   - Computes 2D rigid-body transform: `T = R·p_trigger + t` where R, t are derived from polygon correspondences
   - Publishes transformed pose to `/initialpose` for AMCL re-initialization
   - Executes small in-place rotation to promote AMCL particle convergence
   - Re-sends navigation goal in new map frame

### Configuration

```yaml
# map_manager_config.yaml
maps:
  - id: map01
    map_file: /path/to/map01.yaml
    trigger_polygon:
      - [38.0, -14.8]
      - [39.6, -14.9]
      - [39.6, -16.4]
      - [38.2, -16.3]
    target_map: map02
    drop_polygon:
      - [41.0, -14.8]
      - [42.6, -14.9]
      - [42.6, -16.4]
      - [41.2, -16.3]
```

---

## velocity_smoother

Custom ROS node for smooth velocity command generation.

### Operation

- Subscribes: `/cmd_vel_raw` (from TEB planner)
- Publishes: `/cmd_vel` (to robot base)
- Applies exponential ramp profiles:
  - Linear: `accel_lim = 0.05 m/s²`, `decel_lim = 0.05 m/s²`
  - Angular: `accel_lim = 0.3 rad/s²`, `decel_lim = 0.3 rad/s²`

---

## sensor_fusion

PointCloud processing pipeline for 3D obstacle perception.

### Pipeline

```
Velodyne VLP-16 → /velodyne_points (PointCloud2)
                 → pointcloud_to_laserscan → /scan (LaserScan)
                 → costmap obstacle_layer

RealSense D435i → /camera/depth/points (PointCloud2)
                 → voxel_grid_filter (leaf: 0.05m)
                 → /camera/filtered_points
                 → costmap obstacle_layer
```

---

## Mapping Via Gmapping


```bash
# Terminal 1: source devel/setup.bash
roslaunch robot_navigation mapping_2d_gmapping.launch

# Run robot manually, after completion of map to the the map Opne new terminal:
# Terminal 2: source devel/setup.bash
rosrun map_server map_saver -f my_map

```

---

## Launch Instructions

### Single-Map Navigation

```bash
# Terminal 1:
source devel/setup.bash
roslaunch robot_navigation navigation_amcl.launch

```

### Multi-Map Navigation

```bash
# For Multi-Map Navigation

# Terminal 1: Map manager
source devel/setup.bash
roslaunch robot_navigation multi_map_navigation.launch
```

---

## Dependencies

```bash
# ROS Noetic packages
sudo apt install \
  ros-noetic-navigation \
  ros-noetic-teb-local-planner \
  ros-noetic-amcl \
  ros-noetic-map-server \
  ros-noetic-move-base \
  ros-noetic-costmap-2d \
  ros-noetic-velodyne \
  ros-noetic-velodyne-pointcloud \
  ros-noetic-realsense2-camera \
  ros-noetic-realsense2-description \
  ros-noetic-pointcloud-to-laserscan \
  ros-noetic-robot-localization

# Python dependencies
pip3 install numpy scipy shapely
```

---
