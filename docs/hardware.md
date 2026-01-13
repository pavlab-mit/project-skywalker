# Hardware Assembly Guide

This guide covers the physical assembly and integration of the Project Skywalker UAV platform.

## Bill of Materials

### Core Components

| Component | Description | Quantity | Notes |
|-----------|-------------|----------|-------|
| Skywalker X8 | Fixed-wing airframe, 2.12m wingspan | 1 | EPO foam construction |
| CubeOrange+ | Autopilot with carrier board | 1 | Or CubeOrange |
| Raspberry Pi 4B | Companion computer | 1 | 4GB or 8GB RAM |
| MicroSD Card | For Raspberry Pi OS | 1 | 32GB+ Class 10 |
| RFD900x | Long-range telemetry radio | 1-2 | 1 for UAV, 1 for ground |
| GPS Module | Dual GPS recommended | 1-2 | Usually included with Cube |
| Power Module | Voltage/current sensing | 1 | For Cube |
| LiPo Battery | 4S-6S, 5000-10000mAh | 1+ | Depends on endurance needs |
| Servo Set | For control surfaces | 4-6 | Ailerons, elevator, rudder |
| Brushless Motor | Appropriate for X8 | 1 | ~1000kV typical |
| ESC | Electronic Speed Controller | 1 | 50-80A recommended |
| Propeller | 11-13 inch | Multiple | Various for testing |

### Optional Components

| Component | Description | Use Case |
|-----------|-------------|----------|
| Rocket M5 | High-bandwidth mesh radio | Swarm ops, video |
| Airspeed Sensor | Pitot tube | Fixed-wing recommended |
| Optical Flow | Downward camera + rangefinder | GPS-denied nav |
| FPV Camera | Video feed | Monitoring, research |
| Telemetry Radio #2 | Backup link | Redundancy |

### 3D-Printed Parts

Available in `Info/stl_files/`:
- Raspberry Pi case (provides port access and mounting)
- Radio mounting brackets (in development)
- Antenna mounts (in development)

## Assembly Overview

```
1. Prepare Airframe
2. Install Autopilot (CubeOrange)
3. Install Companion Computer (Raspberry Pi)
4. Connect Communication Radios
5. Wire Power Distribution
6. Mount Servos and Control Linkages
7. Install Propulsion System
8. Final Integration and Testing
```

## Step 1: Prepare Airframe

### Skywalker X8 Assembly

The X8 typically comes as a kit requiring assembly:

1. **Join wing halves**
   - Apply epoxy to wing joiner tube
   - Carefully align and join wing panels
   - Allow to cure per epoxy instructions

2. **Attach tail surfaces**
   - Install vertical stabilizers
   - Secure horizontal stabilizer
   - Check alignment

3. **Prepare electronics bay**
   - Plan component layout
   - Consider weight distribution (CG is critical)
   - Ensure adequate cooling airflow

### Center of Gravity (CG)

**Critical**: X8 CG should be at approximately 65-75mm behind the leading edge at the wing root, measured at fuselage.

- Mark CG position clearly
- Test CG before every flight
- Adjust battery position to achieve correct CG

## Step 2: Install CubeOrange Autopilot

### Mounting

1. **Vibration Isolation**
   - Use provided foam or gel pads
   - Mount Cube securely but not rigidly
   - Orientation: Arrow forward, USB accessible

2. **Carrier Board Location**
   - Place in electronics bay
   - Leave room for Raspberry Pi and radios
   - Ensure all ports accessible

### Connections

```
CubeOrange Ports:
├── TELEM1: Ground station telemetry (RFD900x)
├── TELEM2: Raspberry Pi (MAVLink)
├── GPS1: Primary GPS
├── GPS2: Secondary GPS (optional)
├── POWER1: Power module
├── POWER2: Backup power (optional)
├── RCIN: RC receiver
├── Main Out: Servos and ESC
└── USB: Configuration and firmware updates
```

**Key Connections:**
- TELEM2 → Raspberry Pi USB (via USB-to-serial or direct USB)
- TELEM1 → RFD900x for ground link
- GPS1 → Primary GPS module
- POWER1 → Power module from battery

### Orientation

- Arrow on Cube must point forward
- Use mounting orientation parameters if mounted differently:
  - `AHRS_ORIENTATION` in ArduPilot
  - Configure via QGroundControl

## Step 3: Install Raspberry Pi

### 3D-Printed Case

1. Print case from `Info/stl_files/` directory
2. Install heatsinks on Pi CPU
3. Insert Pi into case
4. Secure with screws

### Mounting Location

- Near CubeOrange for short cable runs
- Ensure good ventilation
- Protect from vibration with foam pads
- Keep antennas clear

### Power Supply

**Option 1: USB-C Power**
- 5V regulator from main battery
- 2.5A minimum, 3A recommended
- Use quality regulator (switching recommended for efficiency)

**Option 2: GPIO Power**
- 5V to GPIO pins 2 and 4
- **Warning**: Bypasses protection, use carefully
- Good regulation essential

### Connections

```
Raspberry Pi:
├── USB-C: Power (5V/3A)
├── USB 2.0: CubeOrange (MAVLink)
├── USB 3.0: RFD900x (optional, if using serial adapter)
├── Ethernet: Rocket M5 (if used)
└── GPIO: (reserved for future sensors)
```

## Step 4: Communication Radios

### RFD900x Installation

**Ground Radio:**
- USB connection to ground station laptop
- External antenna (dipole or directional)
- Clear line of sight when possible

**Air Radio:**
- Option A: Serial to Cube TELEM1
- Option B: USB to Raspberry Pi (for mesh networking)
- Mount antenna externally
- Ensure antenna has clear view (no metal blocking)

**Antenna Placement:**
- Mount antennas away from carbon fiber and metal
- Vertical orientation for omni pattern
- Consider using antenna extension cable
- Test signal strength before flight

### Rocket M5 (Optional)

For high-bandwidth mesh:
- Mount in or on fuselage
- Requires external power (12V typical)
- Ethernet to Raspberry Pi
- Configure mesh protocol (OLSR/BATMAN) - see [Radio Setup](radio-setup.md)

## Step 5: Power Distribution

### Power Architecture

```
LiPo Battery (4S-6S)
    │
    ├─> Power Module ─> CubeOrange (5V)
    │
    ├─> 5V BEC ─> Raspberry Pi (5V/3A)
    │
    ├─> ESC ─> Motor (battery voltage)
    │
    └─> Servos ─> 5-6V (via BEC or separate BEC)
```

### Key Considerations

1. **Separate Power Supplies**
   - CubeOrange: Dedicated power module
   - Raspberry Pi: Dedicated 5V regulator
   - Servos: Can share with Cube or use separate BEC

2. **Current Capacity**
   - Motor: 20-60A typical
   - Servos: 1-2A each under load
   - Raspberry Pi: 2-3A
   - Radios: 1-2A transmitting

3. **Voltage Monitoring**
   - Power module provides voltage/current to ArduPilot
   - Set battery capacity and voltages in parameters
   - Configure low battery failsafe

### Wiring Best Practices

- Use appropriate wire gauge (battery to ESC: 10-12 AWG)
- Secure all connections with heat shrink
- Service loops for vibration
- Label all wires
- Use connectors for easy disassembly

## Step 6: Servos and Control Linkages

### Servo Assignment (Typical)

```
CubeOrange Main Out:
├── 1: Left Aileron
├── 2: Right Aileron
├── 3: Elevator
├── 4: Rudder
├── 5: Throttle (ESC)
├── 6: (Optional: camera gimbal, etc.)
└── 7-8: (Reserved)
```

Configure in ArduPilot parameters:
- `SERVO1_FUNCTION = 4` (Aileron)
- `SERVO2_FUNCTION = 4` (Aileron)
- `SERVO3_FUNCTION = 19` (Elevator)
- `SERVO4_FUNCTION = 21` (Rudder)
- `SERVO5_FUNCTION = 70` (Throttle)

### Control Linkage Setup

1. **Install Servos**
   - Use provided mounting hardware
   - Ensure secure mounting (vibration resistant)
   - Leave room for servo arm movement

2. **Set Neutral Positions**
   - Power on system
   - Center all servos via RC or QGC
   - Attach servo arms at neutral

3. **Connect Pushrods**
   - Adjust length for proper control surface travel
   - Use ball links or clevises
   - Ensure smooth, bind-free movement
   - Check full deflection in both directions

4. **Set Limits**
   - Configure max/min servo PWM values
   - Prevent over-travel and binding
   - Test under load

## Step 7: Propulsion System

### Motor Installation

1. Mount motor to firewall
2. Ensure alignment with fuselage centerline
3. Secure all screws with threadlocker
4. Leave clearance for propeller

### ESC Installation

1. Mount ESC with airflow cooling
2. Connect to motor (bullet connectors typical)
3. Connect to battery (XT60/XT90 typical)
4. Signal wire to CubeOrange Main Out 5

### Propeller Selection

Start with manufacturer recommendations:
- X8: 11-13 inch propellers
- Test various props for efficiency
- Balance all propellers before use
- Mark rotation direction

**Safety**: Never approach a powered aircraft with propeller installed!

## Step 8: Final Integration

### Internal Layout

Optimize component placement:
1. **CG**: Place heavy items (battery) to achieve correct CG
2. **Access**: Ensure SD cards, USB ports accessible
3. **Cooling**: Airflow for ESC, Pi, Cube
4. **Antennas**: External, unobstructed
5. **Wiring**: Neat, secured, labeled

### Weight Budget

Track weight carefully:
- Skywalker X8 empty: ~1.5 kg
- Electronics: 300-500g
- Battery: 500-1200g (depends on capacity)
- **Total**: 2.3-3.2 kg typical
- **Max**: Keep below 3.5 kg for good performance

### Securing Components

- Use velcro for battery (easy removal/repositioning)
- Foam padding for vibration isolation
- Secure all electronics with straps or tape
- Ensure nothing can shift in flight
- Check security before every flight

## Testing and Calibration

### Bench Testing

Before flight testing:

1. **Power-on Test**
   - Verify all systems power up
   - Check for shorts or overheating
   - Monitor current draw

2. **Control Surface Test**
   - Move all surfaces via RC
   - Verify correct direction
   - Check full travel
   - Test failsafe behavior

3. **Sensor Calibration**
   - Accelerometer calibration
   - Compass calibration (away from metal/electronics)
   - RC calibration
   - ESC calibration

4. **Communication Test**
   - Verify MAVLink to ground station
   - Test Raspberry Pi to CubeOrange link
   - Check radio range (walk test)

### Ground Testing

1. **CG Check**
   - Aircraft should balance at marked CG
   - Adjust battery position as needed

2. **Control Check**
   - Move aircraft, verify control response
   - Check correct deflection direction
   - Test all flight modes

3. **Pre-flight Checklist**
   - See [Quick Start Guide](quickstart.md) for detailed checklist

## Maintenance

### Regular Inspections

- Check all screws and connections
- Inspect propeller for damage
- Test servo operation
- Verify antenna integrity
- Check for vibration damage

### After Crashes

- Inspect airframe for cracks
- Check control linkages
- Verify motor alignment
- Test all electronics
- Recalibrate sensors if needed

## Troubleshooting

### Vibration Issues
- Add foam padding to Cube
- Balance propeller
- Check motor mounting
- Verify structural integrity

### Poor Range
- Check antenna placement
- Verify radio configuration
- Test with better antenna
- Eliminate RF interference

### CG Problems
- Move battery forward/backward
- Redistribute electronics
- Add/remove weight as needed
- Always verify before flight

## Next Steps

After hardware assembly:

1. **[Firmware Guide](firmware.md)** - Flash and configure ArduPilot
2. **[Radio Setup](radio-setup.md)** - Configure communication systems
3. **[Installation Guide](installation.md)** - Set up Raspberry Pi software
4. **[Quick Start](quickstart.md)** - System integration testing

## Additional Resources

- [Skywalker X8 Manual](https://www.skywalkermodel.com/)
- [CubeOrange Documentation](https://docs.cubepilot.org/)
- [ArduPilot Plane Setup](https://ardupilot.org/plane/docs/arduplane-setup.html)
- STL Files: `Info/stl_files/` directory in this repository

## Safety Reminders

- ⚠️ Always remove propeller during testing
- ⚠️ Test failsafe behavior before flight
- ⚠️ Secure all components before flight
- ⚠️ Check CG before every flight
- ⚠️ Follow local regulations and safety guidelines

---

**Previous**: [Architecture Guide](architecture.md) | **Next**: [Radio Setup Guide](radio-setup.md)
