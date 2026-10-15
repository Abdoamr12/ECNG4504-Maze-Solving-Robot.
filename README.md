# ECNG 4504 — Autonomous Maze-Solving Robot | Phase 0

## Project overview
An ESP32-S3-based 4-wheel robot uses five digital IR reflectance inputs to follow a black track on a white background. A motor driver controls the left and right wheel groups. The firmware implements sensor-driven steering corrections, turn detection, lost-line searching, an automatic startup delay, and a black-marker probe.

**Phase 0 scope:** demonstration of maze traversal and documentation of the initial firmware and hardware. The uploaded firmware does **not** implement stored maze mapping, shortest-path calculation, or a second-run replay.

## Repository contents

| Path | Description |
| --- | --- |
| `firmware/Maze_Robot_Phase0/Maze_Robot_Phase0.ino` | Exact Arduino firmware supplied by the team |
| `hardware/Phase0_KiCad_Schematic.pdf` | Exact exported KiCad schematic supplied by the team |
| `docs/Software_Architecture.svg` | Software control-flow diagram generated from the code |
| `docs/Software_Architecture.dot` | Editable Graphviz source for the software diagram |
| `docs/Pin_Mapping.md` | Firmware GPIO mapping, sensor conventions and wiring-check notes |
| `docs/Code_Architecture.md` | Detailed code modules, state machine and limitations |

## Hardware represented
- ESP32-S3 development board
- Five-channel digital IR line sensor (five input signals)
- L298N-based motor driving, controlling a right and left pair of DC motors
- Four DC gear motors and 4WD chassis
- Power supply and wiring (verify actual voltages against the assembled robot)

The provided schematic is a **PDF export**. The editable KiCad `.kicad_sch` project was not supplied. For a fully editable KiCad submission, also upload the original KiCad project from the computer where the schematic was designed.

## Firmware pins

| Function | ESP32-S3 GPIO |
| --- | --- |
| IR S1, S2, S3, S4, S5 (left to right) | 4, 5, 6, 7, 15 |
| Right enable (`ENA`) | 20 |
| Right directions (`IN1`, `IN2`) | 21, 47 |
| Left enable (`ENB`) | 0 |
| Left directions (`IN3`, `IN4`) | 48, 45 |

Sensors are read **BLACK = 0**, **WHITE = 1**, with centered line `11011`, all white `11111`, all black `00000`. See [pin mapping](docs/Pin_Mapping.md).

## Control behavior
1. Initialize pins, stop motors and wait 2 seconds after boot.
2. Poll sensors approximately every 5 ms (up to around 200 polls/second in the timing logic).
3. When the line is centered (`11011`), drive forward at PWM 104 after a short stable-reading check.
4. For sideways line deviations, brake, then perform corrective pivots with a speed selected by line error.
5. For right (`11000`) or left (`00011`) corner patterns, make a corresponding pivot and reacquire the center line.
6. When all sensors see white, perform a lost-line search after a delay.
7. When all sensors see black, probe forward and evaluate the next sensor pattern. A short black-to-white event can alternate RIGHT/LEFT turns; a longer event followed by stable white can trigger finish detection.

**Important:** The short-versus-long black marker classification uses a *time threshold*, not position measurements. It depends on speed, marker size, and calibration. Other unexpected probe outcomes default to a right turn. The logic does not verify the availability of every maze exit.

## Getting started
1. Open `firmware/Maze_Robot_Phase0/Maze_Robot_Phase0.ino` in Arduino IDE.
2. Select the matching ESP32-S3 board and serial port, with an installed ESP32 Arduino board package.
3. Check GPIO wiring against the actual vehicle and the supplied schematic before connecting motor power.
4. Upload firmware, open Serial Monitor at **115200 baud**, and test with the wheels safely lifted off the surface first.
5. Place the robot on the maze track and tune PWM and timing constants to the physical maze.

## Validation status
This repository packages the team-supplied firmware and schematic without changing the firmware. **Compilation, KiCad electrical-rule checks, wiring verification, and physical maze success have not been independently verified here.**

## Course submission
For Phase 0, submit your repository URL as part of the team's required submission ZIP, alongside the component spreadsheet, video, contribution list and team-details text file. The individual evaluation form is submitted separately as directed by the course.
