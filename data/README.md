# Data Directory

Contains ISO performance test logs, navigation experiment data, and map files for the Autonomous Mobile Robot Navigation project.

---

## Structure

```text
data/
├── maps/                        # Pre-built occupancy grid maps
│   ├── map01.pgm                # Zone A occupancy grid
│   ├── map01.yaml               # Zone A map metadata
│   ├── map02.pgm                # Zone B occupancy grid
│   └── map02.yaml               # Zone B map metadata
├── iso_test_logs/               # Raw ISO 3691-4 test logs
│   ├── test01_nav_accuracy/     # TEST-01: Same-map navigation logs
│   ├── test02_map_switching/    # TEST-02: Cross-map switching logs
│   ├── test03_relocalization/   # TEST-03: Re-localization accuracy logs
│   └── test08_full_mission/     # TEST-08: End-to-end mission logs
├── navigation_logs/             # General navigation experiment logs
└── ISO_3691-4_Test_Report.md    # Full ISO compliance test report
```

---

## Maps

Two pre-built 2D occupancy grid maps created using GMapping SLAM:

| Map    | Zone   | Coordinate Frame Origin        | Dimensions    | Resolution  |
|--------|--------|-------------------------------|---------------|-------------|
| map01  | Zone A | South-west corner of Zone A   | ~180×270 cells | 0.05 m/cell |
| map02  | Zone B | South-west corner of Zone B   | ~40×200 cells  | 0.05 m/cell |

**Note:** Large map files (`.pgm`) are stored externally. Contact the author for download links.

---

## ISO 3691-4 Test Report

The full compliance test report is available at: [`ISO_3691-4_Test_Report.md`](ISO_3691-4_Test_Report.md)

### Summary

| Test | Name | Result |
|------|------|--------|
| TEST-01 | Same-Map Navigation Accuracy | ✅ PASS — Mean error 0.059 m |
| TEST-02 | Cross-Map Navigation (Switching) | ✅ PASS — 90% goal reach |
| TEST-03 | Re-Localization After Switch | ✅ PASS — Mean error 0.074 m |
| TEST-04 | Static Obstacle Avoidance | ✅ PASS — 100% detection |
| TEST-05 | Dynamic Obstacle Avoidance | ✅ PASS — 0 collisions |
| TEST-06 | Path Following (ATE) | ✅ PASS — ATE 0.090 m |
| TEST-07 | E-Stop & Safety Response | ✅ PASS — ~170 ms |
| TEST-08 | End-to-End Full Mission | ✅ PASS — 80% success |

**Overall: FULLY COMPLIANT with ISO 3691-4:2020**

---

## Navigation Log Format

Navigation logs are recorded using `iso_test_logger.py` and saved as CSV:

```csv
timestamp, x, y, yaw, map_id, goal_x, goal_y, switch_triggered, switch_time
1622000000.123, 25.03, -15.02, 0.01, map01, 25.0, -15.0, False, 0.0
```

---

## Note on Large Files

Rosbag recordings and large map files are **not committed** to this repository due to file size. They are available on request or via the IIT Delhi research data repository.
