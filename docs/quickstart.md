# Quick Start Guide

Get your Project Skywalker UAV system up and running quickly with this streamlined guide.

## Prerequisites

Before starting, ensure you have:
- ✅ Completed the [Installation Guide](installation.md)
- ✅ Raspberry Pi with MOOS-IvP installed
- ✅ CubeOrange autopilot with ArduPilot firmware
- ✅ Basic understanding of MOOS-IvP concepts

## Step 1: Verify Installation

SSH into your Raspberry Pi:

```bash
ssh skywalker1
```

Verify MOOS-IvP is installed:

```bash
which MOOSDB
which pAntler
```

Both commands should return paths to the executables.

## Step 2: Check System Status

### Verify Core Components

```bash
# Check MOOS-IvP build
ls ~/moos-ivp/bin/

# Check UAV-specific applications
ls ~/moos-ivp-uav-base/bin/

# Check MAVSDK installation
ls ~/moos-ivp-uav-base/MAVSDK/build/default/
```

### Test MOOSDB

Start a MOOS database instance:

```bash
MOOSDB
```

You should see output indicating the database is running. Press `Ctrl+C` to stop.

## Step 3: Hardware Connections

### Physical Setup

1. **Connect CubeOrange to Raspberry Pi**
   - Use USB cable from CubeOrange's USB port to Raspberry Pi USB port
   - Autopilot will appear as `/dev/ttyACM0` or similar

2. **Verify Serial Connection**
   ```bash
   ls /dev/tty*
   ```
   Look for `/dev/ttyACM0` or `/dev/ttyUSB0`

3. **Test Connection with MAVProxy** (Optional)
   ```bash
   # Install MAVProxy if not already installed
   pip3 install MAVProxy --user
   
   # Connect to autopilot
   python3 -m MAVProxy.mavproxy --master=/dev/ttyACM0,115200
   ```
   
   You should see heartbeat messages and firmware version.
   Press `Ctrl+C` to exit.

## Step 4: Run a Basic MOOS Mission

### Create a Test Mission Directory

```bash
mkdir -p ~/missions/test_mission
cd ~/missions/test_mission
```

### Create a Simple MOOS Configuration

Create a file `test.moos`:

```bash
cat > test.moos << 'EOF'
ServerHost = localhost
ServerPort = 9000
Community  = skywalker1

ProcessConfig = ANTLER
{
  MSBetweenLaunches = 200
  
  Run = MOOSDB          @ NewConsole = false
  Run = uTimerScript    @ NewConsole = false
}

ProcessConfig = uTimerScript
{
  AppTick   = 4
  CommsTick = 4
  
  paused = false
  reset_max = unlimited
  
  event = var=TEST_VAR, val="Hello from Skywalker", time=0
  event = var=TEST_VAR, val="System running", time=2
  event = var=TEST_VAR, val="All systems nominal", time=4
}
EOF
```

### Launch the Mission

```bash
pAntler test.moos
```

You should see:
- MOOSDB starting
- uTimerScript launching
- Application output

Press `Ctrl+C` to stop.

### Monitor MOOS Variables

In another terminal:

```bash
uPokeDB test.moos MOOS_MANUAL_OVERRIDE=false
uPokeDB test.moos DB_CLIENTS
```

## Step 5: Basic UAV Integration (Simulation)

### Connect to Autopilot

Create a mission configuration that connects to the CubeOrange:

```bash
cat > uav_test.moos << 'EOF'
ServerHost = localhost
ServerPort = 9000
Community  = skywalker1

ProcessConfig = ANTLER
{
  MSBetweenLaunches = 200
  
  Run = MOOSDB          @ NewConsole = false
}

// Additional UAV-specific processes would go here
// See full examples in moos-ivp-uav-base repository
EOF
```

**Note**: Full UAV integration requires mission files from the private `moos-ivp-uav-base` repository. Contact chbenj36@gmail.com for access and complete examples.

## Step 6: Pre-Flight Checklist

Before your first real flight:

### Software Checklist
- [ ] MOOS-IvP installed and verified
- [ ] ArduPilot firmware flashed on CubeOrange
- [ ] Radio links configured (see [Radio Setup](radio-setup.md))
- [ ] Ground control station configured
- [ ] Mission files tested in simulation

### Hardware Checklist
- [ ] Battery fully charged
- [ ] Propeller secure
- [ ] Control surfaces moving correctly
- [ ] GPS lock acquired
- [ ] Radio link established
- [ ] Raspberry Pi powered and communicating
- [ ] All connections secure

### Safety Checklist
- [ ] Flight area surveyed and clear
- [ ] Weather conditions acceptable
- [ ] Emergency procedures reviewed
- [ ] Spotter assigned (if required)
- [ ] Airspace authorization obtained (if required)

## Common Commands

### MOOS-IvP Utilities

```bash
# Launch a mission
pAntler mission.moos

# Monitor MOOS variables
uPokeDB mission.moos MOOS_MANUAL_OVERRIDE=false

# View all variables
uXMS mission.moos

# Check MOOS database
uMS mission.moos

# Query specific variable
uPokeDB mission.moos "VAR_NAME?"
```

### System Utilities

```bash
# Check Raspberry Pi resources
htop

# Monitor disk space
ncdu ~

# View system logs
journalctl -f

# Check network connections
ifconfig
```

## Troubleshooting Quick Fixes

### MOOSDB Won't Start
```bash
# Check if port is already in use
netstat -tuln | grep 9000

# Kill existing MOOS processes
killall MOOSDB pAntler
```

### Can't Find Serial Device
```bash
# List all serial devices
ls -l /dev/tty*

# Check for USB devices
lsusb

# Verify user permissions
sudo usermod -a -G dialout $USER
# Then log out and back in
```

### No Connection to Autopilot
```bash
# Test with MAVProxy
python3 -m MAVProxy.mavproxy --master=/dev/ttyACM0,115200

# Check autopilot power
# Check USB cable connection
# Verify correct baud rate (usually 115200 or 57600)
```

## Next Steps

Now that you have the basics working:

1. **[Usage Guide](usage.md)** - Learn detailed system operation
2. **[Radio Setup](radio-setup.md)** - Configure mesh networking
3. **[Firmware Guide](firmware.md)** - Advanced autopilot configuration
4. **[Hardware Assembly](hardware.md)** - Complete physical integration

## Quick Reference

### File Locations

```
~/moos-ivp/                  # MOOS-IvP core
~/moos-ivp-uav-base/         # UAV-specific MOOS apps
~/uav-common/                # Common utilities and scripts
~/missions/                  # Your mission files (create as needed)
```

### Important Ports

- MOOSDB: Port 9000 (default, configurable)
- MAVLink: Serial /dev/ttyACM0 or /dev/ttyUSB0
- RFD900x: Serial /dev/ttyUSB* (when USB-to-serial adapter used)

### Key Concepts

**MOOSDB**: Central publish-subscribe database for all MOOS applications
**pAntler**: MOOS mission launcher and process manager
**Community**: Named group of MOOS processes sharing a database
**MAVLink**: Protocol for communicating with ArduPilot autopilot

## Getting Help

- 📖 Check [Troubleshooting Guide](troubleshooting.md)
- 🐛 Report issues on [GitHub](https://github.com/pavlab-mit/project-skywalker/issues)
- 📧 Contact team@pavlab-mit.org
- 💬 Access to private repos: chbenj36@gmail.com

---

**Previous**: [Installation Guide](installation.md) | **Next**: [Usage Guide](usage.md)
