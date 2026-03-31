# Tank-Level-Control-TwinCAT

**Water Level Control Lab | University West – Department of Engineering Science**

A TwinCAT 3 PLC project implementing water level control for a simulated tank. The project covers Function Block Diagram (FBD) and Structured Text (ST) programming, including on/off control and a proportional (P) controller — all implemented in IEC 61131-3.

---

## Project Overview

This project controls the water level in a simulated tank using two PLC instances:

- **PLCsim** — runs the tank simulation (water physics, visualization, rocker switch outlet)
- **PLCstudent** — runs the student-written control logic in FBD and ST

Four solutions are implemented across two POUs:

| Task | Language | Description |
|---|---|---|
| Task 1 | FBD | On/off level control with status text |
| Task 2 | FBD | Proportional (P) controller |
| Task 3a | ST | On/off level control (same logic as Task 1) |
| Task 3b | ST | Proportional (P) controller (same logic as Task 2) |

---

## Requirements

| Tool | Version |
|---|---|
| TwinCAT 3 XAE (Engineering) | 3.1 Build 4024 or later |
| TwinCAT 3 XAR (Runtime) | Included with XAE |
| Windows OS | Windows 10 or later (64-bit) |

---

## Directory Structure

```
TwinCAT Project Tank/
│
├── TwinCAT Project Tank.sln              # Visual Studio solution file
│
└── TwinCAT Project Tank/
    ├── TwinCAT Project Tank.tsproj       # TwinCAT project file
    ├── TrialLicense.tclrs                # Trial license file
    │
    ├── PLCsim/                           # PLC instance: Tank simulation (provided)
    │   ├── PLCsim.plcproj                # PLC project file
    │   ├── PlcTask.TcTTO                 # Task configuration
    │   ├── Visualization Manager.TcVMO   # Visualization manager
    │   ├── GlobalTextList.TcGTLO
    │   │
    │   ├── GVLs/
    │   │   └── GVL.TcGVL                 # Shared variables: iwActualLevel, qwPumpSpeed, qsStatusText
    │   │
    │   ├── POUs/
    │   │   └── MAIN.TcPOU                # Tank simulation logic (provided, do not modify)
    │   │
    │   └── VISUs/
    │       └── Visualization.TcVIS       # Tank simulator visualization with outlet rocker switch
    │
    └── PLCstudent/                       # PLC instance: Student control logic
        ├── PLCstudent.plcproj            # PLC project file
        ├── PlcTask.TcTTO                 # Task configuration
        │
        ├── GVLs/
        │   └── GVL.TcGVL                 # Linked GVL: maps to PLCsim shared variables
        │
        └── POUs/
            ├── MAIN.TcPOU                # Entry point: calls FunktionBlockDiagram POU
            └── FunktionBlockDiagram.TcPOU # All four task solutions (FBD Tasks 1&2, ST Tasks 3a&3b)
```

---

## Key Variables (GVL)

| Variable | Type | Direction | Description |
|---|---|---|---|
| `GVL.iwActualLevel` | WORD | Input | Current tank water level (0–100%) |
| `GVL.qwPumpSpeed` | WORD | Output | Pump speed for incoming water (0–100) |
| `GVL.qsStatusText` | STRING | Output | Status text displayed on HMI |

---

## Control Logic

### Task 1 & 3a — On/Off Control (FBD and ST)

The water level is kept between **40%** and **60%**:

| Condition | Pump Speed | Status Text |
|---|---|---|
| Level <= 40% (Low) | 60 | `'Low level'` |
| Level >= 60% (High) | 40 | `'High level'` |
| Level too high (> 60%) | 0 | `'Normal level'` |

Uses `LE` (<=) and `GE` (>=) comparison functions with `MOVE` blocks (FBD) or `IF THEN END_IF` (ST).

### Task 2 & 3b — Proportional (P) Controller (FBD and ST)

Formula: `PumpSpeed = (Setpoint - ActualLevel) x Kp`

| Parameter | Value |
|---|---|
| Setpoint | 50% (constant) |
| Kp | Start at 10, increase until oscillation |

**Important:** `GVL.iwActualLevel` is a `WORD` type and cannot hold negative values. The following type conversions are applied:
1. `WORD_TO_INT` on `iwActualLevel` before subtraction
2. `INT_TO_WORD` on the result before assigning to `qwPumpSpeed`
3. Negative controller output is clamped to zero using `SEL` (FBD) or `IF` (ST)

---

## How to Run the Project

### 1. Open the solution

1. Launch **TwinCAT 3 XAE** (Visual Studio shell)
2. Open `TwinCAT Project Tank.sln` from the project folder

### 2. Activate and start the runtime

1. Go to **TwinCAT → Activate Configuration**
2. Select **UmRT_Default** as the runtime target when prompted
3. Answer **OK** to download the configuration
4. For each PLC instance (PLCsim and PLCstudent):
   - Go to **PLC → Login**
   - Then **PLC → Start**

### 3. Open the Tank Visualization

1. In the PLCsim project, navigate to **VISUs → Visualization**
2. Open it to see the tank simulation
3. Use the **outlet rocker switch** in the visualization to toggle water outflow for testing

### 4. Observe & Test

- Watch `GVL.iwActualLevel` change as the pump reacts to the water level
- Watch `GVL.qsStatusText` update with the current status
- Toggle the outlet rocker switch to simulate varying drain conditions
- To test the P controller: switch the active task in `MAIN.TcPOU` and increase `Kp` progressively until oscillation occurs

---

## Program Architecture

```
PLCsim
└── MAIN              → Tank simulation: updates iwActualLevel based on pump speed and outlet

PLCstudent
└── MAIN
    └── calls FunktionBlockDiagram
        ├── Task 1 (FBD)  → On/Off control with LE, GE, MOVE, status text
        ├── Task 2 (FBD)  → P controller with WORD_TO_INT, SUB, MUL, SEL, INT_TO_WORD
        ├── Task 3a (ST)  → On/Off control using IF THEN END_IF, :=
        └── Task 3b (ST)  → P controller using -, *, IF, :=
```

---

## Author

**Tejashwi Jagadish**
University West – Department of Engineering Science
Course: PLC Programming (Tank Level Control Lab)
