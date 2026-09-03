# DuckLite

DuckLite 是一個低成本、完整開源硬體的視覺雙足機器人平台，目標是讓使用者能以
約 US$150 的單件零售零件自行組裝、訓練及修改。

DuckLite is a low-cost, fully open-hardware vision biped platform targeting an
approximately US$150 self-build using single-unit retail parts.

DuckLite is an open-hardware, low-cost vision biped designed for learning,
simulation, reinforcement learning, and physical experimentation.

The first target is **DuckLite Vision v0.1**:

- 15 physical degrees of freedom
- camera, body IMU, basic ranging, microphone, and speaker
- approximately 22 cm tall and under 500 g
- target self-build cost of about **US$150** using single-unit retail pricing
- compatibility with Microduck's joint IDs, 50 Hz control loop, and ONNX policy interface
- reproducible mechanical, electronic, firmware, and simulation sources

## Start here

- [硬體替代與成本比較 / Hardware replacement and cost table](docs/hardware-comparison.md)
- [Hardware workspace](hardware/README.md)
- [Bridge firmware](firmware/README.md)
- [Compatibility software](software/README.md)
- [Simulation workspace](simulation/README.md)

## Microduck software compatibility

DuckLite aims to provide a low-cost, open-hardware platform for the open-source
Microduck software ecosystem.

Initial compatibility targets include:

- the 50 Hz robot control loop
- public joint and IMU interfaces
- ONNX locomotion policies
- the [`microduck_rl`](https://github.com/pollen-robotics/microduck_rl) MuJoCo,
  PPO, and sim-to-real training pipeline

DuckLite hardware and bridge firmware are independently developed. Compatibility
is a target and will be verified progressively on physical prototypes.

中文：DuckLite 希望成為 Microduck 開源軟體生態的低成本開源硬體平台，優先支援
其控制介面、ONNX 行走模型，以及 `microduck_rl` 模擬與訓練流程。

## Status

The project is at architecture and component-selection stage. Prices are targets,
not supplier quotations. The next milestone is a one-leg actuator and feedback
bench before committing to the complete mechanical design.

## Licensing direction

- Hardware design: CERN-OHL-S-2.0
- Firmware and software: Apache-2.0
- Documentation: CC-BY-SA-4.0

Canonical license texts and per-directory SPDX headers will be added before the
first hardware release.
