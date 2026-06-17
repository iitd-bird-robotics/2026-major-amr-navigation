# Hardware Directory

Contains hardware specifications, sensor mounting details, and system integration information for the Novus AMR 500 platform.

---

## Robot Platform: Novus AMR 500

| Parameter          | Value                              |
|--------------------|------------------------------------|
| Type               | Differential-drive AMR             |
| Dimensions (L×W×H) | 1.17 m × 0.75 m × 0.425 m         |
| Payload Capacity   | 500 kg                             |
| Max Speed          | 1.5 m/s                            |
| Operating Speed    | 0.5 m/s (configured for safety)    |
| Battery            | 48V Li-ion, 8-hour runtime         |
| Communication      | ROS over Ethernet                  |

---

## Sensor Suite

### Velodyne VLP-16 (C16) LiDAR

| Parameter          | Value                         |
|--------------------|-------------------------------|
| Channels           | 16                            |
| Horizontal FOV     | 360°                          |
| Vertical FOV       | ±15° (30° total)              |
| Range              | 0.9 – 100 m                   |
| Range Accuracy     | ±3 cm                         |
| Rotation Rate      | 10 Hz                         |
| Points/Revolution  | ~300,000                      |
| Mount Height       | 0.425 m (robot top surface)   |
| Interface          | Ethernet (UDP), ROS driver: `velodyne_driver` |

### Intel RealSense D435i Depth Camera

| Parameter          | Value                         |
|--------------------|-------------------------------|
| Depth Technology   | Active Infrared Stereo        |
| Depth Range        | 0.11 – 10 m                   |
| Depth FOV (H×V)    | 87° × 58°                     |
| Depth Resolution   | 848 × 480 @ 30 fps            |
| RGB Resolution     | 1920 × 1080 @ 30 fps          |
| IMU                | BMI055 (6-DOF)                |
| Mount Position     | Front-face, 0.30 m height     |
| Interface          | USB 3.1, ROS driver: `realsense2_camera` |

### Onboard IMU

| Parameter          | Value                         |
|--------------------|-------------------------------|
| Type               | 6-DOF (3-axis accel + gyro)  |
| Update Rate        | 100 Hz                        |
| Integration        | `/imu/data` ROS topic         |

---

## Sensor Mounting

```
Top View (Novus AMR 500):

         ┌──────────────────────┐
         │                      │
         │   [VLP-16 LiDAR]    │  ← Center-top mount (0.425 m)
         │                      │
FRONT →  │  [RealSense D435i]  │  ← Front face mount (0.30 m)
         │                      │
         └──────────────────────┘
```

---

## Compute Specifications

| Component     | Specification              |
|---------------|----------------------------|
| CPU           | Intel Core i7 (8-core)     |
| RAM           | 16 GB DDR4                 |
| Storage       | 256 GB SSD                 |
| OS            | Ubuntu 20.04 LTS           |
| ROS           | Noetic (1.16.x)            |
| Network       | Gigabit Ethernet (to robot) |

---

## Wiring & Interface

- VLP-16 → Ethernet switch → Onboard PC
- RealSense D435i → USB 3.1 → Onboard PC
- Novus base controller → Ethernet → Onboard PC (ROS bridge)
- Emergency stop → Hardwired to Novus base motor controller

---

## Bill of Materials (Sensors Only)

| Item                          | Quantity | Notes                          |
|-------------------------------|----------|--------------------------------|
| Velodyne VLP-16 LiDAR         | 1        | Provided by Hi-Tech Robotic Systemz |
| Intel RealSense D435i         | 1        | Purchased separately           |
| Mounting bracket (VLP-16)     | 1        | Custom aluminum bracket        |
| Mounting bracket (RealSense)  | 1        | 3D-printed PLA bracket         |
| Ethernet switch (8-port)      | 1        | For multi-device connectivity  |

---

## Safety Features

- **Hardware E-Stop:** Physical button with <180 ms response time
- **Software E-Stop:** ROS topic `/emergency_stop` triggers immediate velocity zeroing
- **Obstacle Inflation:** 0.2 m inflation radius prevents collision with detected obstacles
- **Velocity Limits:** Maximum 0.5 m/s (software-enforced via `velocity_smoother`)
- **ISO Compliance:** System validated per ISO 3691-4:2020 driverless industrial truck standard
