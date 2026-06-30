# OpenTime Server - Hardware & Software Integration Requirements

## 1. Project Overview and Background

Accurate time synchronization at the millisecond level is essential in modern infrastructure systems, from network edge firewalls such as OPNsense, to core Layer 3 switches such as Cisco devices, and virtualization platforms such as Proxmox VE.

However, enterprise-grade dedicated time servers with hardware timestamping and high-precision holdover capabilities are often expensive and difficult for individual users, independent engineers, makers, and home-lab enthusiasts to adopt.

The goal of this project is to build a practical Stratum 1 time server using dual Raspberry Pi Compute Module 4 (CM4) units. The system aims to provide zero-downtime failover through a Virtual IP (VIP), offering functionality comparable to high-end commercial appliances, while also featuring an engaging hardware-centric user interface.

The entire project, including hardware design and software implementation, will be managed in a monorepo and released globally as an open-source project.

## 2. System Architecture

* **Core Compute:** Dual Raspberry Pi CM4 modules in an Active / Standby configuration.
* **Baseboard:** A custom-designed carrier board integrating the two CM4 modules, external interfaces, and signal buffering circuits.
* **Time Synchronization Flow:** NMEA and PPS signals acquired from a single GPS module are fanned out through on-board logic circuits and fed simultaneously into the GPIO pins of both CM4 modules. Kernel-level time synchronization is performed on Ubuntu using `chrony` and `pps-tools`.
* **Front-Panel Display:** An LED matrix display provides millisecond-level time display and operational status visualization.
* **Supervisor and UI:** An M5Stack Basic is integrated into the front panel. It acts as a system-wide watchdog and provides an intuitive global user interface.

## 3. Functional Requirements

* **NTP Server Functionality:**
  The system must accurately and stably respond to time synchronization requests from devices within the network.

* **PPS Signal Processing:**
  The system must directly receive the PPS (Pulse Per Second) signal from the GPS module through GPIO and process it as a kernel interrupt.

* **Hardware UI using LED Matrix Display:**

  * The current time must be displayed in real time with millisecond-level precision.
  * The display must visually separate the operational status of the Active and Standby nodes.
  * Status indicators must blink at one-second intervals synchronized with the hardware PPS signal.

* **Global Operation UI and Health Monitoring using M5Stack Basic:**

  * The M5Stack Basic must periodically perform health checks on both CM4 modules.
  * When a failure is detected, the system must provide visual alerts using screen colors, such as red, and display patterns.
  * The M5Stack Basic must provide a configuration interface for global users, including support for multiple languages and time zone adjustment.

## 4. High Availability and Failover Design

Following common enterprise redundancy practices, the system must implement transparent redundancy so that client devices see it as a single NTP server.

* **Virtual IP Sharing:**
  The two CM4 modules must run VRRP, for example using Keepalived, to share a single Virtual IP address.

* **Seamless Failover:**
  In the event of a failure on the Active CM4, such as an OS crash, storage failure, or network loss, the Standby CM4 must automatically and promptly take over the VIP. This ensures continuous time distribution while minimizing packet loss.

* **Physical Network Interface:**
  [TBD: A single-port configuration using an internal Gigabit switch IC, or a two-port configuration that supports physical cable redundancy.]

## 5. Non-Functional Requirements

* **Form Factor:**
  The system must fit within a dedicated 1U or 2U enclosure compatible with a standard 19-inch rack mount.

* **Availability and Reliability:**
  The system is designed for continuous 24/7/365 operation and must minimize points of failure.

  * The use of microSD cards is strictly prohibited.
  * Boot and storage must use the high-endurance eMMC storage on the CM4.
  * A read-only OS strategy, such as OverlayFS, must be implemented.
  * Logs must be directed to a RAM disk to prevent flash memory wear.

* **Development and Deployment:**

  * All project assets, including KiCad schematics, enclosure CAD data, C++ code, and shell scripts, must be managed in a single GitHub repository as a monorepo.
  * Appropriate licenses must be defined, and the project must be released as open source.
