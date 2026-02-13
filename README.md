# Project Skywalker 🚁

[![License](https://img.shields.io/badge/License-Not%20Specified-lightgrey.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-GitHub%20Pages-blue.svg)](https://pavlab-mit.github.io/project-skywalker/)
[![Platform](https://img.shields.io/badge/platform-Skywalker%20X8-orange.svg)](https://github.com/pavlab-mit/project-skywalker)

**A comprehensive information hub for building and deploying autonomous UAV swarms using MOOS-IvP on the Skywalker X8 platform.**

This repository serves as the central documentation and knowledge base for all aspects of working with MOOS-IvP (Mission Oriented Operating Suite - Interval Programming) on the Skywalker X8 fixed-wing UAV platform. It includes detailed guides on hardware assembly, software installation, radio configuration, mesh networking, and mission deployment.

## 🎯 Project Overview

Project Skywalker enables research and development of autonomous multi-agent UAV systems using:
- **Platform**: Skywalker X8 fixed-wing UAV
- **Autopilot**: CubeOrange/CubeOrange+ running ArduPilot
- **Computing**: Raspberry Pi running MOOS-IvP
- **Communication**: RFD900x telemetry radios and/or Wi-Fi mesh (Rocket M5)
- **Software**: MOOS-IvP framework for autonomous behavior and coordination

## 🚀 Quick Start

### Prerequisites
- Skywalker X8 airframe
- CubeOrange or CubeOrange+ autopilot
- Raspberry Pi (4B recommended)
- RFD900x telemetry radio (optional: Rocket M5 for high-bandwidth mesh)
- macOS, Linux, or Windows development machine

### Getting Started

1. **Clone this repository**
   ```bash
   git clone https://github.com/pavlab-mit/project-skywalker.git
   cd project-skywalker
   ```

2. **Follow the setup guides**
   - Start with [Installation Guide](docs/installation.md) for setting up your Raspberry Pi
   - See [Hardware Assembly](docs/hardware.md) for physical integration
   - Configure radios with [Radio Setup Guide](docs/radio-setup.md)
   - Flash autopilot firmware using [Firmware Guide](docs/firmware.md)

3. **Explore the documentation**
   - Browse the [comprehensive documentation](docs/index.md)
   - Check the [Quick Start Guide](docs/quickstart.md) for rapid deployment
   - Review [Architecture](docs/architecture.md) to understand the system design

## 📚 Documentation

Full documentation is available in the [docs/](docs/) directory:

- **[Installation Guide](docs/installation.md)** - Detailed setup instructions for Raspberry Pi and MOOS-IvP
- **[Quick Start](docs/quickstart.md)** - Get up and running quickly
- **[Hardware Assembly](docs/hardware.md)** - Physical assembly and 3D-printed parts
- **[Radio Setup](docs/radio-setup.md)** - RFD900x configuration and mesh networking
- **[Firmware Guide](docs/firmware.md)** - ArduPilot firmware flashing and configuration
- **[Usage Guide](docs/usage.md)** - How to run missions and operate the system
- **[Architecture](docs/architecture.md)** - System design and repository structure
- **[Troubleshooting](docs/troubleshooting.md)** - Common issues and solutions
- **[Contributing](docs/contributing.md)** - How to contribute to the project

## 🛠️ Key Features

- **Autonomous Navigation**: MOOS-IvP behavior-based architecture for complex missions
- **Swarm Coordination**: Mesh networking for multi-UAV communication and coordination
- **Long Range**: RFD900x radios provide 20-40+ km range for telemetry
- **High Bandwidth**: Optional Wi-Fi mesh for video and high-rate data sharing
- **Flexible Platform**: Modular design supports research and operational missions

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](docs/contributing.md) for details on:
- How to submit issues and pull requests
- Development workflow and standards
- Code of conduct

For questions or access to private repositories mentioned in the guides, contact:
- Email: chbenj36@gmail.com or team@pavlab-mit.org

## 📖 Citation

If you use this work in your research, please cite:

```bibtex
@misc{project-skywalker,
  title={Project Skywalker: MOOS-IvP on Skywalker X8},
  author={PAVLab MIT},
  year={2024},
  howpublished={\url{https://github.com/pavlab-mit/project-skywalker}}
}
```

## 👥 Authors & Acknowledgements

See [Authors](docs/authors.md) for a full list of contributors and acknowledgements.

## 📄 License

License information to be specified. Please contact the maintainers for details.

## 🔗 Related Projects

- [MOOS-IvP](https://github.com/moos-ivp/moos-ivp) - Mission Oriented Operating Suite
- [ArduPilot](https://ardupilot.org/) - Open-source autopilot software
- [RFDesign](https://rfdesign.com.au/) - RFD900x telemetry radios

## 📞 Contact

- **Team**: team@pavlab-mit.org
- **Repository**: https://github.com/pavlab-mit/project-skywalker
- **Issues**: https://github.com/pavlab-mit/project-skywalker/issues

---

**Status**: Active development and documentation
