# Usage Guide

This guide covers detailed usage patterns for operating the Project Skywalker UAV system with MOOS-IvP.

## Overview

The Project Skywalker system combines several software components:
- **MOOS-IvP**: Autonomy framework running on Raspberry Pi
- **ArduPilot**: Flight control software on CubeOrange
- **MAVSDK**: Communication library between MOOS and ArduPilot
- **Radio Links**: RFD900x and/or Wi-Fi mesh for telemetry and coordination

## Mission Workflow

A typical mission follows this workflow:

```
1. Plan Mission → 2. Configure → 3. Pre-flight Check → 
4. Launch → 5. Monitor → 6. Land → 7. Debrief
```

## 1. Mission Planning

### Mission File Structure

MOOS missions use `.moos` configuration files:

```
mission_name.moos
├── ServerHost/Port     # MOOSDB connection
├── Community           # UAV identifier
├── ProcessConfig       # ANTLER (process launcher)
└── ProcessConfig       # Individual MOOS apps
```

### Basic Mission Template

```moos
ServerHost = localhost
ServerPort = 9000
Community  = skywalker1

//------------------------------------------
// ANTLER Configuration
ProcessConfig = ANTLER
{
  MSBetweenLaunches = 200
  
  Run = MOOSDB          @ NewConsole = false
  Run = pLogger         @ NewConsole = false
  Run = pNodeReporter   @ NewConsole = false
  Run = pHelmIvP        @ NewConsole = false
  // Add UAV-specific apps here
}

//------------------------------------------
// pLogger - Data logging
ProcessConfig = pLogger
{
  AppTick   = 10
  CommsTick = 10
  
  Path      = ./logs/
  File      = skywalker1
  
  SyncLog   = true @ 0.2
  AsyncLog  = true
}

//------------------------------------------
// pNodeReporter - Vehicle state reporting
ProcessConfig = pNodeReporter
{
  AppTick   = 2
  CommsTick = 2
  
  platform_type = UAV
  platform_length = 2.12  // Skywalker X8 wingspan in meters
}

//------------------------------------------
// pHelmIvP - Behavior-based autonomy
ProcessConfig = pHelmIvP
{
  AppTick   = 4
  CommsTick = 4
  
  behaviors  = behaviors.bhv
  domain     = course:0:359:360
  domain     = speed:0:25:26
  domain     = depth:0:500:501  // Altitude in meters
}
```

### Behavior Files

Behaviors define autonomous actions. Example `behaviors.bhv`:

```
//------------------------------------------
// Loiter Behavior
initialize DEPLOY = false
initialize RETURN = false

Behavior = BHV_Loiter
{
  name      = loiter
  priority  = 100
  condition = DEPLOY = true
  condition = RETURN = false
  
  center_activate = true
  speed = 15
  radius = 100
  nm_radius = 100
  polygon = radial::x=0,y=0,radius=100,pts=8
}

//------------------------------------------
// Return Home Behavior
Behavior = BHV_Waypoint
{
  name      = return
  priority  = 100
  condition = RETURN = true
  
  speed = 15
  radius = 10.0
  points = 0,0
}
```

**Note**: Complete behavior examples and UAV-specific configurations are available in the private `moos-ivp-uav-base` repository. Contact chbenj36@gmail.com for access.

## 2. System Configuration

### Raspberry Pi Configuration

Edit `/boot/config.txt` for hardware interfaces:

```bash
# Enable UART for additional serial devices
enable_uart=1

# Disable Bluetooth to free up primary UART
dtoverlay=disable-bt
```

Reboot after changes:
```bash
sudo reboot
```

### Environment Variables

Add to `~/.bashrc`:

```bash
# MOOS-IvP paths
export PATH=$PATH:~/moos-ivp/bin
export PATH=$PATH:~/moos-ivp-uav-base/bin

# IvP behavior library
export IVP_BEHAVIOR_DIRS=$IVP_BEHAVIOR_DIRS:~/moos-ivp-uav-base/lib
```

### ArduPilot Parameters

Key parameters to configure in ArduPilot (via QGroundControl or MAVProxy):

```
# Serial ports
SERIAL2_PROTOCOL = 2      # MAVLink 2
SERIAL2_BAUD = 115200     # Raspberry Pi connection

# Failsafe
FS_SHORT_ACTN = 0         # Continue mission
FS_LONG_ACTN = 1          # RTL (Return to Launch)
FS_GCS_ENABL = 1          # GCS failsafe enabled

# Geofence (optional but recommended)
FENCE_ENABLE = 1
FENCE_TYPE = 6            # Min/max altitude + circle
FENCE_RADIUS = 1000       # meters
FENCE_ALT_MAX = 400       # meters (check local regulations)
```

## 3. Pre-Flight Operations

### System Startup Sequence

```bash
# 1. Power on Raspberry Pi and wait for boot
# 2. SSH into Raspberry Pi
ssh skywalker1

# 3. Navigate to mission directory
cd ~/missions/current_mission/

# 4. Verify autopilot connection
ls /dev/ttyACM0
# or test with MAVProxy
python3 -m MAVProxy.mavproxy --master=/dev/ttyACM0,115200

# 5. Check system resources
htop  # Verify CPU/memory
df -h # Check disk space

# 6. Launch MOOS mission
pAntler mission.moos
```

### Pre-Flight Checklist

Run automated checks:

```bash
# Check MOOS processes
uMS mission.moos

# Verify vehicle state
uPokeDB mission.moos NAV_X?
uPokeDB mission.moos NAV_Y?
uPokeDB mission.moos NAV_HEADING?

# Check autopilot status via MAVLink
# (requires UAV MOOS apps running)
```

## 4. In-Flight Operations

### Monitoring

**From Ground Station:**

Connect QGroundControl or another GCS via:
- RFD900x telemetry link
- Wi-Fi mesh connection
- Direct radio link

**MOOS Monitoring Tools:**

```bash
# Real-time variable viewer
uXMS mission.moos

# Log viewer (post-flight or during mission)
alogview logs/

# Specific variable monitoring
uPokeDB mission.moos DB_CLIENTS
uPokeDB mission.moos NAV_SPEED
uPokeDB mission.moos NAV_ALT
```

### Commanding

**Deploy Mission:**
```bash
uPokeDB mission.moos DEPLOY=true
```

**Return to Launch:**
```bash
uPokeDB mission.moos RETURN=true
```

**Emergency Stop (Kill switch):**
```bash
uPokeDB mission.moos MOOS_MANUAL_OVERRIDE=true
```

**Custom Commands:**
```bash
# Set waypoint
uPokeDB mission.moos GOTO_X=100
uPokeDB mission.moos GOTO_Y=200

# Adjust speed
uPokeDB mission.moos DESIRED_SPEED=15
```

## 5. Multi-Vehicle Coordination

### Mesh Networking

For swarm operations, vehicles communicate via mesh:

**RFD900x Mesh (Long-range, low-bandwidth)**
- Node ID: Unique per vehicle (configured in radio)
- Destination ID: 65535 (broadcast)
- Range: 20-40+ km
- Use for: Heartbeats, state sharing, coordination commands

**Wi-Fi Mesh (High-bandwidth)**
- OLSR or BATMAN-adv routing protocol
- Range: 0.5-15 km depending on radios and antennas
- Use for: Video, maps, high-rate data

See [Radio Setup Guide](radio-setup.md) for configuration details.

### MOOS Inter-Vehicle Communication

Vehicles share variables via `pShare` and `pNodeReporter`:

```moos
ProcessConfig = pShare
{
  AppTick   = 2
  CommsTick = 2
  
  input = route = localhost:9301
  output = src_name=NODE_REPORT_LOCAL, dest_name=NODE_REPORT, route=multicast_8
}

ProcessConfig = pNodeReporter
{
  AppTick   = 2
  CommsTick = 2
  
  vessel_type = UAV
}
```

Vehicles automatically share:
- Position (NAV_X, NAV_Y, NAV_Z)
- Heading (NAV_HEADING)
- Speed (NAV_SPEED)
- Status variables

## 6. Data Logging and Analysis

### Log Files

MOOS automatically logs data to `.alog` files:

```bash
# View logs
alogview logs/skywalker1_20241123.alog

# Convert to CSV
alog2csv logs/skywalker1_20241123.alog

# Extract specific variables
aloggrep logs/skywalker1_20241123.alog NAV_X NAV_Y > position.txt
```

### Log Analysis

```bash
# Generate log summary
alogsummary logs/skywalker1_20241123.alog

# Plot data (requires matplotlib)
alogplot logs/skywalker1_20241123.alog NAV_X NAV_Y
```

### ArduPilot Logs

Download from CubeOrange:
- Use QGroundControl log download feature
- Or copy directly from SD card
- Analyze with Mission Planner or UAV Log Viewer

## 7. Common Operations

### Change Mission Parameters

While mission is running:

```bash
# Change behavior parameters
uPokeDB mission.moos LOITER_SPEED=18
uPokeDB mission.moos LOITER_RADIUS=150

# Update waypoints dynamically
uPokeDB mission.moos WAYPOINT_UPDATES=points=100,200:150,300
```

### Emergency Procedures

**Lost Link:**
- ArduPilot RTL will activate after timeout
- MOOS continues autonomous behaviors
- Regain link and assess situation

**Manual Override:**
```bash
uPokeDB mission.moos MOOS_MANUAL_OVERRIDE=true
# Then use RC transmitter for manual control
```

**Abort Mission:**
```bash
killall pAntler
# This stops all MOOS processes
# ArduPilot continues with configured failsafe behavior
```

## Configuration Files Reference

### Key Configuration Paths

```
~/missions/                          # Your mission files
~/.bashrc                           # Environment setup
~/moos-ivp/missions/examples/       # Example missions
~/moos-ivp-uav-base/missions/       # UAV-specific examples
```

### Environment Files

**~/.bashrc additions:**
```bash
# MOOS-IvP environment
source ~/uav-common/bashrc_common.sh

# Convenience aliases
alias goto_missions='cd ~/missions/'
alias launch='pAntler mission.moos'
alias watch_logs='tail -f logs/*.alog'
```

## Best Practices

### Safety
- ✅ Always test in simulation first
- ✅ Verify geofence is configured
- ✅ Test failsafe behaviors
- ✅ Have manual override ready
- ✅ Monitor battery voltage continuously

### Performance
- ✅ Keep MOOS apps lightweight (low AppTick if possible)
- ✅ Log only necessary variables
- ✅ Minimize network traffic over radio links
- ✅ Use async logging for high-rate data

### Reliability
- ✅ Use SD cards with good endurance
- ✅ Implement watchdog timers
- ✅ Test radio links at operating distances
- ✅ Regular system updates and testing
- ✅ Maintain hardware (connectors, antennas, power systems)

## Next Steps

- **[Architecture Guide](architecture.md)** - Understand system design
- **[Radio Setup](radio-setup.md)** - Configure communication systems
- **[Troubleshooting](troubleshooting.md)** - Solve common problems
- **[Development Guide](development.md)** - Create custom behaviors

## Additional Resources

- [MOOS-IvP Documentation](https://oceanai.mit.edu/moos-ivp/)
- [ArduPilot Plane Documentation](https://ardupilot.org/plane/)
- Example missions: `~/moos-ivp-uav-base/missions/` (private repository)

---

**Previous**: [Quick Start](quickstart.md) | **Next**: [Architecture Guide](architecture.md)
