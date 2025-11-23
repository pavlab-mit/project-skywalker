# Project Skywalker Documentation

Welcome to the comprehensive documentation for Project Skywalker - a platform for autonomous UAV swarm research using MOOS-IvP on the Skywalker X8 fixed-wing aircraft.

## 📖 What This Documentation Covers

This documentation provides everything you need to build, configure, and deploy autonomous UAV systems using the Skywalker X8 platform with MOOS-IvP software.

### Getting Started
- **[Installation Guide](installation.md)** - Set up your Raspberry Pi and install MOOS-IvP stack
- **[Quick Start](quickstart.md)** - Get up and running with a basic configuration
- **[Hardware Assembly](hardware.md)** - Physical assembly and 3D-printed components

### Configuration & Setup
- **[Radio Setup](radio-setup.md)** - Configure RFD900x radios and mesh networking
- **[Firmware Guide](firmware.md)** - Flash and configure ArduPilot on CubeOrange

### Using the System
- **[Usage Guide](usage.md)** - Running missions and operating the UAV
- **[Architecture](architecture.md)** - System design and repository structure

### Reference & Support
- **[Troubleshooting](troubleshooting.md)** - Common issues and solutions
- **[Contributing](contributing.md)** - How to contribute to the project
- **[Authors](authors.md)** - Contributors and acknowledgements
- **[Changelog](changelog.md)** - Version history and updates

## 🎯 Project Overview

Project Skywalker is a research platform that combines:

- **MOOS-IvP**: Mission Oriented Operating Suite with Interval Programming for autonomous behavior
- **Skywalker X8**: Robust fixed-wing UAV platform with 2+ hour endurance
- **CubeOrange+**: Advanced autopilot hardware running ArduPilot
- **Raspberry Pi**: Onboard companion computer for MOOS-IvP and high-level autonomy
- **RFD900x / Rocket M5**: Long-range telemetry and/or high-bandwidth mesh networking

This combination enables research in:
- Multi-agent coordination and swarm behaviors
- Long-range autonomous missions
- Mesh networking in GPS-denied environments
- Behavior-based autonomy with interval programming

## 🚀 Quick Links

- **GitHub Repository**: [pavlab-mit/project-skywalker](https://github.com/pavlab-mit/project-skywalker)
- **MOOS-IvP Website**: [moos-ivp.org](https://moos-ivp.org/)
- **ArduPilot**: [ardupilot.org](https://ardupilot.org/)
- **Contact**: team@pavlab-mit.org

## 📚 Documentation Structure

```
docs/
├── index.md                  # This file - Documentation home
├── installation.md           # Raspberry Pi and MOOS-IvP setup
├── quickstart.md            # Quick start guide
├── usage.md                 # How to use the system
├── architecture.md          # System architecture
├── hardware.md              # Physical assembly
├── radio-setup.md           # Radio configuration
├── firmware.md              # Autopilot firmware
├── development.md           # Development environment
├── troubleshooting.md       # Common problems and solutions
├── contributing.md          # Contribution guidelines
├── authors.md               # Credits and acknowledgements
└── changelog.md             # Version history
```

## 🛠️ Key Components

### Hardware
- Skywalker X8 airframe
- CubeOrange/CubeOrange+ autopilot
- Raspberry Pi 4B companion computer
- RFD900x long-range telemetry radio
- Optional: Rocket M5 for high-bandwidth mesh

### Software Stack
- ArduPilot on CubeOrange for flight control
- MOOS-IvP on Raspberry Pi for autonomy
- Custom MOOS applications for UAV control
- MAVSDK for autopilot communication
- Mesh networking (BATMAN/OLSR) for swarm coordination

### Communication
- MAVLink protocol between Raspberry Pi and CubeOrange
- RFD900x for long-range (20-40+ km) low-bandwidth mesh
- Optional Rocket M5 for high-bandwidth (Mbps) mesh
- Ground control via QGroundControl or MOOS shore-side

## 🎓 Prerequisites

To work with this platform, you should have:
- Basic understanding of Linux/Bash
- Familiarity with UAV concepts and flight operations
- Experience with embedded systems (helpful but not required)
- Access to required hardware components

## 📞 Getting Help

If you need assistance:
1. Check the [Troubleshooting Guide](troubleshooting.md)
2. Search existing [GitHub Issues](https://github.com/pavlab-mit/project-skywalker/issues)
3. Contact the team at team@pavlab-mit.org
4. For access to private repositories, email chbenj36@gmail.com

## 🚦 Current Status

This project is under active development. Documentation is continuously being updated based on field experience and community feedback.

---

**Ready to get started?** Head to the [Installation Guide](installation.md) to begin setting up your system!
