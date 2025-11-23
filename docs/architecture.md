# System Architecture

This document describes the architecture of the Project Skywalker UAV system, including hardware components, software stack, and repository organization.

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Ground Station                        │
│         (QGroundControl / MOOS Shore-Side)              │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ Radio Link (RFD900x / Wi-Fi)
                 │
┌────────────────┴────────────────────────────────────────┐
│                  Skywalker X8 UAV                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │         Raspberry Pi 4B (Companion Computer)     │   │
│  │  ┌────────────────────────────────────────┐     │   │
│  │  │          MOOS-IvP Stack                 │     │   │
│  │  │  - MOOSDB (pub/sub database)           │     │   │
│  │  │  - pHelmIvP (autonomy engine)          │     │   │
│  │  │  - Custom UAV behaviors                │     │   │
│  │  │  - pShare (inter-vehicle comms)        │     │   │
│  │  └────────────────────────────────────────┘     │   │
│  │            │ MAVLink (MAVSDK)                    │   │
│  └────────────┼─────────────────────────────────────┘   │
│               │ Serial (USB)                             │
│  ┌────────────┴─────────────────────────────────────┐   │
│  │      CubeOrange+ (Autopilot)                     │   │
│  │  - ArduPilot firmware                            │   │
│  │  - Flight control                                │   │
│  │  - Sensor fusion (GPS, IMU, compass, etc.)      │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Communication:                                          │
│  - RFD900x (long-range telemetry)                       │
│  - Rocket M5 (high-bandwidth mesh, optional)            │
└──────────────────────────────────────────────────────────┘
```

## Hardware Architecture

### Primary Components

#### 1. Skywalker X8 Airframe
- **Type**: Fixed-wing UAV
- **Wingspan**: 2.12 meters
- **Material**: EPO foam
- **Payload Capacity**: ~1 kg
- **Flight Time**: 2+ hours (depending on payload and battery)
- **Advantages**: Durable, easy to repair, good endurance

#### 2. CubeOrange+ Autopilot
- **Processor**: STM32H757
- **Sensors**: 
  - Triple redundant IMUs
  - Magnetometers
  - Barometers
- **Interfaces**: 
  - Multiple UARTs for telemetry
  - I2C, SPI buses
  - PWM outputs for servos
- **Software**: ArduPilot Plane
- **Power**: 5V from power module

#### 3. Raspberry Pi 4B
- **Processor**: Quad-core ARM Cortex-A72 @ 1.5GHz
- **RAM**: 4GB or 8GB
- **Storage**: MicroSD card (32GB+ recommended)
- **OS**: Raspberry Pi OS Lite (64-bit)
- **Purpose**: High-level autonomy (MOOS-IvP)
- **Power**: 5V USB-C (from battery via regulator)

#### 4. Communication Hardware

**RFD900x Telemetry Radio**
- **Frequency**: 900 MHz (sub-GHz)
- **Range**: 20-40+ km line-of-sight
- **Bandwidth**: Tens of kbps
- **Mode**: Asynchronous mesh
- **Purpose**: Long-range C2 and telemetry

**Rocket M5 (Optional)**
- **Frequency**: 5.8 GHz
- **Range**: 0.5-15 km (depending on antennas)
- **Bandwidth**: Tens to hundreds of Mbps
- **Purpose**: High-bandwidth mesh for video/data
- **Protocols**: OLSR or BATMAN-adv for mesh routing

### 3D-Printed Components

Custom enclosures and mounts (STL files in repository):
- Raspberry Pi case with port access
- Radio mounting brackets (in development)
- Antenna mounts

## Software Architecture

### Layer 1: Flight Control (CubeOrange)

**ArduPilot**
- Low-level flight control
- Sensor fusion and state estimation
- Safety features (geofence, failsafe)
- Waypoint navigation
- Controlled via MAVLink protocol

**Configuration**: Via QGroundControl or MAVProxy

### Layer 2: Autonomy Framework (Raspberry Pi)

**MOOS-IvP Core**
- `MOOSDB`: Central publish-subscribe database
- `pHelmIvP`: Behavior-based decision engine using Interval Programming
- `pLogger`: Data logging
- `pNodeReporter`: Vehicle state reporting
- `pShare`: Inter-vehicle communication

**Custom UAV Applications**
- MAVLink bridge (MAVSDK-based)
- UAV-specific behaviors
- Mission management
- Swarm coordination

### Layer 3: Communication

**MAVLink (Raspberry Pi ↔ CubeOrange)**
- Protocol: MAVLink v2
- Transport: Serial USB (115200 baud typical)
- Library: MAVSDK (C++)
- Purpose: Send commands, receive telemetry

**MOOS Inter-Process (On Raspberry Pi)**
- Protocol: MOOS publish-subscribe
- Transport: Shared memory / local sockets
- Purpose: Communication between MOOS apps

**Inter-Vehicle (UAV ↔ UAV)**
- Protocol: MOOS messages over UDP
- Transport: RFD900x mesh (via PPP/SLIP) or Wi-Fi mesh (native IP)
- Purpose: Swarm coordination, state sharing

**Ground Link (UAV ↔ Ground Station)**
- MAVLink: Via RFD900x or Wi-Fi (for QGroundControl)
- MOOS: Via pShare over network (for shore-side MOOS)

## Software Components

### Repository Structure

```
project-skywalker/
├── README.md                    # Main project README
├── docs/                        # Documentation site
│   ├── index.md                 # Documentation home
│   ├── installation.md          # Setup guide
│   ├── quickstart.md            # Quick start
│   ├── usage.md                 # Detailed usage
│   ├── architecture.md          # This file
│   ├── hardware.md              # Hardware assembly
│   ├── radio-setup.md           # Radio configuration
│   ├── firmware.md              # Firmware flashing
│   ├── development.md           # Development guide
│   ├── troubleshooting.md       # Problem solving
│   ├── contributing.md          # Contribution guide
│   ├── authors.md               # Credits
│   └── changelog.md             # Version history
│
└── Info/                        # Detailed technical guides
    ├── design_decisions/        # Architecture decisions
    │   ├── UAV_Swarm_Mesh_Guide.md
    │   └── telemetry_vs_wi_fi_comparison.md
    │
    ├── software_installation/   # Installation guides
    │   ├── Raspberry_Pi_MOOS-IvP_Setup.md
    │   ├── RFD900x_Asynchronous_Mesh_Setup_Guide.md
    │   ├── custom_firmware_flash_steps.md
    │   └── firmware_version_check.md
    │
    ├── physical_assembly/       # Hardware assembly
    │   └── ReadMe.md
    │
    └── stl_files/               # 3D printable parts
        └── ReadMe.md
```

### External Repositories

**Public:**
- `moos-ivp/moos-ivp`: Core MOOS-IvP framework

**Private (Request Access):**
- `cbenjamin23/uav-common`: Common utilities and scripts
- `cbenjamin23/moos-ivp-uav-base`: UAV-specific MOOS applications
- `pavlab-MIT/moos-ivp-swarm`: Swarm coordination behaviors
- Contact: chbenj36@gmail.com

### Dependencies

**On Raspberry Pi:**
```
System:
- Raspberry Pi OS Lite (64-bit)
- build-essential, cmake, git
- Python 3

MOOS-IvP:
- SVN (for some legacy dependencies)
- X11 libraries (minimal, for some tools)

UAV Specific:
- MAVSDK (included as submodule)
- MAVProxy (optional, for testing)
```

**On Development Machine:**
```
- SSH client
- Git
- Text editor
- QGroundControl (ground station software)
- Optional: MOOS-IvP (for simulation and shore-side)
```

## Data Flow

### Typical Mission Data Flow

```
1. Mission Planning (Ground)
   └─> Upload mission via QGC or MOOS interface

2. Pre-Flight (Ground + UAV)
   └─> MOOS validates mission
   └─> ArduPilot confirms readiness

3. Launch (UAV)
   └─> Manual takeoff or auto-takeoff
   └─> Transition to MOOS autonomous control

4. In-Flight (UAV)
   ┌─> Sensors → ArduPilot → State Estimation
   │   └─> MAVLink → Raspberry Pi
   │       └─> MOOSDB → pHelmIvP
   │           └─> Behaviors evaluate
   │               └─> Decisions → Commands
   │                   └─> MAVLink → ArduPilot
   │                       └─> Control Surfaces + Motor
   │
   └─> Telemetry → Radio → Ground Station

5. Landing (UAV)
   └─> MOOS triggers return behavior
   └─> ArduPilot executes landing sequence

6. Post-Flight (Ground)
   └─> Download logs
   └─> Analysis and debrief
```

### Inter-Vehicle Communication

```
UAV-1                           UAV-2
  │                               │
  │ State (pos, heading, etc.)    │
  ├──────── Mesh Network ─────────>
  │        (RFD900x/Wi-Fi)        │
  │                               │
  │      Coordination msgs        │
  <────────────────────────────────┤
  │                               │
  └─> MOOSDB updates              └─> MOOSDB updates
      └─> Behaviors adapt             └─> Behaviors adapt
```

## Communication Protocols

### MAVLink
- **Purpose**: Raspberry Pi ↔ Autopilot
- **Version**: MAVLink v2
- **Messages Used**:
  - HEARTBEAT
  - GPS_RAW_INT
  - ATTITUDE
  - POSITION_TARGET_LOCAL_NED
  - COMMAND_LONG
  - More...

### MOOS Messages
- **Purpose**: Inter-process and inter-vehicle
- **Format**: Key-value pairs
- **Transport**: UDP multicast or unicast
- **Examples**:
  - `NAV_X`, `NAV_Y`, `NAV_Z`: Position
  - `NAV_HEADING`: Heading in degrees
  - `NAV_SPEED`: Speed in m/s
  - `DEPLOY`, `RETURN`: Mission commands
  - Custom application variables

## Network Architecture

### Single Vehicle (Basic)

```
Ground Station ←─ RFD900x ─→ UAV
    (GCS)                    (Telemetry + Control)
```

### Multi-Vehicle Swarm

```
                    ┌─ UAV-1 ─┐
                    │         │
Ground Station ─────┤  Mesh   ├───── UAV-2
    (GCS)           │ Network │
                    │         │
                    └─ UAV-3 ─┘
```

**Mesh Options:**
1. **RFD900x**: PPP over serial + BATMAN classic (Layer 3)
2. **Wi-Fi**: OLSR or BATMAN-adv (Layer 2/3)
3. **Hybrid**: Both for redundancy

## Processing Allocation

### CubeOrange (ArduPilot)
- ✅ Real-time flight control (fast, deterministic)
- ✅ Sensor fusion
- ✅ Safety monitoring
- ✅ Basic navigation (waypoints, RTL)
- ❌ NOT for: Complex decision-making, vision processing

### Raspberry Pi (MOOS-IvP)
- ✅ High-level autonomy and behaviors
- ✅ Mission planning and replanning
- ✅ Swarm coordination
- ✅ Data logging and processing
- ⚠️ NOT real-time: ~1-10 Hz typical update rates

## Scalability

### Single UAV
- Simple: Raspberry Pi + CubeOrange
- One radio for ground link

### Small Swarm (2-5 UAVs)
- RFD900x mesh or Wi-Fi mesh
- Simple coordination (formation, leader-follower)
- Centralized or distributed autonomy

### Medium Swarm (5-20 UAVs)
- Wi-Fi mesh preferred (higher bandwidth)
- Hierarchical coordination
- Edge computing on ground station

### Large Swarm (20+ UAVs)
- Hierarchical mesh architecture
- Multiple frequency bands
- Distributed decision-making required
- Ground infrastructure (base stations)

## Design Decisions

Key architectural choices and rationale:

### Why MOOS-IvP?
- ✅ Proven in marine robotics (MIT)
- ✅ Behavior-based: easy to add/modify behaviors
- ✅ Interval Programming: handles uncertainty
- ✅ Modular: easy to test and debug
- ✅ Multi-agent capable

### Why Raspberry Pi?
- ✅ Sufficient compute for MOOS-IvP
- ✅ Linux ecosystem
- ✅ Easy development and debugging
- ✅ Low cost
- ✅ Community support

### Why RFD900x?
- ✅ Long range (20-40+ km)
- ✅ Asynchronous mesh capable
- ✅ Built-in encryption
- ✅ Proven in UAV applications

### Why ArduPilot?
- ✅ Open source
- ✅ Mature and stable
- ✅ Extensive feature set
- ✅ Large community
- ✅ Well-documented MAVLink interface

## Security Considerations

- **Radio Encryption**: RFD900x has built-in AES encryption
- **Wi-Fi**: WPA2/WPA3 + VPN (WireGuard recommended)
- **SSH**: Key-based authentication only (no passwords in production)
- **Geofence**: Always configured to prevent runaway
- **Failsafe**: Multiple layers (GCS link, GPS, low battery)

## Performance Characteristics

### Typical Update Rates
- ArduPilot control loop: 50-400 Hz (depends on mode)
- MOOS behaviors: 1-10 Hz
- Inter-vehicle updates: 0.5-2 Hz
- Ground telemetry: 1-4 Hz

### Latency
- Raspberry Pi to ArduPilot: <10ms
- Behavior decision to control: 100-500ms
- Inter-vehicle (mesh): 50-500ms (depends on hops)
- Ground link: 100-1000ms

### Range
- RFD900x: 20-40+ km (LOS)
- Rocket M5 with omni: 0.5-2 km
- Rocket M5 with panel: 5-15 km
- GPS: Limited only by UAV endurance

## Next Steps

- **[Hardware Guide](hardware.md)** - Physical assembly
- **[Radio Setup](radio-setup.md)** - Communication configuration
- **[Usage Guide](usage.md)** - Operating the system
- **[Development Guide](development.md)** - Creating custom behaviors

## References

- [MOOS-IvP Documentation](https://oceanai.mit.edu/moos-ivp/)
- [ArduPilot Architecture](https://ardupilot.org/dev/docs/apmcopter-programming-architecture.html)
- [MAVLink Protocol](https://mavlink.io/)

---

**Previous**: [Usage Guide](usage.md) | **Next**: [Hardware Guide](hardware.md)
