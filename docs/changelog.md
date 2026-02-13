# Changelog

All notable changes to the Project Skywalker documentation and guides will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Planned
- Video streaming guide for high-bandwidth mesh
- Advanced mission planning tutorials
- Swarm coordination examples
- Hardware integration testing checklist
- Performance tuning guide
- Custom behavior development tutorial

## [2024-11-23] - Major Documentation Update

### Added
- **Comprehensive README.md** with project overview, badges, and quick start
- **Complete documentation site** in `docs/` directory:
  - `index.md` - Documentation home with navigation
  - `installation.md` - Detailed Raspberry Pi and MOOS-IvP setup guide
  - `quickstart.md` - Quick start guide for rapid deployment
  - `usage.md` - Detailed usage patterns and mission operations
  - `architecture.md` - System architecture and design overview
  - `hardware.md` - Physical assembly and hardware integration
  - `radio-setup.md` - RFD900x and Wi-Fi mesh configuration
  - `firmware.md` - ArduPilot firmware flashing and configuration
  - `development.md` - Development environment and workflow
  - `contributing.md` - Contribution guidelines and process
  - `troubleshooting.md` - Common issues and solutions
  - `authors.md` - Contributors and acknowledgements
  - `changelog.md` - This file

### Changed
- Expanded existing `docs/index.md` with comprehensive navigation
- Updated `docs/_config.yaml` for GitHub Pages compatibility

### Preserved
- All content in `Info/` directory maintained:
  - Design decisions documentation
  - Software installation guides
  - Physical assembly notes
  - STL files for 3D printing

## [2024-11-XX] - Initial Repository

### Added
- Basic README.md with project description
- `Info/` directory structure with technical guides:
  - `design_decisions/UAV_Swarm_Mesh_Guide.md`
  - `design_decisions/telemetry_vs_wi_fi_comparison.md`
  - `software_installation/Raspberry_Pi_MOOS-IvP_Setup.md`
  - `software_installation/RFD900x_Asynchronous_Mesh_Setup_Guide.md`
  - `software_installation/custom_firmware_flash_steps.md`
  - `software_installation/firmware_version_check.md`
  - `physical_assembly/ReadMe.md`
  - `stl_files/ReadMe.md`
- Basic `docs/` directory with minimal content
- Jekyll configuration for GitHub Pages

## Documentation Guidelines

When updating this changelog:

### Types of Changes
- **Added** - New features, documents, or guides
- **Changed** - Changes to existing content
- **Deprecated** - Soon-to-be removed features
- **Removed** - Removed features or documents
- **Fixed** - Bug fixes or corrections
- **Security** - Security-related changes

### Format
```markdown
## [YYYY-MM-DD] - Release Name

### Added
- Brief description of what was added

### Changed
- Brief description of what was changed

### Fixed
- Brief description of what was fixed
```

## Future Releases

### Planned Features

**Short Term (1-3 months)**
- [ ] Video tutorials for hardware assembly
- [ ] Example mission files repository
- [ ] Automated testing procedures
- [ ] Flight test checklist and logs
- [ ] Parameter tuning guide for X8

**Medium Term (3-6 months)**
- [ ] Multi-UAV coordination examples
- [ ] Advanced mesh networking configurations
- [ ] Custom behavior library
- [ ] Integration with external sensors
- [ ] Ground control station setup guide

**Long Term (6+ months)**
- [ ] Simulation environment setup
- [ ] Machine learning integration examples
- [ ] Advanced autonomy behaviors
- [ ] Research collaboration guidelines
- [ ] Academic paper template

## Contributing to Changelog

When making changes to the repository:

1. Update this changelog with your changes
2. Use the format shown above
3. Place new entries under [Unreleased]
4. Move to dated section when released
5. Link to relevant issues/PRs if applicable

Example:
```markdown
### Added
- New troubleshooting section for GPS issues ([#123])

[#123]: https://github.com/pavlab-mit/project-skywalker/issues/123
```

## Version Numbering

This documentation follows calendar-based versioning:
- Format: `[YYYY-MM-DD]`
- Major updates get their own dated section
- Minor updates can be grouped

## Contact

For questions about changes or to suggest updates:
- **Email**: team@pavlab-mit.org
- **Issues**: https://github.com/pavlab-mit/project-skywalker/issues

## Links

- [README](../README.md)
- [Documentation Home](index.md)
- [Contributing Guide](contributing.md)
- [Authors](authors.md)

---

**Previous**: [Authors](authors.md) | **Back to**: [Documentation Home](index.md)
