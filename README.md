# Low-Cost F1TENTH-Like Build for Jetson Orin Nano

This repository describes a self-assembled, low-cost **F1TENTH-like** vehicle for robotics courses, student projects, and research prototyping. It retains the core architecture—a 1:10 four-wheel-drive chassis, Ackermann steering, VESC, ROS 2, 2D LiDAR, and NVIDIA Jetson—but it is **not guaranteed to comply with current RoboRacer/F1TENTH competition rules**.

[中文版（人民币预算）](https://github.com/pengliangtian06-commits/f1tenth-low-cost-build-cn)

> Assumption: you already own a NVIDIA Jetson Orin Nano, so the computer is excluded from all totals. Prices are planning estimates dated 2026-07-26. USD values use a fixed planning rate of **USD 1 = CNY 7.20**; shipping, tax, exchange-rate changes, and regional availability are excluded.

## Budget tiers (USD)

| Build | Added cost | Sensors | Suitable for |
|---|---:|---|---|
| A: vision starter | $319–417 | UVC wide-angle camera | Teleoperation, chassis control, vision following, data capture |
| B: budget LiDAR (recommended) | $500–667 | LD19-class 2D LiDAR + camera | Mapping, localization, wall following, obstacle avoidance, pure pursuit |
| C: reliability upgrade | $667–903 | RPLIDAR A2 + camera | Repeatable courses, capstone projects, longer-term lab use |

If the hard ceiling is about **$417**, build Tier A first. Verify the chassis, VESC, steering, power distribution, and camera, then add LiDAR. Never remove the fuse, emergency stop, proper regulators, or balance charger just to fit LiDAR into the initial budget.

## Recommended BOM: LD19 budget-LiDAR build

| Component | Recommended specification | Estimated cost | Search phrase |
|---|---|---:|---|
| 1:10 4WD short-course rolling chassis | 300–330 mm wheelbase, metal drivetrain, Ackermann steering | $97–153 | `1/10 4WD short course rolling chassis` |
| 3650 brushless motor | About 2,500–3,300 KV; match chassis gearing | $17–31 | `3650 brushless motor 3300KV` |
| Metal-gear steering servo | 20–25 kg, 6–8.4 V | $14–31 | `25kg metal gear servo` |
| VESC 6.x-compatible controller | USB, speed/current feedback, flashable firmware | $76–125 | `VESC 6.6 USB CAN` |
| Budget 2D LiDAR | LD19/LD06 class, 360°, verified ROS 2 driver | $97–153 | `LD19 LiDAR ROS2` |
| UVC wide-angle camera | 1080p, 90°–120° field of view | $14–31 | `UVC wide angle camera 120 degree` |
| 3S LiPo battery | 11.1 V, 4,000–5,200 mAh, ≥30 C, XT60, hard case | $36–58 | `3S 5200mAh XT60 hard case LiPo` |
| Balance charger | Genuine 3S LiPo balance charging, power supply included | $25–44 | `B6AC LiPo balance charger` |
| Dedicated Orin regulator | 3S input; low-ripple 12 V/5 A or 19 V/3 A output | $17–36 | `low ripple buck boost converter 12V 5A` |
| Servo UBEC | Independent 6 V/5 A or 7.4 V/5 A output | $7–17 | `UBEC 6V 5A servo` |
| Distribution and safety hardware | XT60 splitter, fuse, master switch, E-stop, terminals | $21–42 | `XT60 fuse emergency stop switch` |
| Upper deck and mounts | 3–5 mm FR4/acrylic, standoffs, damping mounts | $21–49 | `custom acrylic laser cutting` |
| Cables and fasteners | Short USB cables, heat-shrink, screws, ties | $21–39 | `short USB cable heat shrink screws` |
| Cooling and protection | Orin fan, LiDAR guard, LiPo fire-resistant bag | $14–31 | `LiPo safety bag fan LiDAR guard` |

An editable itemized list is available in [BOM.csv](docs/BOM.csv). The practical target for the complete LD19 build is **$500–667** after choosing value-oriented parts and fabricating the upper deck locally.

## Stage 1: target budget of $417 or less

1. Chassis, motor, and servo: target $132–174.
2. VESC 6.x-compatible controller: target $76–104.
3. 3S battery and balance charger: target $63–90.
4. Orin regulator and servo UBEC: target $25–42.
5. Fuse, E-stop, switch, and distribution: target $17–31.
6. Upper deck, mounts, and cables: target $25–49.
7. UVC camera: target $14–25.

Reusing a charger and cables or fabricating the upper deck can bring the practical total to **$319–417**. A fully new, ordinary retail purchase can reach about $351–514, so obtain quotes before ordering.

## Power and safety architecture

Use star distribution from the battery. All branches share ground, but the servo and computer use separate regulation:

```text
3S LiPo
├─ Fuse / master switch / drive E-stop → VESC → 3650 motor
├─ Independent 6 V/5 A UBEC → steering servo
└─ Low-ripple DC-DC regulator → Jetson Orin Nano → USB LiDAR/camera/VESC
```

- Never power the servo from Jetson GPIO or its 5 V rail.
- Put the fuse close to battery positive; the physical E-stop must be reachable from outside the vehicle.
- Perform the first powered test with all wheels lifted and conservative VESC current/speed limits.
- Test only in a closed area. Software obstacle avoidance does not replace a physical E-stop or barriers.
- Use a real balance charger and fire-resistant LiPo bag. Retire swollen, damaged, or abnormally hot batteries.

## Suggested ROS 2 stack

- JetPack 6.x / Ubuntu 22.04, subject to board support
- ROS 2 Humble
- `vesc_driver` and `ackermann_msgs/AckermannDriveStamped`
- LiDAR `/scan`, camera `/camera/image_raw`, odometry `/odom`
- TF: `map → odom → base_link → laser/camera`
- SLAM Toolbox and Nav2
- Follow-the-Gap, Wall Follow, and Pure Pursuit
- F1TENTH Gym ROS for simulation before hardware tests
- `rosbag2`, RViz/Foxglove, and `jtop`/`tegrastats` for recording and monitoring

Pin the JetPack/L4T version first, then select compatible ROS 2 and driver branches. Avoid combining the newest JetPack and ROS release with an old VESC or LiDAR driver without verification.

## Mechanical layout

| Location | Hardware | Reason |
|---|---|---|
| Lowest layer | Battery, VESC, fuse, master switch | Low center of gravity and short high-current wiring |
| Middle front | Jetson Orin Nano | Easy access to USB/Ethernet; separated from motor noise |
| Middle rear | Regulators, UBEC, distribution terminals | Power-domain separation, cooling, and service access |
| Top center | 2D LiDAR | Clear 360° scan near the vehicle rotation center |
| Low front | UVC camera | Adjustable road view without blocking LiDAR |
| Reachable outer edge | Physical E-stop and master switch | Power can be cut without reaching near wheels |

Prototype the upper deck in inexpensive acrylic or FR4 before upgrading to carbon fiber or aluminum. Measure the actual chassis before cutting any plate or copying a TF configuration.

## Assembly and validation sequence

1. Bench-test JetPack, ROS 2, camera, LiDAR, and VESC USB; create stable device names.
2. Assemble the chassis, motor, servo, and wheels; check gear mesh and symmetric steering with wheels lifted.
3. Build star power distribution, fuses, E-stop, regulator, and UBEC; verify voltage under load.
4. Configure conservative VESC current and RPM limits, then test low-speed commands.
5. Mount Orin and sensors, provide active cooling, short cables, strain relief, and thread locking.
6. Calibrate wheelbase, track width, wheel circumference, gearing, steering center/gain, and maximum safe angle.
7. Measure LiDAR/camera poses and configure TF; push the unpowered vehicle to validate scan direction and map quality.
8. Test Wall Follow or Follow-the-Gap, then Pure Pursuit at 0.5–1 m/s.
9. Increase speed only 10%–20% per step while measuring braking distance, temperature, and P95 command latency.

## Minimum acceptance criteria

- The physical E-stop cuts drive power; the fuse and cables do not overheat.
- Orin, VESC, and LiDAR run for 60 minutes without rebooting or disconnecting.
- A 5 m straight run at 0.5 m/s is repeatable; left and right steering are reasonably symmetric.
- An indoor loop map closes without large double-wall artifacts.
- The vehicle completes 10 laps or the specified route at 0.5–1 m/s without collision.
- Disconnecting LiDAR or stopping the control node triggers a safe stop.

## LiDAR choice

| Item | LD19-class | RPLIDAR A2 |
|---|---:|---:|
| Typical cost | $97–153 | $250–347 |
| ROS 2 support | Available, but verify the exact seller/model/firmware | Larger community and more mature documentation |
| Best use | Indoor, low-speed teaching where cost matters | Repeated lab work where support and stability matter |
| Initial speed | Limit to 1–2 m/s | Increase only after measured validation |

Before ordering, request the exact model, USB adapter, voltage, baud rate, scan rate, Linux ARM64/ROS 2 repository, and return policy. Neither option is a safety-rated automotive sensor.

## Open-source references

- [RoboRacer / F1TENTH](https://roboracer.ai/)
- [F1TENTH System](https://github.com/f1tenth/f1tenth_system)
- [F1TENTH Gym ROS](https://github.com/f1tenth/f1tenth_gym_ros)
- [F1TENTH Documentation](https://f1tenth.readthedocs.io/)
- [NVIDIA Jetson Orin Nano Getting Started](https://developer.nvidia.com/embedded/learn/get-started-jetson-orin-nano-devkit)
- [ROS 2 Humble](https://docs.ros.org/en/humble/)
- [Nav2](https://docs.nav2.org/)
- [SLAM Toolbox](https://github.com/SteveMacenski/slam_toolbox)

## Disclaimer

This project is for education and research. Re-check current prices, dimensions, connectors, power requirements, ARM64/ROS 2 driver support, return policies, and current competition rules before purchasing. Do not operate the vehicle on public roads or near crowds, stairs, glass doors, or flammable materials.
