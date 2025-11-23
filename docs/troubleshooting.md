# Troubleshooting Guide

This guide covers common issues and solutions for the Project Skywalker UAV system.

## General Troubleshooting Approach

When encountering issues:

1. **Check basics first** - Power, connections, configuration
2. **Review logs** - MOOS logs, ArduPilot logs, system logs
3. **Isolate the problem** - Test components individually
4. **Search documentation** - Check this guide and official docs
5. **Ask for help** - GitHub issues or contact team

## Quick Diagnostics

### System Health Check

```bash
# On Raspberry Pi
ssh skywalker1

# Check system resources
htop                    # CPU and memory usage
df -h                   # Disk space
vcgencmd measure_temp   # CPU temperature

# Check MOOS processes
uMS mission.moos

# Check serial devices
ls /dev/tty*

# Check network
ifconfig
ping 8.8.8.8
```

### Log Files

```bash
# MOOS logs
ls ~/missions/*/logs/*.alog
alogview logs/latest.alog

# System logs
journalctl -xe
dmesg | tail

# ArduPilot logs (from SD card)
# Download via QGroundControl
```

## Installation Issues

### MOOS-IvP Build Fails

**Symptom:** `./build.sh` fails with errors

**Common Causes:**

1. **Missing Dependencies**
   ```bash
   # Ubuntu/Raspberry Pi
   sudo apt-get update
   sudo apt-get install -y build-essential cmake git subversion
   ```

2. **Out of Memory**
   - Raspberry Pi runs out of RAM during compilation
   ```bash
   # Use fewer parallel jobs
   ./build.sh -j1
   
   # Add swap space if needed
   sudo dphys-swapfile swapoff
   sudo nano /etc/dphys-swapfile  # Set CONF_SWAPSIZE=1024
   sudo dphys-swapfile setup
   sudo dphys-swapfile swapon
   ```

3. **Submodule Issues**
   ```bash
   git submodule update --init --recursive
   ```

### SSH Connection Fails

**Symptom:** Cannot connect to Raspberry Pi

**Solutions:**

1. **Wait Longer** - First boot takes 1-2 minutes
2. **Check Network**
   ```bash
   ping skywalker1.local
   # If fails, try IP address
   nmap -sn 192.168.1.0/24  # Find Pi on network
   ```

3. **Check mDNS**
   ```bash
   # macOS - should work by default
   # Linux - install avahi
   sudo apt-get install avahi-daemon
   ```

4. **Use IP Address**
   ```bash
   ssh uav@192.168.1.100  # Replace with actual IP
   ```

5. **Check SSH Service**
   - Re-flash SD card with SSH enabled
   - Or mount SD card and create empty `ssh` file in boot partition

## Hardware Issues

### CubeOrange Not Detected

**Symptom:** QGroundControl or MAVProxy can't find autopilot

**Solutions:**

1. **Check USB Cable**
   - Try different cable
   - Try different USB port
   - USB 2.0 sometimes more reliable than 3.0

2. **Check Power**
   - Verify power LED lit on Cube
   - Check power module connection
   - Try USB power alone (disconnect battery)

3. **Check Serial Device**
   ```bash
   # macOS
   ls /dev/tty.usb*
   
   # Linux
   ls /dev/ttyACM*
   
   # Should see device appear when connected
   ```

4. **Driver Issues (Windows)**
   - Install CubeOrange drivers
   - Use Device Manager to verify

5. **Try Bootloader Mode**
   - Disconnect Cube
   - Hold safety button while connecting USB
   - Should enter bootloader (for firmware flashing)

### GPS No Lock

**Symptom:** GPS won't acquire satellite fix

**Solutions:**

1. **Wait Longer** - First lock can take 5-15 minutes
2. **Check Antenna** - Must have clear view of sky
3. **Check Cable** - Ensure GPS cable secure
4. **Check Configuration**
   ```
   GPS_TYPE = 1    # Auto-detect
   GPS_AUTO_SWITCH = 1
   ```

5. **Verify GPS Function**
   - GPS LED should blink
   - Check in QGC sensor page
   - Verify HDOP and satellite count improving

### Compass Errors

**Symptom:** "Pre-Arm: Compass variance" or "Compass inconsistent"

**Solutions:**

1. **Recalibrate**
   - Move away from metal and electronics
   - Follow QGC calibration procedure carefully
   - Rotate in all axes slowly and completely

2. **Check Interference**
   ```bash
   # In QGC, check compass offsets
   # Should be < 600 on any axis
   # High offsets indicate interference
   ```
   - Move GPS/compass away from power wires
   - Use external compass on mast if possible

3. **Disable Bad Compass**
   ```
   COMPASS_USE = 1      # Use only compass 1
   COMPASS_USE2 = 0     # Disable compass 2
   COMPASS_USE3 = 0     # Disable compass 3
   ```

4. **Check Orientation**
   ```
   COMPASS_ORIENT = 0   # Default orientation
   # Adjust if mounted differently
   ```

### Servo Issues

**Symptom:** Servos don't move or move incorrectly

**Solutions:**

1. **Check Power**
   - Verify servo power (5-6V)
   - Check BEC or power module

2. **Check Connections**
   - Verify servo plugs secure
   - Check signal wire not loose

3. **Check Configuration**
   ```
   SERVO1_FUNCTION = 4   # Aileron
   SERVO1_MIN = 1000
   SERVO1_MAX = 2000
   SERVO1_TRIM = 1500
   ```

4. **Test Servos**
   - Manual mode with RC
   - Move sticks, verify response
   - Check servo output page in QGC

5. **Check Reversed**
   ```
   SERVO1_REVERSED = 0   # Change to 1 if reversed
   ```

## Communication Issues

### RFD900x Connection Problems

**Symptom:** No telemetry link or poor range

**Solutions:**

1. **Verify Configuration**
   ```
   ATI5    # Check all settings
   
   # Key parameters:
   ATS10 = unique node ID
   ATS11 = 65535 (broadcast)
   ATS12 = 30 (TX power)
   ```

2. **Check Antenna**
   - Verify antenna connected
   - Check for damage
   - Try different antenna
   - Ensure clear line of sight

3. **Check Baud Rate**
   - Radio baud matches system
   - Default 57600
   - QGC or MAVProxy baud matches

4. **Test on Bench**
   ```bash
   # Connect both radios
   # Type in one, should appear on other
   picocom -b 57600 /dev/tty.usbserial-XXX
   ```

5. **Reduce TX Power for Testing**
   ```
   ATS12=10   # Lower power for close-range test
   AT&W
   ATZ
   ```

### Wi-Fi Mesh Issues

**Symptom:** Mesh nodes can't see each other

**Solutions:**

1. **Check OLSR/BATMAN Running**
   ```bash
   # For OLSR
   ps aux | grep olsrd
   /etc/init.d/olsrd status
   
   # For BATMAN
   batctl o  # Show neighbors
   ```

2. **Check IP Addressing**
   ```bash
   ifconfig
   # Verify all nodes on same subnet
   ```

3. **Test Connectivity**
   ```bash
   ping <neighbor-ip>
   traceroute <neighbor-ip>
   ```

4. **Check Channel/SSID**
   - All nodes must use same channel
   - Same SSID for mesh
   - Check with `iwconfig`

5. **Check Firewall**
   ```bash
   # Temporarily disable for testing
   sudo iptables -F
   ```

### MAVLink Communication Fails

**Symptom:** Raspberry Pi can't talk to CubeOrange

**Solutions:**

1. **Check Serial Device**
   ```bash
   ls /dev/ttyACM0
   # If not found, check connections
   ```

2. **Check Baud Rate**
   - Match on both sides (usually 115200)
   - ArduPilot: `SERIAL2_BAUD = 115`
   - MAVProxy/MAVSDK: 115200

3. **Check Protocol**
   ```
   SERIAL2_PROTOCOL = 2   # MAVLink 2
   ```

4. **Test with MAVProxy**
   ```bash
   python3 -m MAVProxy.mavproxy --master=/dev/ttyACM0,115200
   # Should see heartbeat within seconds
   ```

5. **Check Permissions**
   ```bash
   sudo usermod -a -G dialout $USER
   # Log out and back in
   ```

## Flight Issues

### Pre-Arm Checks Fail

**Symptom:** Cannot arm vehicle

**Common Errors:**

1. **"Pre-Arm: Need 3D Fix"**
   - Wait for GPS lock
   - Need 6+ satellites
   - Check GPS antenna placement

2. **"Pre-Arm: Compass variance"**
   - See Compass Errors section above
   - Recalibrate away from metal

3. **"Pre-Arm: RC not calibrated"**
   - Run RC calibration in QGC
   - Verify all channels working

4. **"Pre-Arm: Accelerometer not calibrated"**
   - Run accelerometer calibration
   - Place aircraft level during calibration

5. **"Pre-Arm: Barometer not healthy"**
   - Check for loose connections
   - Verify barometer not blocked
   - Reboot system

### In-Flight Control Issues

**Symptom:** Aircraft doesn't respond correctly

**Immediate Actions:**

1. **Switch to Manual Mode**
   - Use RC transmitter
   - Take manual control

2. **Check Flight Mode**
   - Verify correct mode active
   - Try different mode

3. **Land Safely**
   - Return to manual
   - Land as soon as practical

**Post-Flight Analysis:**

1. **Download Logs**
   - ArduPilot logs from SD card
   - MOOS logs from Raspberry Pi

2. **Check Control Surfaces**
   - Verify mechanically sound
   - Check for binding or damage

3. **Review Parameters**
   - Check PID gains
   - Verify servo settings
   - Review mode settings

### Lost Link

**Symptom:** Loss of telemetry during flight

**Expected Behavior:**
- ArduPilot executes failsafe (usually RTL)
- Aircraft should return autonomously

**If Aircraft Doesn't Return:**

1. **Keep Transmitting** - Radio link may recover
2. **Move to Higher Ground** - Improve line of sight
3. **Use Directional Antenna** - If available
4. **Check Last Known Position** - From telemetry
5. **Follow Safety Procedures** - As per your flight plan

**Post-Incident:**

1. **Review Logs** - Understand what happened
2. **Test Radio Range** - Verify link quality
3. **Adjust Failsafe Settings** - If needed
4. **Consider Redundant Link** - Backup radio

## MOOS-IvP Issues

### MOOSDB Won't Start

**Symptom:** `pAntler mission.moos` fails

**Solutions:**

1. **Check Port Available**
   ```bash
   netstat -tuln | grep 9000
   # If in use, kill process or change port
   ```

2. **Kill Existing Processes**
   ```bash
   killall MOOSDB pAntler
   ```

3. **Check Mission File Syntax**
   ```bash
   # Look for typos or missing values
   cat mission.moos
   ```

4. **Check File Permissions**
   ```bash
   chmod +x mission.moos
   ```

### MOOS Application Crashes

**Symptom:** Application exits unexpectedly

**Solutions:**

1. **Check Logs**
   ```bash
   alogview logs/*.alog
   # Look for error messages
   ```

2. **Run with Debug**
   ```bash
   # Add --verbose to see more output
   MyApp mission.moos --verbose
   ```

3. **Check Configuration**
   - Verify all required parameters present
   - Check for typos in config block

4. **Test Standalone**
   ```bash
   # Create minimal mission with just MOOSDB + app
   ```

### Variables Not Publishing

**Symptom:** uXMS shows no data for variables

**Solutions:**

1. **Check Registration**
   ```cpp
   // In application code:
   Register("MY_VAR", 0);
   ```

2. **Check Publishing**
   ```cpp
   Notify("MY_VAR", value);
   ```

3. **Check CommsTick**
   - Increase if too low
   - CommsTick in mission file

4. **Verify MOOSDB Connection**
   ```bash
   uMS mission.moos
   # Should show application connected
   ```

## Performance Issues

### Raspberry Pi Slow/Laggy

**Solutions:**

1. **Check CPU Usage**
   ```bash
   htop
   # Look for processes using high CPU
   ```

2. **Check Temperature**
   ```bash
   vcgencmd measure_temp
   # Should be < 80°C
   # Add heatsinks if too hot
   ```

3. **Check Memory**
   ```bash
   free -h
   # Ensure swap not heavily used
   ```

4. **Reduce Load**
   - Lower MOOS AppTick rates
   - Disable unnecessary processes
   - Reduce logging verbosity

### Short Flight Time

**Solutions:**

1. **Check Battery**
   - Verify capacity sufficient
   - Check for damage/wear
   - Measure actual capacity

2. **Reduce Weight**
   - Remove unnecessary equipment
   - Optimize component placement

3. **Optimize Cruise Speed**
   - Too fast or slow reduces efficiency
   - Test various speeds

4. **Check Motor/Prop**
   - Ensure efficient combination
   - Balance propeller
   - Check motor timing

## Error Messages Reference

### Common ArduPilot Errors

| Error | Meaning | Solution |
|-------|---------|----------|
| Pre-Arm: Need 3D Fix | No GPS lock | Wait for satellites |
| Pre-Arm: Compass variance | Compass mismatch | Recalibrate compass |
| Pre-Arm: RC not calibrated | RC not configured | Calibrate radio |
| Failsafe: GCS | Lost ground station link | Check radio |
| Failsafe: Battery | Low battery | Land immediately |
| EKF variance | State estimation error | Check sensors |

### Common MOOS Errors

| Error | Meaning | Solution |
|-------|---------|----------|
| MOOSDB not found | Can't connect to DB | Check port, start MOOSDB |
| Variable not registered | App not subscribed | Register variable |
| Port already in use | MOOSDB running | Kill existing or change port |
| Behavior error | Behavior exception | Check behavior config |

## Getting Additional Help

If issues persist:

1. **Check Documentation**
   - [Installation Guide](installation.md)
   - [Hardware Guide](hardware.md)
   - [Firmware Guide](firmware.md)
   - [Usage Guide](usage.md)

2. **Search Issues**
   - [GitHub Issues](https://github.com/pavlab-mit/project-skywalker/issues)
   - ArduPilot forums
   - MOOS-IvP forums

3. **Contact Team**
   - General: team@pavlab-mit.org
   - Technical: chbenj36@gmail.com

4. **Include Information**
   - Hardware versions
   - Software versions
   - Steps to reproduce
   - Log files
   - What you've tried

## Preventive Maintenance

Regular maintenance prevents many issues:

### Before Each Flight

- [ ] Check all connections
- [ ] Verify battery voltage
- [ ] Test control surfaces
- [ ] Confirm GPS lock
- [ ] Check radio link
- [ ] Review mission plan
- [ ] Test failsafe

### Weekly

- [ ] Check for loose screws
- [ ] Inspect wiring
- [ ] Clean sensors
- [ ] Update software (if needed)
- [ ] Backup configurations

### Monthly

- [ ] Deep inspection of airframe
- [ ] Check servo wear
- [ ] Test battery capacity
- [ ] Verify calibrations
- [ ] Update documentation

## Emergency Procedures

### In-Flight Emergency

1. **Stay Calm**
2. **Switch to Manual** (if possible)
3. **Assess Situation**
4. **Land Safely** - Priority #1
5. **Document** - Note what happened

### Lost Aircraft

1. **Note Last Position**
2. **Check Logs** - Last telemetry
3. **Search Area** - Systematic
4. **Use Telemetry Beeper** - If installed
5. **Report** - To authorities if required

### Equipment Damage

1. **Safety First** - Disconnect battery
2. **Document Damage** - Photos
3. **Assess Repairs** - What's needed
4. **Test Thoroughly** - Before next flight

---

**Previous**: [Contributing Guide](contributing.md) | **Next**: [Authors](authors.md)
