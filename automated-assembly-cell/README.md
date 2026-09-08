# Automated Assembly Cell — Siemens TIA Portal V17

## Project Overview

This project is an automated assembly cell developed and simulated using **Siemens TIA Portal V17** and **S7-PLCSIM**.

The system controls a conveyor and a pneumatic assembly cylinder through a sequential control architecture implemented with PLC Function Blocks. The project includes automatic sequence control, operator commands, sensor monitoring, fault detection, and timeout protection.

The main objective of the project is to demonstrate practical skills in **PLC programming, industrial automation, HMI development, sequential control, and machine fault handling**.

---

## Technologies

* Siemens TIA Portal V17
* Siemens S7-1200 PLC
* S7-PLCSIM
* Siemens HMI
* Ladder Logic (LAD)
* PROFINET / Industrial Ethernet
* Timers (TON)
* PLC Function Blocks
* Data Blocks
* HMI Tags

---

## System Architecture

The project is organized using a modular PLC architecture:

                    ┌─────────────────┐
                    │        OB1           │
                    │   Main PLC Program   │
                    └──────────┬──────┘
                                  │
                ┌─────────────┴────────┐
                │                             │
                ▼                             ▼
       ┌─────────────┐          ┌─────────────┐
       │  FB_Conveyor    │          │  FB_Assembly    │
       │ Conveyor Control│          │ Assembly Logic  │
       └──────┬────┘             └─────┬───────┘
                │                            │
                ▼                            ▼
       ┌─────────────┐          ┌─────────────┐
       │  DB_Conveyor    │          │  DB_Assembly    │
       └─────────────┘          └─────────────┘
```

The PLC communicates with the HMI through a **PROFINET PN/IE network**.

---

## Assembly Sequence

The assembly process is controlled using a state-machine approach.

```text
             ┌───────────┐
             │  0 — IDLE    │
             └──────┬────┘
                      │
                   START
                     ▼
             ┌───────────┐
             │ 10 — WAIT    │
             │ FOR PART     │
             └──────┬────┘
                      │
               Part detected
                    ▼
             ┌───────────┐
             │20 — EXTEND   │
             │ CYLINDER     │
             └────┬──────┘
                    │
             Cylinder extended
                    ▼
            ┌────────────┐
            │30 — ASSEMBLY   │
            │    2 sec       │
            └──────┬─────┘
                     │
                     ▼
             ┌───────────┐
             │40 — RETRACT  │
             │ CYLINDER     │
             └────┬──────┘
                   │
          Cylinder retracted
                   ▼
           ┌──────────────┐
           │  50 — COMPLETE   │
           └──────┬───────┘
                    │
                    ▼
           ┌──────────────┐
           │  0 — IDLE        │
           └──────────────┘
```

### Fault state

If the cylinder does not reach the expected position within the configured timeout, the sequence enters:

```text
90 — FAULT
```

Two timeout conditions are implemented:

```text
State 20
   │
   ├── Cylinder_Extended
   │       ↓
   │     State 30
   │
   └── 3 s timeout
           ↓
       State 90
```

and:

```text
State 40
   │
   ├── Cylinder_Retracted
   │       ↓
   │     State 50
   │
   └── 3 s timeout
           ↓
       State 90
```

This prevents the machine from remaining indefinitely in an intermediate state when a sensor or actuator fails to respond.

---

##  PLC Program Structure

### OB1

The main organization block coordinates the machine logic and calls the different function blocks.

### FB_Conveyor

Responsible for the conveyor control logic.

### DB_Conveyor

Instance data block associated with `FB_Conveyor`.

### FB_Assembly

Controls the complete assembly sequence using state-based logic.

Main states:

| State | Description      |
| ----: | ---------------- |
|     0 | IDLE             |
|    10 | WAITING FOR PART |
|    20 | EXTENDING        |
|    30 | ASSEMBLY         |
|    40 | RETRACTING       |
|    50 | COMPLETE         |
|    90 | FAULT            |

### DB_Assembly

Instance data block associated with `FB_Assembly`, containing the data required by the assembly sequence.

---

## HMI

The HMI provides an operator interface for controlling and monitoring the cell.

### Operator Controls

* **START** — starts the machine cycle.
* **STOP** — stops machine operation.
* **RESET** — resets the sequence and allows recovery from a fault.

### Machine Status

The HMI displays:

* Machine RUN status
* Machine FAULT status
* E-STOP status
* Part detection
* Cylinder extended position
* Cylinder retracted position
* Assembly complete status
* Emergency Stop State

The HMI communicates with the PLC through the configured **PN/IE_1 PROFINET network**.

---

## Main Signals

### Operator Inputs

```text
Start_PB
Stop_PB
Reset_PB
```

### Process Inputs

```text
Part_Sensor_Assembly
Cylinder_Extended
Cylinder_Retracted
```

### Machine Outputs

```text
Cylinder_Extend
Cylinder_Retract
```

### Status Signals

```text
Machine_Run
Machine_Fault
Assembly_Complete
```

---

## Fault Handling

The project includes basic machine fault detection through actuator response monitoring.

For example, during cylinder extension:

```text
Command:
Cylinder_Extend = TRUE

Expected feedback:
Cylinder_Extended = TRUE
```

If the feedback is not received within **3 seconds**, the PLC transitions to the fault state:

```text
State = 90
```

The same strategy is applied during cylinder retraction.

This approach represents a basic form of **timeout-based diagnostic logic**, commonly used in industrial automation to detect actuator or sensor failures.

---

## Simulation and Testing

The complete sequence was tested using **S7-PLCSIM**.

### Normal Cycle

The following sequence was successfully tested:

```text
START
 ↓
Part Detection
 ↓
Cylinder Extension
 ↓
Extended Sensor
 ↓
2 s Assembly Time
 ↓
Cylinder Retraction
 ↓
Retracted Sensor
 ↓
Cycle Complete
```

### Fault Tests

The following fault conditions were also tested:

* Cylinder extension timeout
* Cylinder retraction timeout
* Machine reset
* Machine stop

The timeout logic correctly transitions the sequence to the `FAULT` state when the expected sensor feedback is not received.

---

## Project Structure

```text
Automated-Assembly-Cell/
│
├── TIA Portal Project/
│   └── Automated Assembly Cell.ap17
│
├── README.md
│
└── Documentation/
    ├── PLC/
    ├── HMI/
    └── Screenshots/
```

---

## Future Improvements

Possible future development includes:

* Recipe management
* Production counter
* Cycle time monitoring
* Alarm history
* Manual/Automatic operating modes
* Maintenance mode
* HMI diagnostics screen
* Conveyor speed control
* Additional safety interlocks
* Real hardware commissioning

---

## Project Purpose

This project was developed as a personal automation project to demonstrate practical experience with:

**PLC Programming → Sequential Control → HMI → Industrial Networking → Diagnostics → Simulation**

The project focuses on applying industrial automation concepts in a modular and structured PLC architecture using Siemens TIA Portal V17.
