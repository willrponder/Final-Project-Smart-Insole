# Smart Insole: Multimodal Sensing for Foot Health Monitoring

## Project Overview

This project aims to develop a smart insole incorporating multimodal sensing to monitor foot health and investigate the detection of diabetic foot ulceration risk.

The system combines photoplethysmography (PPG) and pressure sensing to characterize localized tissue health and mechanical loading across different regions of the foot.

## System Architecture

The proposed system consists of:

- **PPG Sensors:** MAX30101-based sensing modules for collecting localized optical measurements.
- **Pressure Sensors:** Pressure-sensing elements for measuring localized plantar loading.
- **Peripheral Sensor Boards:** Distributed electronics for acquiring data from sensors positioned throughout the insole.
- **Central Controller:** Coordinates sensor communication, data acquisition, and system operation.
- **Communication Interface:** Provides communication between peripheral sensor boards and the central controller.

## Repository Structure

```text
Smart-Insole/
├── Hardware/
│   └── Peripheral-Board/
├── Libraries/
│   ├── Footprints/
│   ├── Symbols/
│   └── 3D-Models/
├── Documentation/
│   ├── Datasheets/
│   ├── Design-Docs/
│   └── Deliverables/
├── Test/
├── README.md
└── .gitignore
```

## Current Development Status

- [ ] Finalize system architecture
- [ ] Complete peripheral board schematic
- [ ] Finalize sensor selection and interfaces
- [ ] Complete PCB layouts
- [ ] Manufacture and assemble prototype boards
- [ ] Develop embedded firmware
- [ ] Integrate sensors into the insole
- [ ] Validate sensing performance

## Design Tools

- **PCB Design:** KiCad
- **Embedded Development:** TBD
- **Version Control:** Git / GitHub

## Documentation

Project documentation, component datasheets, design decisions, and system diagrams are maintained in the `Documentation/` directory.

## Contributors

Project development is currently coordinated through this repository. Contributions and design feedback are welcome.

---

*This is an ongoing research and development project. The system is intended for investigation and prototyping, not as a clinically validated diagnostic device.*