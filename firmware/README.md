# Firmware

This directory will contain the RP2040 compatibility bridge.

Initial responsibilities:

- parse Dynamixel Protocol 2 packets at 1 Mbps
- emulate joint IDs `10-14`, `20-24`, and `30-34`
- emulate the body IMU at ID `200`
- convert goal positions to 15 PWM outputs
- sample and calibrate analog joint feedback
- report position, estimated velocity, voltage, and fault state
- enter a safe relaxed state on host timeout or invalid commands
