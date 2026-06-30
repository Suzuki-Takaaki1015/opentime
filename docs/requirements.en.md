# OpenTime Server - Hardware & Software Integration Requirements

## 1. Project Overview and Background
From network edge firewalls (like OPNsense) to core L3 switches (like Cisco) and virtualization platforms (like Proxmox VE), accurate time synchronization at the millisecond level is critical in modern infrastructure systems.
However, enterprise-grade dedicated time servers equipped with hardware timestamps and high-precision holdover capabilities are extremely expensive and often out of reach for general engineers and home-lab enthusiasts.
The goal of this project is to build a highly practical "Stratum 1 Time Server for the masses" using dual Raspberry Pi Compute Module 4 (CM4) units. It will feature zero-downtime failover capabilities via Virtual IP (VIP)—rivaling high-end commercial appliances—and an inspiring, hardware-centric UI. The entire project (hardware design and software code) will be managed as a monorepo and released globally as an Open Source Software (OSS) project.

## 2. System Architecture
* **Core Compute:** Dual Raspberry Pi CM4 (Active / Standby configuration).
* **Baseboard:** A custom-designed carrier board integrating the two CM4s, interfaces, and signal buffering circuits.
* **Time Sync Flow:** NMEA and PPS signals acquired from a single GPS module are fanned out (duplicated) via on-board logic circuits and fed simultaneously into the GPIO pins of both CM4s. Kernel-level time synchronization (`chrony` + `pps-tools`) is handled on Ubuntu.
* **Front-end Display:** Matrix LEDs for millisecond-precision time and operational status visualization.
* **Supervisor & UI:** An M5Stack Basic integrated into the front panel. It serves as a system-wide watchdog and provides a global, intuitive UI for users worldwide.

## 3. Functional Requirements
* **NTP Server Operations:** Must accurately and stably respond to time requests from devices within the network.
* **PPS Signal Processing:** Must directly receive the PPS (Pulse Per Second) signal from the GPS module via GPIO and process it as a kernel interrupt.
* **Hardware UI (Matrix LED):**
    * Display the current time in real-time with millisecond precision.
    * Visually split the display to show the operational status of the Active and Standby nodes, synchronizing specific dots to blink at a 1-second interval driven directly by the hardware PPS signal.
* **Global UI & Health Monitoring (M5Stack Basic):**
    * Perform regular health checks on both CM4s and provide visual alerts (e.g., flashing the screen red) upon detecting a failure.
    * Serve as a configuration interface supporting multiple languages and time zone adjustments for global users.

## 4. High Availability (HA) & Failover Design
Adhering to enterprise de facto standards, the system must implement transparent redundancy, appearing as a single NTP server to client devices.
* **Virtual IP (VIP) Sharing:** Run VRRP (e.g., Keepalived) between the two CM4s to share a single Virtual IP address.
* **Seamless Failover:** In the event of a failure on the Active CM4 (OS crash, storage failure, network loss), the Standby CM4 must instantly take over the VIP, ensuring continuous time distribution with minimal packet loss.
* **Physical Network Interface:** [TBD: 1-port configuration using an internal Gigabit switch IC, or a 2-port configuration for physical cable redundancy].

## 5. Non-Functional Requirements
* **Form Factor:** Must fit within a dedicated 1U or 2U chassis compatible with a standard 19-inch rack mount.
* **Availability & Reliability:** Designed for 24/7/365 continuous operation, minimizing the failure rate.
    * The use of microSD cards is strictly prohibited.
    * Boot and storage must utilize the high-endurance eMMC on the CM4.
    * Implement a Read-Only OS strategy (e.g., OverlayFS) and direct all logs to a RAM disk to prevent flash memory wear.
* **Development & Deployment:**
    * Manage all project assets—including schematics (KiCad), enclosure CAD data, and C++/Shell script code—in a single GitHub repository (Monorepo).
    * Define appropriate licenses and publish the project as open-source.