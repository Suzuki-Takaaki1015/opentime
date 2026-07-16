# OpenTime

English | [日本語](docs/README.ja.md)

OpenTime is an open-source project developing a time-synchronization appliance that obtains accurate time from GNSS (satellite navigation systems such as GPS) and reliably distributes it to devices on a local network via NTP.

The first product model, **OpenTimeServer**, is intended to be an accessible rack-mount time server for individuals, home labs, small businesses, educational institutions, and research organizations, designed with continuous operation and maintainability in mind. The project aims to bridge the gap between expensive enterprise-grade appliances and typical DIY systems that may lack fault tolerance and reproducibility.

> [!IMPORTANT]
> The project is currently in the prototype planning, requirements definition, and mechanical-reference collection stage. This repository does not yet contain working hardware or software implementations.

## Why accurate time matters

When clocks drift across servers, routers, switches, firewalls, and monitoring systems, logs from multiple devices can no longer be reliably ordered. Accurate synchronization is fundamental to incident investigation, security forensics, authentication and certificate validity checks, and monitoring event records.

OpenTimeServer aims to provide a stable time reference for network equipment such as Cisco and Allied Telesis devices, as well as servers, virtualization platforms, and monitoring systems.

## Current prototype concept

The current prototype plan v0.2 proposes the following architecture:

- Two Raspberry Pi Compute Module 5 (CM5) units with eMMC in an Active / Standby configuration
- Time data and PPS signals from a timing-grade GNSS receiver
- A signal distribution circuit that feeds the GNSS signals to both CM5 units simultaneously
- Linux time synchronization and NTP service using `chrony` and `pps-tools`
- Automatic failover using Keepalived / VRRP and a virtual IP address
- Dual power inputs with Power ORing so operation can continue if one power source fails
- Time and system-status display and monitoring using an M5Stack Basic, LEDs, and an LED matrix
- A continuous-operation design using eMMC, a read-only OS, and RAM-based logging without relying on microSD cards
- A dedicated 1U or 2U enclosure compatible with a standard 19-inch rack

```text
GNSS antenna
      │
GNSS receiver (time data / PPS)
      │
Signal distribution circuit
      ├──────────────┐
      ▼              ▼
CM5-A (Active)     CM5-B (Standby)
      │              │
      └──── Wired LAN ─┘
             │
         Network clients
```

Clients will see a single NTP server at a virtual IP address. If the Active node suffers an OS, storage, or network failure, the Standby node is intended to take over. The design will expose time-synchronization, power, network, temperature, and related status through front-panel displays and monitoring functions.

## Development approach

OpenTime aims to be more than a one-off DIY build: it is intended to become a product design that others can reproduce, validate, and improve. The following deliverables are planned for publication in this monorepo:

- Schematics and custom carrier-board design files
- Mechanical and CAD data for the rack enclosure
- Software for NTP, redundancy, monitoring, and displays
- Bill of materials (BOM)
- Assembly, configuration, testing, and operating instructions

## Project status and roadmap

| Phase | Scope | Current repository status |
| --- | --- | --- |
| Phase 0 | Planning, requirements, and purchasing preparation | In progress; prototype plan and requirements are available |
| Phase 1 | Validate time reception and NTP distribution with one CM5 and GNSS | Not started |
| Phase 2 | Validate Active / Standby failover with two CM5 units | Not started |
| Phase 3 | Validate dual power inputs and uninterrupted switching | Not started |
| Phase 4 | Design and prototype the custom carrier board | Not started |
| Phase 5 | Design and prototype the 1U / 2U rack enclosure | Collecting reference 3D data |

The prototype's main success criteria are the ability to distribute GNSS-derived time over NTP, continue service after either one compute node or one power input fails, and expose the system's operating state.

## Repository structure

```text
opentime/
├── .github/                 Issue and pull request templates
├── docs/                    Plans, requirements, rules, and member information
│   ├── requirements/        Integration requirements in Japanese and English
│   └── rules/               GitHub workflow rules
├── hardware/
│   ├── electronics/         Circuit and PCB files (currently empty)
│   ├── mechanical/          Enclosure and mechanical files (currently empty)
│   └── references/          Official CM5 and M5Stack 3D reference data
├── software/                NTP, redundancy, monitoring, and UI code (currently empty)
├── LICENSE
└── README.md
```

Key documents:

- [Prototype development plan v0.2 (Japanese)](docs/opentime-prototype-plan.md)
- [Integration requirements (Japanese)](docs/requirements/requirements.jp.md)
- [Integration requirements (English)](docs/requirements/requirements.en.md)
- [GitHub workflow rules (Japanese)](docs/rules/github_rules.md)
- [Development team (Japanese)](docs/members.md)
- [Source of the official CM5 3D reference data](hardware/references/cm5/README.md)
- [Source of the official M5Stack Basic v2.7 3D reference data](hardware/references/m5stack/basic-v2.7/README.md)

> [!NOTE]
> The existing integration requirements are based on the original CM4 concept, while the newer prototype plan v0.2 adopts CM5. This README presents CM5 as the current direction. The requirements still need to be updated, and the physical network design—one port or two ports—has not yet been decided.

## Contributing and development workflow

Feature development, bug fixes, and documentation changes begin with an Issue, continue on a dedicated branch, and are merged into `main` through a reviewed pull request. See the [GitHub workflow rules](docs/rules/github_rules.md) for branch naming, commit-message, and review requirements.

## License

This repository is released under the [MIT License](LICENSE). Reference files supplied by third parties may also be subject to their respective providers' terms.
