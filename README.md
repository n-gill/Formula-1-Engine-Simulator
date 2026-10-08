# Formula-1-Engine-Simulator

[Formula 1 Engine Simulator - Google Slides](https://docs.google.com/presentation/d/17_DAtIQvywoFSsHqjK8iKNxvyVcOsvedLOcB2KJyPus/edit?usp=sharing)

A bare-metal C application that simulates vehicle powertrain physics and renders a real-time Formula 1-style digital heads-up display via VGA. Designed for the DE1-SoC FPGA board, the system features a custom physics loop handling gear ratios, engine torque curves, and overheating, alongside memory-mapped audio feedback and hardware interrupts.

## Core Features

* **Double-Buffered VGA Graphics:** Renders a high-refresh F1 dash featuring a dynamic RPM LED bar, an analog/digital speedometer dial (calculated via hardware-accelerated trigonometry), gear indicator, and telemetry status rows (DRS, BBAL, TYRE, FUEL).
* **Powertrain Physics Engine:** Calculates acceleration and speed dynamically using an 8-speed transmission system, distinct gear load factors, and an RPM-mapped torque curve. Includes clutch modeling for 1st-gear launches.
* **Audio Synthesis:** Memory-mapped exhaust audio feedback that scales the playback sampling rate based on current engine RPM.
* **Fault & Systems Simulation:** Integrates an Energy Recovery System (ERS) for torque boosting, an overheat accumulator that stalls the engine if redlined excessively, and hardware switch toggles for limp mode and cylinder misfires.

## Hardware Memory Map

The application communicates directly with the following DE1-SoC memory-mapped peripherals:

| Peripheral | Base Address | Purpose |
| --- | --- | --- |
| **Timer** | `0xFF202000` | Controls the main physics and rendering simulation loop (500,000 tick period). |
| **VGA Pixel Buffer** | `0xFF203020` | Front/Back buffer control for tear-free 320x240 RGB565 rendering. |
| **PS/2 Controller** | `0xFF200100` | Scans keyboard make/break codes for vehicle control. |
| **Audio Controller** | `0xFF203040` | FIFOs for left/right channel raw audio sample playback. |
| **Switches (SW)** | `0xFF200040` | Toggles hardware faults and simulation resets. |
| **HEX Displays** | `0xFF200020` / `0xFF200030` | 7-segment fallback display for raw engine RPM. |

## Controls

### PS/2 Keyboard

Vehicle operation is mapped to the PS/2 keyboard interface:

* **W** - Throttle
* **S** - Brake (Recharges ERS battery)
* **A / D** - Downshift / Upshift (1-8 & Neutral)
* **Spacebar** - Toggle ERS (Energy Recovery System) Deployment
* **Enter** - Engage/Release Clutch (Required for launching from a standstill in 1st gear)

### Hardware Switches

The physical sliding switches on the DE1-SoC trigger specific engine states:

* **SW0 (Limp Mode):** Hard-caps the engine to 8,000 RPM.
* **SW1 (Reset):** Clears overheat counters, resets the transmission to Neutral, and drops speed to 0.
* **SW2 (Misfire):** Simulates a cylinder misfire, randomly dropping RPM during acceleration.

## Compilation & Execution

This project is intended to be compiled and run either via the Intel FPGA Monitor Program or within the CPUlator emulator for the Nios II/V architecture.

Ensure that your environment allocates standard SDRAM (`0x20800000`) for the VGA back buffer to support the double-buffered frame swapping, and that the `engAccel` audio sample array header is included in your build directory alongside this source file.
