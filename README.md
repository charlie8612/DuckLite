# DuckLite

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

- [Hardware replacement and cost table](docs/hardware-comparison.md)
- [Hardware workspace](hardware/README.md)
- [Bridge firmware](firmware/README.md)
- [Compatibility software](software/README.md)
- [Simulation workspace](simulation/README.md)

## Compatibility target

DuckLite is not a copy of Microduck hardware. Compatibility means preserving the
software-facing contract where practical:

- joint IDs `10-14`, `20-24`, and `30-34`
- IMU bridge ID `200`
- Dynamixel Protocol 2 transport presented to the host
- 14 policy-controlled joints plus one beak joint
- the existing 61-value observation and 14-action ONNX policy shape

Some low-cost devices need a DuckLite driver or bridge. Those differences are
tracked explicitly in the hardware comparison instead of being described as
drop-in compatible.

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
