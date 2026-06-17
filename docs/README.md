# Documentation Directory

Contains supplementary technical documentation for the Autonomous Mobile Robot Navigation project.

---

## Index

| Document | Description |
|----------|-------------|
| [parameter_tuning_guide.md](parameter_tuning_guide.md) | Detailed guide for tuning AMCL, TEB, costmap, and velocity smoother parameters |
| [multi_map_architecture.md](multi_map_architecture.md) | Technical deep-dive into the multi-map navigation architecture and coordinate transform derivation |
| [iso_compliance_summary.md](iso_compliance_summary.md) | Summary of ISO 3691-4:2020 compliance test results |
| [troubleshooting.md](troubleshooting.md) | Known issues and debugging guide |

---

## Parameter Tuning Guide

### AMCL Parameters

| Parameter | Value Used | Effect |
|-----------|-----------|--------|
| `min_particles` | 500 | Minimum particle count — too low causes localization loss |
| `max_particles` | 2000 | Maximum particles — higher is more accurate but uses more CPU |
| `kld_err` | 0.05 | KLD-sampling target error — lower = more particles on demand |
| `update_min_d` | 0.1 m | Motion threshold before update — prevents update spam |
| `laser_max_range` | 25.0 m | Clip LiDAR beyond this range to reduce noise weight |
| `recovery_alpha_slow` | 0.001 | Slow filter for kidnap detection |
| `recovery_alpha_fast` | 0.1 | Fast filter — ratio triggers random particle injection |

**Tuning Tip:** Start with `max_particles=5000` and reduce until CPU usage stabilizes below 60%. With KLD-sampling, 2000 particles achieves equivalent accuracy with ~60% less CPU vs. fixed 5000.

### TEB Local Planner Parameters

| Parameter | Value Used | Effect |
|-----------|-----------|--------|
| `max_vel_x` | 0.5 m/s | Maximum forward speed — reduce for narrow corridors |
| `max_vel_theta` | 0.8 rad/s | Maximum rotation rate |
| `min_obstacle_dist` | 0.30 m | Minimum distance to maintain from obstacles |
| `inflation_dist` | 0.6 m | Influence zone of obstacles on trajectory |
| `dt_ref` | 0.3 s | Reference time resolution of elastic band |
| `no_inner_iterations` | 5 | TEB optimization inner loop count |
| `no_outer_iterations` | 4 | TEB outer optimization iterations |
| `enable_homotopy_class_planning` | true | Enables exploration of multiple paths — crucial for corridors |

**Tuning Tip:** Increase `weight_kinematics_turning_radius` to 3.0 for smoother turns on a differential-drive robot. Reduce `max_vel_x` if the robot frequently overshoots goals.

### Velocity Smoother Parameters

| Parameter | Value Used | Effect |
|-----------|-----------|--------|
| `accel_lim_x` | 0.05 m/s² | Linear acceleration limit — prevents wheel slip |
| `decel_lim_x` | 0.05 m/s² | Linear deceleration limit |
| `accel_lim_w` | 0.3 rad/s² | Angular acceleration limit |
| `speed_lim_v` | 0.5 m/s | Maximum linear speed (redundant safety cap) |
| `speed_lim_w` | 0.8 rad/s | Maximum angular speed |

---

## Multi-Map Architecture

### Coordinate Transform Derivation

Given trigger polygon `P_trigger` in map01 and corresponding drop polygon `P_drop` in map02, the 2D rigid-body transform is computed as:

```
1. Compute centroids: c_trigger = mean(P_trigger), c_drop = mean(P_drop)
2. Compute oriented edges: e_trigger = P_trigger[1] - P_trigger[0], e_drop = P_drop[1] - P_drop[0]
3. Rotation angle: θ = atan2(e_drop.y, e_drop.x) - atan2(e_trigger.y, e_trigger.x)
4. Rotation matrix: R = [[cos θ, -sin θ], [sin θ, cos θ]]
5. Translation: t = c_drop - R · c_trigger
6. Transform: p_map02 = R · p_map01 + t
```

This closed-form solution requires only two polygon pairs and executes in microseconds, making it suitable for real-time re-localization.

### State Machine

```
IDLE → NAVIGATING → APPROACHING_TRANSITION → SWITCHING_MAP → RE-LOCALIZING → NAVIGATING
                                                                     ↓
                                                              (on failure)
                                                           RECOVERY → IDLE
```

---

## Troubleshooting

### "Could not transform the global plan to the frame of the controller"

**Cause:** TF buffer not populated immediately after map switch.  
**Fix:** Add `_wait_for_tf_to_settle(extra_sec=2.0)` in `map_manager.py` after AMCL re-initialization.

### AMCL particles not converging after switch

**Cause:** Robot stationary — AMCL only updates on motion.  
**Fix:** `map_manager` executes a small in-place rotation (±15°) to trigger AMCL particle re-sampling.

### Ghost obstacles persisting in costmap

**Cause:** `costmap_2d` obstacle layer `inf_is_valid` not propagating clearing rays for PointCloud2 inputs.  
**Workaround:** Periodic costmap clearing timer (`clear_costmaps` service every 5 seconds) in `map_manager.py`.

### High CPU usage from AMCL

**Fix:** Reduce `max_particles` to 2000, enable KLD-sampling (`kld_err: 0.05`, `kld_z: 0.99`).
