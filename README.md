# ESP32-S3 Open-Source CNC Motion Controller

**Project Title:** Design and Implementation of an Open-Source ESP32-S3-Based Multi-Axis CNC Motion Controller Optimised for Fused Deposition Modelling Applications  
**Platform:** HOIISP — Habib Open Innovation & Independent Study Platform  
**Institution:** Habib University, Engineering & Computer Science Division  
**Author:** Basil Saeed Bari (BB09892) — Computer Engineering  
**Contact:** bb09892@st.habib.edu.pk  
**Status:** Active — In Development  
**Primary MCU:** ESP32-S3 N16R8 (dual-core 240 MHz, 16 MB Flash, 8 MB PSRAM)  
**Target Axes:** 6 stepper axes (X×2, Y×2, Z×2) + 1 extruder  
**Firmware:** Klipper (primary) / Marlin 2.x (fallback)  
**PCB Target:** Single-layer (university CNC milling machine); two-layer fallback via local fabrication service

---

## What This Project Is

This repository contains the complete open-source design of a **multi-axis CNC motion controller PCB** built around the ESP32-S3 N16R8 microcontroller. The board is designed to serve as the primary controller for FDM 3D printers but is general-purpose enough for other CNC applications (laser engravers, CNC routers, pick-and-place).

The project was motivated by a specific gap in the Karachi electronics ecosystem: no well-documented, student-reproducible, locally-sourceable motion controller PCB exists. All commonly available boards (RAMPS 1.4, MKS Nano, SKR series) are either imported clones of aging designs, or proprietary units with poor local support. This board is designed from schematics to Gerbers to BOM using only components routinely stocked in the Karachi local electronics market, with PCB fabrication achievable on university equipment.

The completed board is the intended permanent controller for the companion [CSY FDM Printer](https://github.com/BasilSaeedBari/csy-fdm-printer) project and a candidate for retrofitting the bricked MakerBot Replicator 2 in the Habib University Engineering Workshop.

---

## Feature Overview

### Core Processing
- **ESP32-S3 N16R8** — Xtensa LX7 dual-core @ 240 MHz, 16 MB Flash, 8 MB PSRAM
- USB OTG (CDC serial) for firmware flashing and host communication
- Wi-Fi 802.11 b/g/n + Bluetooth 5 LE (available for future wireless control features)
- Hardware timers, DMA, PWM channels for precise stepper pulse generation
- Watchdog timer, brown-out detection in hardware

### Power System
- **Input:** 12 V or 24 V DC (solder-jumper selectable)
- **Reverse polarity protection:** P-channel MOSFET series switch (low drop, self-healing)
- **Overcurrent protection:** Blade fuse + self-resetting polyfuse at input
- **Logic rails:** 5 V and 3.3 V from cascaded regulators; electrically isolated from heater/motor supply
- **Heater outputs:** N-channel power MOSFET switching on raw input rail; fly-back diode protection
- **Back-EMF isolation:** Schottky diode on motor driver Vmot rail; controlled power-path prevents stepper back-drive energy from reaching logic circuits

### Stepper Motor Control
- **7 Pololu-style stepper driver sockets:** X1, X2, Y1, Y2, Z1, Z2, E (extruder)
- Compatible with A4988, DRV8825, TMC2208, TMC2209 and pin-compatible variants
- STEP / DIR / EN signal routing to all sockets
- UART line exposed per socket (for TMC smart driver configuration)
- All microstepping pins shorted to logic high by default (1/16 step for A4988/DRV8825)
- Per-driver directional LED, driven from STEP signal line (functional as bench diagnostics, not back-EMF-derived)

### Thermal Management
- **Heater outputs:** 1× heated bed (high-current, appropriately traced), 2× hotend heaters
- **Thermistor inputs:** 2× (bed + hotend), with pull-up resistors and dedicated ADC channels
- PID control loop in firmware for both bed and hotend
- Thermal runaway protection (firmware-enforced)

### Endstops & Probing
- X/Y/Z min and max endstop headers (6 total)
- Probe input + servo output for auto bed levelling (push-button probe supported; BLTouch-compatible header)

### Connectivity & Storage
- USB-C (CDC serial) — primary comms and firmware flashing
- UART (hardware) for Klipper host communication
- SPI + I2C breakout headers
- SD card slot (FAT32, for standalone G-code printing)
- Internal Flash/PSRAM for configuration storage

### Cooling
- 1× PWM-controlled part cooling fan output (MOSFET switched)
- 1× always-on hotend cooling fan output

### Expansion & Safety
- Filament runout sensor input (digital GPIO)
- Power loss detection input
- RGB / NeoPixel LED output (single data line, 5 V logic)
- Servo output header (for probe deployment)
- General-purpose GPIO breakout

---

## Block Diagram

```
                         ┌───────────────────────────────────┐
  12/24V DC INPUT        │           POWER SYSTEM             │
  ─────────────────────► │  [P-FET Reverse Polarity]          │
                         │  [Fuse] → [Vmot Rail]              │
                         │  [Vmot] → [5V Reg] → [3.3V Reg]   │
                         └──────────┬──────────────┬──────────┘
                                    │ Vmot         │ 3.3V / 5V
                    ┌───────────────┘              └───────────────────┐
                    │                                                   │
          ┌─────────▼──────────┐                         ┌────────────▼──────────┐
          │   HEATER OUTPUTS   │                         │     ESP32-S3 N16R8     │
          │  [Bed MOSFET]      │                         │                        │
          │  [HE0 MOSFET]      │◄────── PWM signals ────│  STEP/DIR/EN → ×7      │
          │  [HE1 MOSFET]      │                         │  ADC → thermistors     │
          └────────────────────┘                         │  GPIO → endstops       │
                                                         │  GPIO → fans, probe    │
          ┌─────────────────────────────────────────┐   │  USB OTG / UART        │
          │         STEPPER DRIVER SOCKETS (×7)     │◄──│  SPI → SD card         │
          │  [X1][X2][Y1][Y2][Z1][Z2][E]            │   └────────────────────────┘
          │  A4988 / DRV8825 / TMC2208 / TMC2209    │
          └─────────────────────────────────────────┘
```

---

## Repository Structure

```
.
├── kicad/
│   ├── esp32s3_cnc.kicad_sch      # Master schematic
│   ├── esp32s3_cnc.kicad_pcb      # PCB layout
│   └── esp32s3_cnc.kicad_pro      # KiCad project file
├── fabrication/
│   ├── R1/                        # R1 Gerbers, drill files, fabrication notes
│   └── R2/                        # R2 Gerbers (post-errata correction)
├── revisions/
│   ├── R1_errata.md               # Documented faults found during R1 bring-up
│   └── CHANGELOG.md               # Revision history
├── firmware/
│   ├── klipper/
│   │   ├── printer.cfg            # Klipper configuration for CSY printer
│   │   └── KLIPPER_COMPAT.md      # ESP32-S3 Klipper port evaluation notes
│   └── marlin/
│       └── Configuration.h        # Marlin fallback configuration
├── data/
│   └── validation/                # Oscilloscope screenshots, measurement CSVs
├── docs/
│   ├── BOM.csv                    # Full BOM with Karachi market sourcing
│   ├── ASSEMBLY.md                # PCB assembly guide (R1 and R2)
│   ├── BRINGUP.md                 # Board bring-up procedure (power sequencing)
│   ├── schematic.pdf              # Exported schematic PDF
│   └── MAKERBOT_RETROFIT.md       # Notes on MakerBot Replicator 2 retrofit (stretch goal)
├── project.md                     # HOIISP submission file
└── README.md                      # This file
```

---

## Design Decisions & Rationale

### Why ESP32-S3?

The ESP32-S3 N16R8 offers a combination of processing power, peripheral count, and memory that significantly exceeds the AVR-based (RAMPS) and STM32F1-based (MKS Nano) boards it is intended to replace, while remaining locally available in Karachi at a competitive price point. The dual-core 240 MHz LX7 architecture, combined with hardware timers and DMA, is capable of generating precise step pulses for all seven axes simultaneously without the timer-interrupt contention that limits AVR-based systems at high step rates.

### Why Single-Layer PCB Target?

The Habib University Engineering Workshop has a PCB CNC milling machine. Single-layer FR4 PCB fabrication on this machine eliminates lead time, eliminates international shipping cost, and allows R1 to be produced within days of layout completion. The design accepts this constraint by using 0-ohm resistor bridges where traces must cross, and by carefully arranging the component placement to minimise crossover count. If more than approximately 10 bridges are required, or if the USB differential pair cannot be routed with adequate trace geometry, the layout will be revised for two-layer fabrication via a local PCB service.

### Why Pololu-Socket Stepper Drivers (not onboard drivers)?

Onboard driver ICs would reduce board size and connector count, but they require individual IC replacement when a driver fails — a common occurrence during bring-up and calibration. Pololu-socket drivers can be replaced in seconds for under Rs 300. This is the correct choice for a university project where reliability of the board-as-a-platform is more important than optimising footprint.

### Why Not an Existing Open-Source Board (SKR, Octopus, etc.)?

SKR and BTT Octopus boards are well-designed, but they are imported, opaque to students who did not design them, and unavailable for local repair or modification. The engineering value of this project is precisely that the designer understands every trace and every component choice, and that a future student can open the KiCad files and modify or repair the board without starting from scratch.

---

## Bring-Up Procedure (Summary)

Full procedure in `docs/BRINGUP.md`. Key sequence:

1. **Visual inspection** — verify all components are correctly oriented and soldered; no solder bridges.
2. **Continuity check** — verify all power supply nets with multimeter; verify no short between Vmot, 5V, 3.3V, and GND rails.
3. **Power up without ESP32-S3 installed** — apply 24 V through a bench supply with 1 A current limit. Verify 5 V and 3.3 V at test points.
4. **Install ESP32-S3 module** — verify USB enumeration over CDC serial.
5. **Flash firmware** — Klipper or Marlin over USB DFU.
6. **Test stepper outputs** — command individual axes; verify STEP signal on oscilloscope; verify motor movement.
7. **Test heater outputs** — enable bed and hotend heaters; verify MOSFET switching and thermistor readback.
8. **Test endstop inputs** — trigger each endstop mechanically; verify firmware response.
9. **Thermal soak** — run bed at 60°C and hotend at 200°C for 30 minutes; measure MOSFET case temperature.
10. **Back-EMF test** — power off board; manually drive Z axis; measure 3.3V rail.

---

## Companion Project

This board is designed as the permanent controller for:

- **[CSY FDM Printer](https://github.com/BasilSaeedBari/csy-fdm-printer)** — A compact 100×100×100 mm FDM 3D printer implementing the novel Circumferential Synchronous Y-axis (CSY) belt mechanism. Currently uses RAMPS 1.4 as an interim controller while this board is validated.

---

## Stretch Goal: MakerBot Replicator 2 Retrofit

The Habib University Engineering Workshop owns a MakerBot Replicator 2 that is permanently non-functional due to a bricked proprietary control board with no viable repair or replacement path. The R2 board from this project is a strong candidate for retrofit:

- The Replicator 2 mechanical system (gantry, stepper motors, heated bed, extruder) is intact.
- The bricked board can be physically removed and the R2 board installed in its place.
- Motor, heater, thermistor, and endstop connectors will need adapters (documented in `docs/MAKERBOT_RETROFIT.md`).
- Klipper configuration for the Replicator 2 kinematics will be derived from the CSY printer config.

This is explicitly a stretch goal and is contingent on R2 validation being completed with schedule margin remaining.

---

## Licence

All schematic, PCB layout, and hardware design files are released under the **CERN Open Hardware Licence Version 2 – Strongly Reciprocal (CERN-OHL-S v2)**.  
All firmware configuration files and documentation are released under the **MIT Licence**.

You are free to fabricate, modify, and redistribute this board. Attribution appreciated. If you improve the design, please share it back.

---

## Acknowledgements

Supervised under the HOIISP Independent Study Programme, Habib University Engineering & Computer Science Division. PCB fabrication resources provided by the Habib University Engineering Workshop. Component sourcing and pricing validated against the Karachi Saddar and Urdu Bazaar electronics markets.
