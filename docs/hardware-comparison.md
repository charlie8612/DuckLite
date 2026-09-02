# DuckLite Vision v0.1 hardware comparison

Updated: 2026-09-02

This document compares the hardware requirements publicly visible in Pollen
Robotics' Microduck software and product documentation with the proposed
DuckLite Vision v0.1 implementation. It does not claim to reproduce Microduck's
unpublished production BOM.

Costs are estimated single-unit retail costs in USD, before tax, shipping, tools,
and failed prototype parts. A listed price is not a volume-purchase price.

## Horizontal comparison

| Function | Microduck public information | Microduck retail reference | DuckLite Vision v0.1 replacement | Qty. | DuckLite target cost | Expected difference / status |
|---|---|---:|---|---:|---:|---|
| Joint actuators | 15 Dynamixel XL330-class smart servos; exact runtime uses the XL330 control table | XL330-M288-T is $23.90 each; 15 would be $358.50 | 10x MG90S metal-gear servos for legs + 5x SG90 for head and beak | 15 | $52 | **Major difference.** PWM servos have more backlash and no native digital telemetry. RP2040 must emulate the bus; leg position feedback will be added by sensing the internal potentiometers. Walking policy must be retrained. |
| Main computer | Radxa Zero 3W, RK3566, 1 GB RAM, 32 GB storage | 1 GB/no-eMMC version is listed at $18 but currently sold out | Radxa Zero 3W 1 GB/no-eMMC + 32 GB microSD | 1 | $27 | **Should work.** Same CPU family and board support. microSD is slower and less power-loss tolerant than eMMC. Availability is a sourcing risk. |
| Motor and IMU interface | Custom `imu_to_dxl v2` board, Dynamixel Protocol 2, ID 200 | Official PCB/BOM not published | Raspberry Pi Pico/RP2040 + PCA9685 + analog multiplexer | 1 set | $10 | **Firmware bridge required.** It will expose the original servo IDs and ID 200 at 1 Mbps. This is protocol-compatible, not electrically identical. |
| Body orientation | Body IMU behind ID 200; exact production IMU IC not public | Unknown | MPU-6050-class 6-axis module on the RP2040 board | 1 | Included above | **Likely adequate for v0.1.** More drift and vibration sensitivity are expected; calibration and filtering are required. |
| Second/head IMU | Product specification says two IMUs; exact IC not public | Unknown | Connector reserved; not populated in v0.1 | 0 | $0 | **Feature omitted.** The current locomotion policy primarily needs the body IMU. Add it if head stabilization or sensor fusion requires it. |
| Front camera | Pi Camera v2 / Sony IMX219 is the only camera documented as tested; camera details are still provisional in the press kit | Official camera pricing varies; exact bundled module price not published | OV5647 CSI camera module | 1 | $8 | **Supported with a different overlay.** Lower resolution and low-light quality, but the official setup already names the OV5647 overlay. Vision models may need retuning. |
| Depth/range sensor | VL53L5CX or VL53L8CX, 8x8 ToF output | VL53L5CX breakout: about $22.70-$32.50 | VL53L1X single-zone ToF module | 1 | $6 | **Not equivalent.** DuckLite can preserve the software message shape, but cannot recover an 8x8 depth image. Suitable for forward obstacle range only. Keeping a VL53L5CX adds roughly $17-$27. |
| Audio | TLV320AIC3104 codec on a custom HAT; microphone and speaker part numbers not public | Complete module cost unknown | INMP441 I2S microphone + MAX98357A amplifier + 2 W speaker | 1 set | $5 | **Driver/configuration change required.** Recording and playback APIs can remain the same; mixer controls will differ. |
| NFC | Two antennas are advertised; controller and antenna design are not public; no NFC driver is present in the current public runtime | Unknown | Header reserved, not populated | 0 | $0 | **Feature omitted in v0.1.** Does not block walking, policy deployment, camera, or gamepad control. |
| Battery | Removable NP-F550, nominal 2S, 2600 mAh | Brand and purchase price not published | Retail-compatible NP-F550, 2200-2600 mAh | 1 | $14 | **Mechanically and electrically similar.** Cell quality and peak-current protection vary substantially by supplier. |
| Power system | Unpublished custom power/HAT electronics | Unknown | 6 V high-current servo UBEC + isolated 5 V SBC buck + fuse, switch, and bulk capacitors | 1 set | $12 | **Needs bench validation.** Separate rails reduce SBC resets, but servo stall current, noise, and battery protection trips remain risks. |
| Wiring and carrier PCB | Custom HAT, custom harnesses, and motor cables; manufacturing files not public | Unknown | Open KiCad carrier, JST-style connectors, servo leads, and ribbon cables | 1 set | $8 | **Should work after layout verification.** Connector retention and strain relief matter during falls. |
| Structure | Unpublished production CAD; simulation meshes are available but are not manufacturing CAD | Unknown | Open parametric CAD, PETG/PLA shell, TPU soles and beak pads, commodity fasteners | 1 set | $12 | **New mechanical design.** Target is 21-22.5 cm and under 500 g so the cheaper servos remain usable. MuJoCo mass and inertia must be updated. |
| Controller | Game controller included; software has been tested with an Xbox Wireless Controller | Exact bundled model not public | Existing USB/Bluetooth gamepad | 1 | $0 | **Should work** when Linux exposes standard gamepad events. Buying a controller is optional and excluded from the robot cost. |
| **Estimated DuckLite total** |  |  |  |  | **$154** | Target uncertainty is approximately +/- $20 until a real cart and one-leg bench are completed. |

## Decision summary

The approximately $150 design is feasible as an experimental platform, but it is
not performance-equivalent to a smart-servo robot. Its three deliberate cost cuts
are PWM servos, a single-zone ToF sensor, and omission of the second IMU and NFC.

The highest technical risk is the leg actuator system. Before designing the full
body, build one hip-knee-ankle chain and measure tracking error, backlash, peak
current, temperature, and usable torque at 50 Hz. If that bench fails, upgrade only
the four most loaded hip-pitch and knee joints rather than replacing all 15 servos.

## Price and compatibility references

- Microduck software: https://github.com/pollen-robotics/microduck
- Microduck RL and simulation assets: https://github.com/pollen-robotics/microduck_rl
- Microduck press kit and open-source scope: https://pollen-robotics.com/microduck/press-kit/
- ROBOTIS XL330-M288-T: https://en.robotis.com/shop_en/item.php?it_id=902-0163-000
- Radxa Zero 3W listing: https://shop.allnetchina.cn/products/copy-of-radxa-zero-3w
- Raspberry Pi camera documentation: https://www.raspberrypi.com/documentation/accessories/camera.html
- VL53L5CX retail reference: https://www.digikey.com/en/products/detail/sparkfun-electronics/18642/15775141
- Raspberry Pi Pico: https://www.raspberrypi.com/news/raspberry-pi-silicon-pico-now-on-sale/
- MG90S retail reference: https://www.circuitspecialists.com/mg90s-digital-servo

Prices and stock change. Recheck every source before ordering parts.
