# Smart Insole Multimodal Sensing — Project Context and Agent Instructions

## Purpose
Develop a prototype smart shoe insole for multimodal measurements of plantar pressure and photoplethysmography (PPG). The research motivation is monitoring local tissue loading and perfusion-related optical signals, potentially informing studies of diabetic foot-ulcer risk. This is a research prototype, not a validated medical diagnostic device. PPG does not directly measure absolute blood-flow rate, and neither sensor alone establishes tissue health or detects ulceration without validation.

## System Architecture (current design)
- Five **identical peripheral sensor boards**, distributed under the insole at regions of interest; each carries one MAX30101 optical sensor, one FlexiForce A301-100 force-sensing resistor (FSR) interface, a microcontroller, and a half-duplex RS-485 transceiver.
- One **central Arduino-family controller** collects data from the five addressed peripheral nodes. The exact central controller model and firmware protocol are not yet finalized.
- Peripheral boards are intended to be **small, rigid FR-4 sensor islands**, with **flexible polyimide interconnects** between islands. Mechanical fit, strain relief, cable routing, connector height, and survivability under a foot are important.
- The five boards must use the same PCB design. Optional RS-485 termination components may be populated only on the physical bus endpoints. Do not assume a central-controller location or which ends need termination until the physical wiring is established.
- A GitHub repository named `Smart-Insole-Multimodal-Sensing` is the project source of truth; inspect actual repo files before assuming paths or statuses.

## Chosen Peripheral Components
| Function | Selected device / architecture | Notes |
|---|---|---|
| MCU | Texas Instruments MSPM0L1306TDYYR | Selected in part for its integrated op amp, ADC, UART, I²C, and flash; verify pin multiplexing against exact package/device documentation |
| Optical PPG | Analog Devices / Maxim MAX30101 | Primarily red-channel operation, target 100 samples/s; verify sensor configurations and power assumptions |
| Pressure sensor | Tekscan FlexiForce A301-100 | Analog resistive sensor; signal-conditioning circuit uses MCU-integrated op amp and ADC |
| I²C translation | PCA9306DCUR | Bridges MAX30101's 1.8-V I²C domain and MCU's 3.3-V domain |
| RS-485 | TI SN65HVD75DGK | Half-duplex, multidrop daisy-chain, nominal 230400 baud; MCU controls driver-enable/receiver-enable |
| 3.3-V LDO | TLV73333PDBVT | For MCU / applicable 3.3-V devices; verify specific rail and dropout limits |
| 1.8-V regulator | Part / exact implementation must be confirmed in current schematic | MAX30101 VDD supply |
| 5-V boost | TPS61023 | Optional battery-sourced 5-V LED / system supply; check feedback, enable, inductor, and caps against datasheet |
| Flexible bus connector | JST GH 1×06 vertical was considered/selected in design discussions | Confirm final exact part number, pinout, mating harness, and mechanical fit from schematic/BOM |
| FSR connector | 1×02, 2.54-mm-pitch vertical header was discussed | Confirm final footprint, mating connector, and suitability for footwear before purchase |
| Programming | SWD from TI MSPM0 LaunchPad / debug probe | Maintain access to correct programming and reset connections |

## Power Architecture and Constraints
- Support **two supply modes**: (1) 5-V input from central controller through wired connection; (2) local coin-cell battery powering an on-board boost converter.
- A **manual jumper** selects between central 5 V and boosted battery-derived 5 V; avoid paralleling the independent power sources. Verify backfeeding conditions and jumper topology before enabling both sources.
- A **second manual SPDT selection** was planned to supply MAX30101 VLED+ from either 3.3 V or 5 V, for reduced-voltage tests versus higher headroom when wired. Verify MAX30101 operating limits and LED current behavior at each selected voltage.
- MAX30101 logic VDD is **1.8 V**; the MCU is **3.3 V**. I²C signals must be translated appropriately.
- Battery choices explored include CR2450 and rechargeable LIR2450 options, but **the final battery arrangement, holder, chemistry, capacity, and practical runtime are NOT verified**. Never assume a CR2450 can deliver the boost converter's startup or LED-related peak current. Assess worst-case current, battery internal resistance, voltage sag, runtime, and pulse duty cycle before recommending a battery design.
- The discussed input-protection topology includes a Schottky series diode and bulk/local bypass capacitors. Confirm exact polarity, voltage ratings, placement, and actual wiring in KiCad.
- Each IC requires datasheet-appropriate bypass capacitors and stable power sequencing as applicable. Verify the precise TPS61023 implementation and all LDO input/output capacitance and ESR requirements.

## Sensors and Analog Front End
- MAX30101 will communicate over I²C through PCA9306. Its interrupt output is intended to reach an MCU GPIO. Check the **actual pull-up voltage and whether the MAX30101 INT pin is tolerant of that voltage**, rather than reusing a previously suggested pull-up uncritically.
- Initial MAX30101 operation targets **100 samples per second** in red-only mode; LED current, pulse width, timing, optical contact, and power optimization need experimental validation.
- FSR conditioning should use a **single-supply, non-inverting/reference-offset scheme** with the MCU's integrated op amp, followed by an MCU ADC input. The reference range discussed was approximately **0.2–1.25 V**, but actual reference voltage, feedback, input range, force calibration, and headroom must be proven.
- Verify that op-amp input/output and ADC channels are both available on the **actual MCU package pins**, including pin-mux and internal connection capabilities. Never assume an internal op amp output automatically reaches an ADC channel without validating the TI configuration.
- Synchronize or timestamp PPG and pressure samples so gait phase can be analyzed; note that the physical and physiological meaning of signals requires validation.

## RS-485 Bus and Firmware Design
- Use a **half-duplex, multidrop daisy-chain** RS-485 link at a target **230400 baud**.
- MCU UART TX -> transceiver D/DI; transceiver R/RO -> MCU UART RX; MCU GPIO drives DE and active-low /RE with the correct polarity. Confirm the actual schematic pins and bus idle behavior.
- Make line termination **selectively populated** so all peripheral boards remain identical. The entire physical bus should normally have termination at its two ends, not every node.
- Review bus fail-safe bias, whether integrated transceiver failsafe features suffice, termination loading, common reference conductor, ESD/TVS protection, 10-ohm series resistors proposed ahead of protection, wire lengths, supply-voltage compatibility, and connector pinout. Do not add redundant bias resistors solely because a generic application figure shows them.
- A packet format, five unique node addresses, collision-avoidance/master polling approach, CRC/checksum, sample timestamps, and fault behavior need to be designed. Do not invent protocol details as established facts.
- Firmware should implement sensor setup, pressure ADC sampling, I²C reads, safe RS-485 direction switching, and testable node communications. Favor small, readable, modular embedded C and clear assumptions.

## Latest Schematic Status — Important
- **Peripheral-board schematic is under review and PCB layout is not yet ready.** A previously uploaded project was schematic-only; any existing PCB data may be stale or unrelated to the latest schematic.
- A previous review reported **missing/inconsistent cross-sheet SDA1/SCL1/SDA2/SCL2 labels** and a **possible pin conflict**: proposed PA17/PA18 I²C connections were associated with package pins 9/10 that may already be assigned to RS-485 UART in the actual schematic.
- Earlier *proposed* pin map (NOT approved or verified): PA26/pin 1 MAX30101 INT; PA17/pin 9 SCL; PA18/pin 10 SDA; PA23/pin 14 UART TX; PA22/pin 13 UART RX; PA6/pin 8 DE + /RE; PA19/PA20 SWD. This map conflicts with other schematic-review observations and must not be treated as authoritative.
- Other open items: actual MCU pin-mux allocation (UART, I²C, integrated op amp, ADC, SWD, reset); proper MCU analog support/ROSC/VCORE connections; MAX30101 logic and LED rails; regulator values and enables; cross-sheet net continuity; RS-485 protection, fail-safe bias, and endpoint termination; FSR amplifier topology/gain/reference; connector footprint and net-to-pin consistency.
- Some schematic component values or footprints may still be placeholders, including regulator feedback/passives and FSR components. Identify placeholders, do not assume readiness for fabrication.

## Immediate Work Plan
1. **Inventory the repository**: locate project files, KiCad version, schematics, symbols, footprints, libraries, bill of materials, and any existing ERC output; report what is and is not present.
2. **Read-only schematic review first**: parse the native `.kicad_sch` and hierarchical sheets, enumerate real nets, trace power and signals, compare actual MCU pin assignments with the exact datasheet and TI pin-mux tool, and identify missing labels or disconnected pins.
3. Run KiCad ERC using installed tooling if available, but **do not treat a clean ERC as proof of electrical correctness**.
4. Deliver a prioritized issue list: **blocking**, **important**, and **optional**, citing file/sheet/symbol/pin and datasheet evidence where possible. Separate confirmed faults from uncertain hypotheses.
5. Propose a minimal, actionable schematic fix plan and explain its impacts before modifying files.
6. After explicit approval, implement fixes on a dedicated Git branch or with easily reviewable diffs. Rerun ERC and summarize changes.
7. Only after schematic review, begin component placement, routing, DRC, mechanical review, BOM verification, and manufacturing checks. Firmware can proceed in parallel when interface pin mapping is stable.

## Working Rules for OpenCode
- **Do not edit files, run destructive commands, commit, push, or order components without explicit approval**. For the first review, read-only commands and non-destructive ERC/netlist export are acceptable if they don't overwrite repository data.
- The existing architecture and listed components are **design preferences/decisions**, not invitations for wholesale redesign. Explain any required substitutions before making changes.
- **Ground assertions in current local files**. Past conversation history is background and may be outdated. If it conflicts with KiCad, call out the mismatch and ask for a decision.
- Check exact **datasheets, device revisions, and package pinouts** before asserting electrical compatibility; do not fabricate resistor values, power ratings, footprint part numbers, or net names.
- Be careful with electrical and safety-critical claims: flags should be testable and supported; avoid saying a design is 'guaranteed to work.'
- Prefer concise engineering tables (issue / exact location / reason / suggested fix / verification method), plus clear step-by-step instructions.
- Preserve KiCad file integrity, project-specific library references, and existing net connectivity unless a reviewed change requires otherwise.
- Treat underfoot height, pressure distribution, bending fatigue, protection, battery safety, and accessibility for programming/debugging as important practical constraints.
