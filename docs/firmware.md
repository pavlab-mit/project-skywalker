# Firmware Guide

This guide covers flashing and configuring ArduPilot firmware on the CubeOrange/CubeOrange+ autopilot for the Skywalker X8 platform.

## Overview

The CubeOrange runs ArduPilot Plane firmware, which provides:
- Low-level flight control
- Sensor fusion and state estimation
- Waypoint navigation
- Safety features (geofence, failsafe)
- MAVLink communication interface

## Prerequisites

### Hardware
- CubeOrange or CubeOrange+ with carrier board
- USB cable (CubeOrange to computer)
- Computer (macOS, Linux, or Windows)

### Software
- QGroundControl (recommended for firmware flashing)
- Or MAVProxy (command-line alternative)

## Part 1: Install QGroundControl

### Download and Install

**macOS:**
```bash
# Download from official site
open https://qgroundcontrol.com/

# Or install via Homebrew Cask
brew install --cask qgroundcontrol
```

**Linux:**
```bash
# Download AppImage from qgroundcontrol.com
chmod +x QGroundControl.AppImage
./QGroundControl.AppImage
```

**Windows:**
- Download installer from [qgroundcontrol.com](https://qgroundcontrol.com/)
- Run installer
- Follow on-screen instructions

## Part 2: Flash ArduPilot Firmware

### Step 1: Prepare CubeOrange

1. **Disconnect Power**
   - Ensure Cube is not powered
   - Disconnect battery and other power sources

2. **Disconnect from Aircraft**
   - Optional but recommended for initial flashing
   - Prevents issues from connected peripherals

### Step 2: Enter Bootloader Mode

1. **Launch QGroundControl**

2. **Navigate to Firmware Section**
   - Click vehicle icon (top toolbar)
   - Select **"Firmware"** from left sidebar
   
3. **Connect Cube via USB**
   - Plug USB cable directly to computer (no hubs)
   - QGC should detect: "Found device: CubePilot"
   - Bootloader Version: 5
   - Board ID: 1063 (CubeOrange+) or 140 (CubeOrange)

### Step 3: Select Firmware

**Option A: Stable Release (Recommended for first-time)**

1. Enable "Advanced settings" checkbox
2. Select firmware type:
   - **ArduPilot**
   - **CubeOrange** or **CubeOrange+** (match your hardware)
   - **Plane** (not Copter or Rover)
   - **Stable** version

3. Click **"OK"** to start flashing

**Option B: Custom Firmware (Advanced)**

For specific ArduPilot versions or custom builds:

1. **Enable Advanced Settings**
   - Check ✔️ "Advanced settings"

2. **Download Firmware File**
   
   **Latest stable:**
   ```
   https://firmware.ardupilot.org/Plane/latest/CubeOrangePlus/
   ```
   
   **Specific version (e.g., 4.5.7):**
   ```
   https://firmware.ardupilot.org/Plane/stable-4.5.7/CubeOrangePlus/
   ```
   
   Download: **`arduplane.apj`**
   
   **Important**: Use `.apj` file, not `.hex`, `.elf`, or `.bin`

3. **Select Custom File**
   - Click **"Custom firmware file..."** at bottom
   - Browse to downloaded `.apj` file
   - Select and open

### Step 4: Flashing Process

QGC will:
1. Verify board ID matches firmware
2. Erase previous firmware
3. Upload new firmware  
4. Verify upload
5. Reboot Cube

**Typical output:**
```
Board ID: 1063
Successfully decompressed image
Erasing previous program...
Programming new firmware...
Verifying...
Success! Rebooting...
```

**Duration**: 1-2 minutes

### Step 5: Verify Installation

After reboot, QGC should:
- Connect to Cube automatically
- Show firmware version in status
- Display parameter editor
- Show sensor data (if connected)

**Check version via MAVProxy:**

```bash
# Install MAVProxy
pip3 install MAVProxy --user

# Connect
python3 -m MAVProxy.mavproxy --master=/dev/tty.usbmodem2101,115200

# Wait for output showing:
# AP: ArduPlane V4.6.3 (or your version)
# Board: CubeOrange (ID: 1063)
```

Exit with `Ctrl+C`

## Part 3: Basic Configuration

### Initial Setup Wizard

QGC provides a setup wizard for first-time configuration:

1. **Airframe Selection**
   - Select: **"Standard Plane"**
   - Or specific Skywalker X8 profile if available

2. **Radio Calibration**
   - Connect RC receiver
   - Follow on-screen instructions
   - Move all sticks and switches through full range
   - Verify correct channel mapping

3. **Sensor Calibration**
   - **Accelerometer**: Level calibration (follow prompts)
   - **Compass**: Rotate aircraft in all directions
   - **Level Horizon**: Place aircraft level, calibrate
   
   **Important**: Perform compass calibration away from metal objects and electronics

4. **Flight Modes**
   - Assign modes to RC switches
   - Recommended modes:
     - Manual
     - FBWA (Fly-By-Wire A)
     - Auto (for missions)
     - RTL (Return to Launch)
     - Loiter

### Key Parameters

These parameters are critical for Skywalker X8 operation:

#### Serial Ports

```
SERIAL2_PROTOCOL = 2          # MAVLink 2 (for Raspberry Pi)
SERIAL2_BAUD = 115200         # Baud rate

SERIAL1_PROTOCOL = 2          # MAVLink 2 (for telemetry radio)
SERIAL1_BAUD = 57600          # Match radio setting
```

#### Servo Configuration

```
SERVO1_FUNCTION = 4           # Aileron left
SERVO2_FUNCTION = 4           # Aileron right  
SERVO3_FUNCTION = 19          # Elevator
SERVO4_FUNCTION = 21          # Rudder
SERVO5_FUNCTION = 70          # Throttle

# Set servo limits based on testing
SERVO1_MIN = 1000
SERVO1_MAX = 2000
# Repeat for all servos
```

#### Safety and Failsafe

```
# GCS Failsafe
FS_GCS_ENABLE = 1             # Enable GCS failsafe
FS_SHORT_ACTN = 0             # Continue if short loss
FS_LONG_ACTN = 1              # RTL if long loss
FS_TIMEOUT = 5                # Timeout in seconds

# Battery Failsafe
BATT_FS_LOW_ACT = 2           # RTL on low battery
BATT_FS_CRT_ACT = 1           # Land on critical battery
BATT_LOW_VOLT = 14.4          # Low voltage (adjust for your battery)
BATT_CRT_VOLT = 13.2          # Critical voltage

# Geofence
FENCE_ENABLE = 1              # Enable geofence
FENCE_TYPE = 6                # Min/max altitude + circle
FENCE_RADIUS = 1000           # meters from home
FENCE_ALT_MAX = 400           # Max altitude (check regulations)
FENCE_ALT_MIN = 30            # Min altitude  
FENCE_ACTION = 1              # RTL if breach
```

#### Control Tuning (Starting Points)

```
# Roll Control
RLL2SRV_P = 1.0
RLL2SRV_I = 0.3
RLL2SRV_D = 0.08
RLL2SRV_IMAX = 3000

# Pitch Control
PTCH2SRV_P = 1.0
PTCH2SRV_I = 0.3
PTCH2SRV_D = 0.08
PTCH2SRV_IMAX = 3000

# Airspeed (if using airspeed sensor)
ARSPD_FBW_MIN = 12            # Min airspeed in auto modes (m/s)
ARSPD_FBW_MAX = 25            # Max airspeed (m/s)
```

**Note**: These are starting values. Fine-tune based on test flights.

### Configuration via MAVProxy (Alternative)

For command-line configuration:

```bash
# Connect
python3 -m MAVProxy.mavproxy --master=/dev/tty.usbmodem2101,115200

# View parameter
param show SERIAL2_PROTOCOL

# Set parameter
param set SERIAL2_PROTOCOL 2
param set SERIAL2_BAUD 115200

# Save to EEPROM
param save
```

## Part 4: Advanced Configuration

### Raspberry Pi Connection

Configure serial port for Raspberry Pi:

1. **Physical Connection**
   - Raspberry Pi USB → CubeOrange USB, OR
   - Raspberry Pi GPIO UART → Cube TELEM2

2. **Parameters**
   ```
   SERIAL2_PROTOCOL = 2
   SERIAL2_BAUD = 115200
   ```

3. **Test Connection**
   From Raspberry Pi:
   ```bash
   python3 -m MAVProxy.mavproxy --master=/dev/ttyACM0,115200
   ```

### Logging

```
LOG_BACKEND_TYPE = 1          # SD card logging
LOG_BITMASK = 65535           # Log everything (adjust as needed)
LOG_DISARMED = 1              # Log when disarmed (useful for debugging)
```

### Airspeed Sensor (Recommended for Fixed-Wing)

If using pitot tube:

```
ARSPD_TYPE = 1                # Enable airspeed sensor (type depends on sensor)
ARSPD_USE = 1                 # Use for control
ARSPD_AUTOCAL = 1             # Auto-calibrate
```

### GPS Configuration

```
GPS_TYPE = 1                  # Auto (most common)
GPS_AUTO_SWITCH = 1           # Auto-switch between GPS modules
GPS_BLEND_MASK = 5            # Blend GPS data if dual GPS
```

## Part 5: Testing and Validation

### Bench Testing

Before flight, verify on bench:

1. **Sensor Data**
   - GPS: Verify fix (blue LED solid on GPS module)
   - Compass: Check heading matches actual
   - Accelerometer: Level indicator correct
   - Barometer: Altitude reading reasonable

2. **Control Surface Movement**
   - Move RC sticks
   - Verify correct surface movement
   - Check direction (push nose down → elevator goes down)
   - Ensure no reversed channels

3. **Modes**
   - Switch through all flight modes
   - Verify mode changes register in QGC
   - Test failsafe (disconnect RC, should enter failsafe mode)

4. **MAVLink**
   - Verify Raspberry Pi can connect
   - Test telemetry radio link
   - Check data rates

### Pre-Flight Checks

Using QGC Pre-Flight checklist:

- ✅ Firmware version correct
- ✅ GPS lock acquired (need 6+ satellites)
- ✅ Compass calibrated and healthy
- ✅ Accelerometer calibrated
- ✅ Battery voltage good
- ✅ Radio link strong
- ✅ Control surfaces respond correctly
- ✅ Geofence configured
- ✅ Failsafe tested

### Ground Testing

1. **Control Surface Check**
   - Manual mode
   - Full stick deflection
   - Verify smooth operation
   - Check for binding or glitches

2. **Stabilization Check**
   - FBWA mode
   - Tilt aircraft
   - Verify stabilization response

3. **Throttle Check**
   - Arm system
   - Slowly advance throttle
   - Check motor response
   - Test motor cutoff (disarm)

## Part 6: Updating Firmware

### When to Update

- Security patches
- Bug fixes for issues you're experiencing
- New features needed
- Scheduled maintenance

### Update Process

1. **Backup Parameters**
   - QGC: Vehicle Setup → Parameters → Tools → Save to file
   - Or via MAVProxy: `param save /path/to/backup.param`

2. **Flash New Firmware**
   - Follow same process as initial installation
   - Use updated `.apj` file

3. **Restore Parameters**
   - QGC: Load from file
   - Or MAVProxy: `param load /path/to/backup.param`

4. **Verify**
   - Check critical parameters
   - Re-test on bench
   - Verify all functionality

### Downgrading

If new firmware has issues:

1. Download older firmware `.apj` file
2. Flash using custom firmware option
3. Restore parameter file (if compatible)
4. Re-validate

## Troubleshooting

### Firmware Won't Flash

**Board Not Detected:**
- Try different USB cable
- Try different USB port (direct, no hub)
- Restart QGC
- Manually enter bootloader (hold safety button, plug in USB)

**Board ID Mismatch:**
- Error: "140 != 1063" or similar
- Using wrong firmware for your Cube
- Download correct version (CubeOrange vs CubeOrange+)

### Compass Issues

**Pre-Arm: Compass variance:**
- Perform compass calibration
- Move away from metal/electronics
- Check for magnetic interference
- Consider disabling compass 2/3 if only one good

**Compass inconsistent:**
- Multiple compasses disagree
- Check mounting orientation
- Verify `COMPASS_ORIENT` parameters
- Disable bad compass

### GPS Issues

**No GPS Lock:**
- Ensure GPS has clear view of sky
- Wait longer (can take minutes on first boot)
- Check GPS cable connection
- Verify `GPS_TYPE` parameter

### Sensor Calibration Failed

- Ensure aircraft is stable during calibration
- Follow prompts exactly
- Keep away from metal and electronics (compass)
- Try multiple times if needed

## Next Steps

After firmware configuration:

1. **[Hardware Guide](hardware.md)** - Complete physical integration
2. **[Quick Start](quickstart.md)** - System integration
3. **[Usage Guide](usage.md)** - Mission operations

## Additional Resources

### Official Documentation
- [ArduPilot Plane Documentation](https://ardupilot.org/plane/)
- [Parameter Reference](https://ardupilot.org/plane/docs/parameters.html)
- [CubeOrange Docs](https://docs.cubepilot.org/user-guides/autopilot/the-cube-orange)

### Firmware Downloads
- [ArduPilot Firmware](https://firmware.ardupilot.org/)
- [Latest Stable](https://firmware.ardupilot.org/Plane/latest/CubeOrangePlus/)

### In This Repository
- `Info/software_installation/custom_firmware_flash_steps.md` - Detailed flash procedure
- `Info/software_installation/firmware_version_check.md` - How to check version

### Community
- [ArduPilot Discuss](https://discuss.ardupilot.org/)
- [ArduPilot Discord](https://ardupilot.org/discord)

---

**Previous**: [Radio Setup Guide](radio-setup.md) | **Next**: [Development Guide](development.md)
