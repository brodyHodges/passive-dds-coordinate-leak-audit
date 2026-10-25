# Target Device Configuration Profile: Field Robot Simulation Node

This document provides a thorough technical overview of the System Under Test (SUT) utilized during the passive data leakage laboratory environment.

## 1. Hardware Specifications
* **Device Make & Model:** Raspberry Pi 5 Single Board Computer (SBC)
* **Memory Capacity:** 16GB LPDDR4X RAM
* **Processor Architecture:** Broadcom BCM2712 (ARMv8-A / 64-bit Cortex-A76)
* **Physical Core Count:** Quad-core running up to 2.4 GHz

## 2. Firmware & Operating System Layer
* **Operating System Base:** Ubuntu Server
* **OS Release Version:** 24.04.5 LTS (Noble Numbat)
* **Linux Kernel Version:** Generic Ubuntu ARM64 Kernel (`6.8.0-xx-generic`)

## 3. Robotics & Middleware Stack
* **Robotics Distribution:** ROS 2 Jazzy Jalisco
* **Default DDS Middleware:** eProsima Fast DDS (`rmw_fastrtps_cpp`)
* **DDS Domain ID Configuration:** `0` (System Default Broadcast Domain)

## 4. Network Configuration & Active Interfaces
* **Hardware Interface:** Integrated 802.11ac Wi-Fi Interface (`wlan0`)
* **Subnet Scope:** `192.168.56.0/24` (Isolated Local Laboratory VLAN)
* **Assigned Host IP:** `192.168.56.109`
* **Network Topology:** Connected via a physical wireless Access Point mapped directly to an External Virtual Switch to ensure independent routing paths from the Audit Station.
