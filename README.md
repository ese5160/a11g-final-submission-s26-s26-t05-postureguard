[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/Y5lYn2wb)

# a11g-final-submission

**Team Number:** T05

**Team Name:** PostureGuard

| Team Member Name | Email Address | GitHub Username |
| ---------------- | ------------- | --------------- |
| Aditya Agarwal         | aditya10@seas.upenn.edu     | aditya10-source    |
| Vihaan Ravishankar         | vihaan1@seas.upenn.edu     | vihaanpenn    |

[GitHub Repository URL](https://github.com/ese5160/a11g-final-submission-s26-s26-t05-postureguard.git)

## 1. Video Presentation

## 2. Project Summary

### Posture Detection and Monitoring Device

#### Device Description

Our posture detection and monitoring device provides real-time feedback for poor sitting posture through a wearable haptic interface. The system detects when a user slouches or reclines excessively and vibrates to encourage corrective movement, with live data visualized in a cloud-connected web dashboard.

The inspiration came from understanding that chronic poor posture leads to back pain and musculoskeletal issues—problems that are nearly invisible until they cause damage. A reactive, passive alert system that the user can feel immediately addresses the core issue: awareness. Rather than passive logging, the haptic motor makes postural feedback instantaneous and tangible.

#### Internet Augmentation

The device leverages Azure cloud infrastructure to centralize data processing and provide persistent analytics. All sensor data streams wirelessly from the MCU to an Azure VM running Node-RED, which handles real-time alert logic, data aggregation, and historical trend analysis. Users access a live web dashboard (HTML + index.html frontend) that displays current posture metrics, alert history, and progress tracking. This cloud connectivity transforms a standalone sensor device into a complete monitoring ecosystem—enabling remote insights, pattern recognition across sessions, and caregiver oversight.

---

### Device Functionality

The device integrates 10 distributed pressure sensors across a chair or cushion pad to measure seat contact distribution, a flex sensor to track recline angle, and a 6-DOF IMU for detecting spinal tilt and rotation. These sensors feed into a custom PCB built around the **SiWx917 wireless MCU**, which performs sensor fusion and hosts the alert logic. When posture deviates beyond configurable thresholds—asymmetric pressure, excessive recline, or slouching detected by the IMU—the MCU triggers a haptic vibration motor to alert the user in real-time.

All sensor data streams wirelessly to an Azure VM running Node-RED for centralized processing and storage. A lightweight web dashboard displays live posture metrics, alert history, and trend analysis, allowing users and caregivers to monitor posture correction progress.

#### System-Level Block Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      SENSOR TIER                             │
├──────────────┬──────────────┬──────────────────────────────┤
│  Pressure    │  Flex        │  IMU (6-DOF)                 │
│  Sensors(10) │  Sensor      │  Rotation & acceleration     │
└──────┬───────┴──────┬───────┴──────────┬────────────────────┘
       │              │                  │
       └──────────────┼──────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│              CUSTOM PCB + SiWx917 MCU                        │
│  • I²C/UART sensor interface                                │
│  • Sensor fusion & posture detection                        │
│  • Haptic motor control & feedback                          │
│  • WiFi wireless transmission                               │
└─────────────────────────────────────────────────────────────┘
                      │
                      │ WiFi
                      ▼
┌─────────────────────────────────────────────────────────────┐
│              AZURE VM + NODE-RED                             │
│  • Real-time data processing & aggregation                  │
│  • Alert logic & thresholds                                 │
│  • Historical data storage                                  │
└─────────────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│          BROWSER DASHBOARD (index.html)                      │
│  • Live posture metrics & visualizations                    │
│  • Alert history & trends                                   │
│  • User-facing configuration                                │
└─────────────────────────────────────────────────────────────┘
```

#### Hardware Setup

![Hardware setup and sensor placement](hardware_base.jpeg)

#### Cloud Dashboard

![Node-RED dashboard and processing logic](node-red.png)

#### Web Interface

![Live index.html web dashboard screenshot](html_page.png)

---

### Challenges

#### Hardware Bring-Up Issues

We discovered three critical issues during initial PCB testing:

- **Missing battery charger connector:** The connector was completely absent from the board layout, leaving no way to charge the battery.
- **DNP jumpers that were actually required:** Several jumpers marked as "Do Not Populate" turned out to be essential for sensor configuration and power distribution.
- **Unsoldered SMT resistor:** A critical pull-up resistor for I²C communication was missing from the reflow process.

**How we overcame it:** We hand-soldered the missing connector using fine-pitch soldering techniques, populated the required jumpers, and reflowed the missing resistor. While time-consuming, this approach was faster than spinning a new PCB and allowed us to proceed with firmware development. We learned the value of functional validation before manufacturing—a simple continuity test and voltage check would have caught these issues during design review.

#### Scheduling Setback

Both team members fell ill during the critical Altium PCB design phase, losing nearly two weeks to illness and recovery. This compression forced us to prioritize ruthlessly: we stripped non-essential features (SD card logging, BLE multi-device support) and focused entirely on core sensor acquisition, local processing, and cloud integration.

**Resolution:** Despite the setback, the core system shipped on schedule. The illness actually taught us that modular design pays dividends—because sensor acquisition was decoupled from optional features, we could cut scope without destabilizing the critical path.

---

### Prototype Learnings

#### 1. Design for Assembly

We learned the hard way that component placement and pad accessibility matter as much as electrical correctness. A gaping hole in the center of the board—left from an early design iteration—was wasted real estate and made the PCB look unfinished. Scattered component placement also made hand-soldering the missing parts unnecessarily difficult.

**If we rebuilt it:** We'd plan the layout with assembly-in-mind from the start. Group components by function (all sensor I/O on one edge, power delivery on another), leave no dead zones, and ensure all pads are accessible. We'd also run a design review with manufacturing in mind—catching layout issues before tape-out.

#### 2. Redundancy in Critical Connectors and Passives

A single missing connector or resistor shouldn't halt the entire project. The fix was manual soldering, but it exposed a fragile design.

**Next iteration:** We'd add pull-up/pull-down resistors as solderable footprints (not just on-board traces), include backup connector test points, and route critical power rails to multiple accessible locations. We'd also keep spares of the charger connector and critical passives on hand.

#### 3. Modular Testing Saves Integration Time

We tested the full assembled board as a monolith. Catching the jumper and resistor issues earlier would have been trivial if we'd verified each sensor's I²C/UART communication independently before full integration.

**Next time:** Build a simple verification routine for each peripheral during PCB bringup—test pressure sensor readout, flex sensor linearity, IMU initialization—before wiring everything together. This shifts debugging from "the whole system is broken" to "peripheral X is broken."

#### 4. Cloud-First Data Pipeline

Streaming raw sensor data to Node-RED allowed us to prototype alert thresholds and data fusion logic without recompiling firmware. When we realized the pressure distribution algorithm needed tuning, we could adjust it server-side in minutes.

**Lesson:** Even for embedded devices, consider where the business logic lives. Moving alert thresholds to the cloud made the device more flexible and the user experience more responsive to iteration.

---

### Summary

This project taught us that hardware integration is iterative, scheduling happens in the real world (not on a Gantt chart), and that modular design—separating sensor acquisition from processing from presentation—is how you survive setbacks. The posture detection device works, alerts users in real-time, and provides meaningful data on their sitting habits. Next time, we ship with a cleaner PCB, better pre-manufacturing validation, and a more robust firmware test suite.

## 3. Hardware & Software Requirements

## 4. Project Photos & Screenshots

## 5. Codebase

Do *not* commit any of your source code to this repository. Rather, provide links to the other GitHub repository you've already been using with your firmware.

- A link to your final embedded C firmware codebases
- A link to your Node-RED dashboard code
- Links to any other software required for the functionality of your device