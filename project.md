# ESP32-S3-Based Multi-Axis CNC Motion Controller

`Design and Implementation of an Open-Source ESP32-S3-Based Multi-Axis CNC Motion Controller Optimised for Fused Deposition Modelling Applications`

---

## GitHub Repository

`https://github.com/BasilSaeedBari/esp32s3-cnc-controller`

---

## Team Members

| Full Name | Student ID | GitHub Username | Habib Email | Program | Year | Role |
|---|---|---|---|---|---|---|
| Basil Saeed Bari | BB09892 | BasilSaeedBari | bb09892@st.habib.edu.pk | CE | 4 | Lead / Principal Investigator |

---

## Abstract

Commercial 3D printer control boards available in the Pakistani market are either imported at a significant cost premium, clones of aging designs (RAMPS 1.4, MKS Nano) with limited processing capability and poor thermal and electrical protection, or proprietary units locked to specific machine ecosystems. None are designed around components that are reliably stocked in the Karachi local electronics market, and none expose the full feature set of a modern 32-bit microcontroller in an open, student-reproducible form.

This project designs, fabricates, and validates a complete multi-axis CNC motion controller PCB centred on the ESP32-S3 N16R8 microcontroller — a dual-core 240 MHz Xtensa LX7 processor with 16 MB Flash, 8 MB PSRAM, integrated USB OTG, Wi-Fi, and Bluetooth LE. The board is designed as a general-purpose CNC motion controller but is specifically optimised and characterised for FDM 3D printing use: it provides stepper driver sockets for up to six axes (dual X, dual Y, dual Z, extruder), high-current MOSFET-switched outputs for a heated bed and two hotend heaters, thermistor inputs, a full endstop and probe header set, SD card, PWM fan outputs, filament runout and power-loss detection inputs, and a USB/UART firmware flashing interface.

The board is designed with a strong preference for single-layer PCB fabrication to enable manufacturing on the PCB milling machine available in the Habib University Engineering Workshop, while maintaining clear upgrade paths to two-layer fabrication where signal integrity demands it. All active and passive components are selected from lines stocked in the Karachi electronics market.

The project will produce Gerber files, schematics, a full BOM with local sourcing, firmware configuration, and a characterisation report validating stepper timing accuracy, thermal management, and power domain isolation. The completed board is intended to serve as the permanent controller for the companion CSY FDM printer project and as a reusable platform for future student CNC and motion control projects at Habib University.

---

## Problem Statement

The Habib University Engineering Workshop presently operates 3D printers running MKS Nano 1.2 control boards — STM32-based clones that, while functional, are closed designs offering limited documentation, no local repair ecosystem, and no expansion path. When these boards fail, replacements must be sourced internationally. The lab also holds a permanently non-functional MakerBot Replicator 2 whose original proprietary control board has no viable repair or replacement path; the machine is currently a dead asset.

More broadly, students undertaking CNC, robotics, and additive manufacturing projects at Habib University have no locally designed, open, and documented motion controller to build on. Every project that requires stepper motor control begins from scratch or imports commodity boards whose design constraints are opaque.

The specific engineering problem this project addresses is: can a full-featured, multi-axis CNC motion controller — adequate in electrical protection, thermal management, stepper driver compatibility, and firmware support to serve as the primary controller of a functional FDM 3D printer — be designed from schematics through to validated PCB using only components stocked in the Karachi local electronics market, fabricated on university equipment or low-cost local PCB services, and documented well enough that a subsequent student can build a second unit without assistance from the original designer?

A secondary problem is the recovery of the bricked MakerBot Replicator 2: the completed board, if validated in the companion printer project, is a direct drop-in candidate for retrofitting the Replicator 2 frame, restoring a high-value asset at no additional hardware cost.

---

## Domain & IEEE Alignment

**Primary Domain:** *(select one by replacing `[ ]` with `[x]`)*
- [ ] Electrical Engineering
- [ ] Computer Science
- [x] Computer Engineering
- [ ] Mechatronics / Robotics
- [ ] Telecommunications
- [ ] Power Systems
- [ ] Signal Processing
- [ ] Other: _______________

**Sub-Field / Specialization:**  
`Embedded Systems Design, PCB Design, Power Electronics, CNC Motion Control`

**Relevant IEEE Technical Society:**  
`IEEE Industrial Electronics Society; IEEE Power Electronics Society; IEEE Solid-State Circuits Society`

**Applicable IEEE Standard(s), if any:**  
`IPC-2221B (Generic Standard on Printed Board Design) — trace width and clearance calculations. IEC 60664-1 — insulation coordination for low-voltage equipment (applied informally to creepage and clearance decisions around mains-adjacent heater circuits). IEEE 1149.1 JTAG — considered for debug interface but not implemented in v1.`

---

## Objectives

1. To produce a complete, reviewed schematic for a multi-axis ESP32-S3-based CNC motion controller covering all power, stepper, thermal, endstop, connectivity, and protection sub-systems, using KiCad as the EDA tool.
2. To design a PCB layout that targets single-layer fabrication compatible with the university PCB milling machine, with all high-current traces (heated bed, heater outputs) sized to carry their rated current with less than 10°C temperature rise at 25°C ambient per IPC-2221B.
3. To fabricate and assemble a minimum of two board revisions — the first for functional bring-up and fault identification, the second incorporating corrections — producing at least one fully functional board by project completion.
4. To validate the completed board's stepper pulse timing accuracy by measuring step frequency error against a reference oscilloscope capture, targeting ≤ 1% deviation from nominal at 1/16 microstepping across all populated axes simultaneously.
5. To validate power domain isolation by measuring logic-rail voltage stability (3.3 V and 5 V rails) during simultaneous heated bed and hotend heater switching, targeting ≤ 50 mV ripple on the 3.3 V rail.
6. To publish the complete design package — KiCad project files, Gerbers, BOM with local sourcing, assembly guide, bring-up procedure, and characterisation data — to the project GitHub repository under an open hardware licence.

---

## Methodology

### Design & Simulation Phase

All schematic capture and PCB layout will be performed in **KiCad 8**. The design will proceed sub-system by sub-system, with each block reviewed against its datasheet and relevant IPC/IEC guidelines before integration into the master schematic.

**Power System design** will begin with input characterisation: the board must accept 12 V or 24 V DC (user-selectable via solder jumper). Reverse polarity protection will be implemented with a P-channel MOSFET in series with the input (lower forward drop than a diode, negligible voltage loss at operating current). A blade fuse holder and self-resetting polyfuse will provide overcurrent protection at the input stage. Separate 5 V (LDO or buck, TBD based on current budget) and 3.3 V (LDO from 5 V) regulators will supply logic and the ESP32-S3 respectively. Heated bed and heater outputs will be on the raw input voltage rail, switched by N-channel power MOSFETs (gate-driven by 5 V logic through a gate resistor network) with fly-back protection diodes.

A specific challenge is **back-EMF from stepper motors acting as generators** when printer axes are moved manually with power off (or during rapid deceleration). The design will include schottky diode isolation on the motor driver Vmot supply rail and a controlled power-path architecture to prevent this back-fed energy from reaching the logic circuitry or latching the ESP32 into an undefined boot state.

**Stepper driver interface** will support A4988, DRV8825, and TMC2208/TMC2209 drivers in standard Pololu-style sockets. Six sockets will be provided: X1, X2, Y1, Y2, Z1, Z2 for a dual-drive configuration on each linear axis, plus one E (extruder) socket. Each socket will expose STEP, DIR, EN, and UART/SPI lines. All microstepping configuration pins will be shorted to the appropriate logic level by default for maximum resolution (1/16 for A4988/DRV8825, UART-controlled for TMC series). Per-driver directional LEDs will be included, decoupled through the motor driver's STEP line rather than from the motor back-EMF signal, to be useful as on-bench diagnostics.

PCB layout will prioritise: high-current trace width per IPC-2221B, ground plane coverage to minimise impedance, physical separation between high-current heater traces and signal traces, and a single-layer constraint where achievable without sacrificing signal integrity.

### Hardware / Software Implementation Phase

**Revision 1 (R1) — Bring-up Board:**  
Fabricated on the university PCB milling machine (single-layer, hand-soldered). The objective of R1 is not a production-quality board but functional validation of every sub-circuit: power rails, USB enumeration, stepper pulse output, MOSFET switching, ADC thermistor readings, and endstop input detection. All identified faults will be documented in the `/revisions/R1_errata.md` file.

**Revision 2 (R2) — Corrected Board:**  
Incorporating all R1 errata. If single-layer is confirmed insufficient for signal integrity reasons identified in R1, R2 will be submitted to a local two-layer PCB service (JLCPCB or equivalent, ordered via a Pakistani freight forwarder). R2 is the board that will be installed in the companion CSY FDM printer for operational validation.

**Firmware:**  
The board will be configured to run **Klipper** firmware on the ESP32-S3, communicating with a host computer (repurposed laptop) over USB serial. The ESP32-S3 port of Klipper (or a sufficiently close derivative such as Klipper-on-ESP32 community ports) will be evaluated. If a stable ESP32-S3 Klipper port is unavailable at the time of firmware bring-up, **Marlin 2.x** compiled for the ESP32 will be used as a fallback, with Klipper re-evaluated for R2.

All firmware configuration files and build scripts will be version-controlled in this repository.

### Testing & Validation Phase

**Power domain validation:** Measure 3.3 V and 5 V rail voltage with an oscilloscope during bed heater and hotend heater PWM switching at 100% duty cycle, simultaneously. Target: ≤ 50 mV ripple on 3.3 V rail.

**Stepper timing validation:** Command all populated axes to step at 1,000 steps/second simultaneously. Capture STEP signal on oscilloscope. Measure step period deviation from 1 ms nominal across 1,000 consecutive steps. Target: ≤ 1% deviation.

**Thermal validation:** Run heated bed at 60°C and hotend at 200°C for 30 minutes. Measure MOSFET case temperature with a thermocouple or IR thermometer. Target: MOSFET case temperature ≤ 80°C.

**Back-EMF isolation test:** With board powered off, manually drive the Z-axis lead screw rapidly. Measure voltage on the 3.3 V rail. Target: ≤ 100 mV transient.

**Full system validation:** Install R2 board in the companion CSY printer. Complete a 2-hour print without thermal runaway, stepper fault, or USB disconnect. Pass/fail.

---

## Work Breakdown Structure (WBS)

| Milestone # | Milestone Name | Key Deliverables | Start Date | End Date | Status |
|---|---|---|---|---|---|
| M1 | Schematic Completion | Full KiCad schematic with all sub-systems reviewed; net list exported; BOM draft | 2025-06-01 | 2025-06-28 | Not Started |
| M2 | PCB Layout — R1 | Single-layer PCB layout completed; DRC clean; Gerbers generated; R1 milling/fabrication initiated | 2025-06-29 | 2025-07-19 | Not Started |
| M3 | R1 Assembly & Bring-up | R1 board assembled; all sub-systems bench-tested; errata documented | 2025-07-20 | 2025-08-09 | Not Started |
| M4 | PCB Layout — R2 | R2 layout incorporating R1 errata; Gerbers sent to fabrication | 2025-08-10 | 2025-08-23 | Not Started |
| M5 | R2 Assembly & Validation | R2 board assembled; all validation tests completed and data committed | 2025-08-24 | 2025-09-06 | Not Started |
| M6 | Printer Integration & Documentation | R2 board installed in CSY printer; 2-hour print validation passed; full design package committed to GitHub | 2025-09-07 | 2025-09-20 | Not Started |

**Estimated Total Duration:** `16 weeks`

---

## Resource Management Matrix

| Resource | Lab / Location | Estimated Hours | Purpose in Project | Required From | Required Until |
|---|---|---|---|---|---|
| PCB Milling Machine (CNC Router) | Engineering Workshop | 4 hrs | Fabricate R1 single-layer PCB prototype | 2025-07-12 | 2025-07-19 |
| Oscilloscope (≥ 2-channel, ≥ 50 MHz) | Electronics Lab | 20 hrs | Stepper pulse timing, power rail ripple, back-EMF transient measurements | 2025-07-20 | 2025-09-10 |
| Digital Multimeter | Electronics Lab | 10 hrs | Continuity checks, DC rail voltage measurements, MOSFET gate drive verification | 2025-07-20 | 2025-09-10 |
| Variable DC Bench Power Supply (≥ 24 V, ≥ 10 A) | Electronics Lab | 15 hrs | Board bring-up under controlled supply conditions before connecting switching PSU | 2025-07-20 | 2025-09-06 |
| Hot Air Rework Station | Electronics Lab | 6 hrs | SMD component rework during R1 bring-up and errata correction | 2025-07-20 | 2025-08-09 |
| Soldering Station (Fine-tip) | Electronics Lab | 20 hrs | Through-hole and SMD component soldering for R1 and R2 | 2025-07-20 | 2025-09-06 |
| IR Thermometer or Thermocouple + Meter | Electronics Lab | 4 hrs | MOSFET case temperature measurement during thermal validation | 2025-08-24 | 2025-09-10 |

> If you require access to Tier 2/3 equipment (Lathe, Drill Press, Milling Machine, PCB CNC), confirm below:

- [x] I have read Schedule A of the Terms & Conditions and will complete all required safety training before using listed Tier 2/3 equipment.

---

## Risk Assessment

| Risk | Likelihood (H/M/L) | Impact (H/M/L) | Mitigation Strategy |
|---|---|---|---|
| Single-layer routing impossible for critical signal paths (e.g., USB differential pair, UART lines crossing high-current traces) | M | M | Identify this early in M2 layout. If unavoidable, use 0-ohm resistor jumpers as layer-crossing bridges (standard single-layer workaround). Escalate to two-layer only if bridge count exceeds 10 or impedance control is required. |
| ESP32-S3 Klipper port is unstable or unsupported at time of firmware bring-up | M | M | Evaluate Klipper ESP32-S3 support during M1 (parallel to schematic work). Fallback: Marlin 2.x ESP32 build. Document evaluation in `/firmware/KLIPPER_COMPAT.md`. |
| PCB milling machine produces traces with insufficient width tolerance, causing open circuits | M | H | Design R1 with ≥ 0.5 mm minimum trace width (milling-safe). Verify milled trace width against gerber with callipers before soldering. Run continuity checks on all nets before powering. |
| Key component (ESP32-S3 N16R8, DRV8825 drivers, power MOSFET) unavailable in Karachi market | M | H | Survey component availability in Karachi market during M1 before finalising component selection. Identify pinout-compatible substitutes (ESP32-S3 N8R8, TMC2208, IRLZ44N or similar) before committing to footprints. |
| R1 board has a PCB fault causing the ESP32-S3 to be damaged during bring-up | L | H | Bring up power rails first with ESP32-S3 uninstalled. Verify 3.3 V and 5 V rails at nominal ±5% before installing microcontroller. Use a socketed ESP32-S3 module (on pin headers) for R1 to allow replacement if damaged. |
| MakerBot Replicator 2 mechanical dimensions do not accommodate the R2 board form factor | L | M | This is a stretch-goal application; incompatibility does not affect primary project success. If time permits, measure Replicator 2 cavity dimensions during M1 and flag any form-factor conflicts for R3 (future work). |

---

## Success Metrics

| Metric | Target Value | Measurement Method |
|---|---|---|
| 1. 3.3 V rail ripple during simultaneous bed + hotend heater switching at 100% PWM duty | ≤ 50 mV peak-to-peak | Oscilloscope, 10× probe on 3.3 V rail, heaters on full load |
| 2. Stepper pulse period deviation from nominal at 1,000 steps/s (all axes simultaneously) | ≤ 1% | Oscilloscope STEP signal capture, statistical measurement over 1,000 steps |
| 3. MOSFET case temperature during 30-minute bed (60°C) + hotend (200°C) simultaneous operation | ≤ 80°C | IR thermometer or thermocouple probe on MOSFET package |
| 4. Back-EMF transient on 3.3 V rail during manual axis driving with board powered off | ≤ 100 mV | Oscilloscope on 3.3 V rail during vigorous manual axis movement |
| 5. Continuous print duration without fault on R2 board installed in CSY printer | ≥ 2 hours | Supervised print run, pass/fail |
| 6. Independent reproducibility: second unit buildable from repository documentation alone | Pass (binary) | Structured walkthrough review or documented independent attempt |

---

## Project Updates

### Update Log

#### 2025-06-01 — Project Initiated
Schematic sub-system breakdown completed. Power system and stepper driver socket design begun in KiCad. Component availability survey of Karachi electronics market initiated.

---

## Data & Documentation Plan

- KiCad project source files (schematic + PCB) in `/kicad/`
- Gerber files for each revision in `/fabrication/R1/` and `/fabrication/R2/`
- Bill of Materials (with Karachi market sources) as `/docs/BOM.csv`
- Revision errata logs in `/revisions/`
- Oscilloscope screenshots and measurement data in `/data/validation/`
- Firmware configuration files in `/firmware/`
- Assembly and bring-up guide in `/docs/ASSEMBLY.md`
- Schematic PDF export in `/docs/`

**Repo structure commitment:** I understand that the HOIISP project page links directly to this repository. The repo should be organised and have a readable top-level README.

- [x] Confirmed.

---

## Budget Estimate

| Item | Source | Estimated Cost (PKR) |
|---|---|---|
| ESP32-S3 N16R8 module (×3, for R1, R2, and spare) | Karachi electronics market | 3,600 |
| A4988 / DRV8825 stepper driver modules (×7) | Karachi electronics market | 2,100 |
| N-channel power MOSFETs (IRLZ44N or equivalent, ×6) | Karachi electronics market | 600 |
| P-channel MOSFET for reverse polarity protection (×2) | Karachi electronics market | 300 |
| LDO regulators (5 V, 3.3 V) + capacitors + inductors | Karachi electronics market | 500 |
| Resistors, capacitors (SMD and through-hole assortment) | Karachi electronics market | 400 |
| Screw terminals (5.08 mm pitch, assorted) | Karachi electronics market | 400 |
| JST connectors (2.54 mm, 2–6 pin, assorted) | Karachi electronics market | 400 |
| Pin headers (2.54 mm, male and female) | Karachi electronics market | 300 |
| USB-C connector (for firmware flashing) | Karachi electronics market | 200 |
| SD card socket | Karachi electronics market | 200 |
| Blade fuse holder + polyfuse (×4 each) | Karachi electronics market | 300 |
| Schottky diodes (1N5819 or equivalent, ×20) | Karachi electronics market | 200 |
| FR4 single-sided copper-clad board (for PCB milling) | Karachi electronics market | 600 |
| Solder, flux, desoldering braid | Electronics Lab (at cost) | 300 |
| Two-layer PCB fabrication (R2, if required; JLCPCB via freight forwarder) | JLCPCB | 2,500 |
| Miscellaneous (standoffs, enclosure hardware, thermal compound) | Local market | 500 |
| | **Total Estimated:** | **13,400** |

---

## References

[1] Espressif Systems, *ESP32-S3 Technical Reference Manual*, v1.3, 2023. [Online]. Available: https://www.espressif.com/sites/default/files/documentation/esp32-s3_technical_reference_manual_en.pdf

[2] Texas Instruments, *DRV8825 Stepper Motor Controller IC Datasheet*, SLVSA73F, 2015. [Online]. Available: https://www.ti.com/lit/ds/symlink/drv8825.pdf

[3] Trinamic Motion Control GmbH, *TMC2209 Datasheet — Ultra-Silent Motor Driver IC*, Rev. 1.09, 2020. [Online]. Available: https://www.trinamic.com/

[4] IPC, *IPC-2221B: Generic Standard on Printed Board Design*, IPC International, Bannockburn, IL, 2012.

[5] Klipper3d Project, *Klipper Firmware Documentation*, 2024. [Online]. Available: https://www.klipper3d.org/

[6] Marlin Firmware Project, *Marlin 2.x Documentation*, 2024. [Online]. Available: https://marlinfw.org/

[7] R. Mammano and B. Andreycak, "Fundamentals of power supply design," *Texas Instruments Seminar Series*, 2014. [Online]. Available: https://www.ti.com/

[8] A. Sadiku and C. Alexander, *Fundamentals of Electric Circuits*, 7th ed. New York: McGraw-Hill, 2020.

---

## Declaration

- [ ] I confirm that the information in this `project.md` is accurate and complete.
- [ ] I have read and agree to the HOIISP Terms & Conditions in full.
- [ ] I confirm that all listed team members are aware of this submission and have agreed to participate.
- [ ] I confirm that at least one team member has committed to this repository using a `@st.habib.edu.pk` Git email address.
- [ ] I understand that approval of this submission grants access only to the resources explicitly listed in the Resource Management Matrix.
- [ ] I understand that I must push a `project.md` update at least once every two weeks to maintain Active project status and lab access.
- [ ] I accept that all content parsed from this repository will be publicly visible on the HOIISP platform.
- [ ] I confirm this repository is set to **Public** visibility on GitHub (HOIISP cannot read private repositories).

---

## For Office Use Only
> *Do not fill in this section.*

| Field | Value |
|---|---|
| HOIISP Submission ID | |
| GitHub Verification Status | |
| Assigned Project Slug | |
| Review Outcome | Approved / Rejected / Revisions Required |
| Review Notes | |
| Admin | |
| Date of Decision | |
| Resource Request Forwarded | |
| Lab Access Activated | |
