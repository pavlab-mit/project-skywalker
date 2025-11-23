# Radio Setup and Mesh Networking Guide

This guide covers setup and configuration of communication radios for the Project Skywalker UAV system, including RFD900x telemetry radios and optional Wi-Fi mesh networks.

## Overview

The Project Skywalker platform supports multiple communication options:

1. **RFD900x** - Long-range (20-40+ km), low-bandwidth telemetry and mesh
2. **Rocket M5** - Medium-range (0.5-15 km), high-bandwidth mesh
3. **Hybrid** - Both radios for redundancy and different use cases

## Communication Architecture

### Use Cases by Radio Type

| Radio | Range | Bandwidth | Best For |
|-------|-------|-----------|----------|
| RFD900x | 20-40+ km | 10-100 kbps | C2, telemetry, heartbeats, resilient backup |
| Rocket M5 | 0.5-15 km | 10-300 Mbps | Video, maps, high-rate data, swarm coordination |

### Layer 2 vs Layer 3 Mesh

**Layer 2 (Data Link)**
- Protocols: 802.11s, BATMAN-adv
- Works like a giant switch
- Transparent to applications
- ❌ Broadcast floods can overwhelm mobile networks

**Layer 3 (Network)**
- Protocols: BATMAN classic, OLSR, AODV  
- Works like IP routing
- Better for mobile UAVs
- ✅ Scales better, adapts to mobility

**Recommendation**: Layer 3 (OLSR or BATMAN classic) for UAV swarms.

## Part 1: RFD900x Setup

### Prerequisites

- RFD900x radio (air and ground units)
- FTDI-to-USB cable (included with radio)
- Windows PC for firmware flashing
- macOS/Linux for configuration (or Windows)

### Step 1: Flash Asynchronous Firmware (Windows Required)

**Why Windows?** The RFD firmware tools only work reliably on Windows for flashing.

#### Download Tools and Firmware

1. **RFD Modem Tools V4**  
   Download from: [https://files.rfdesign.com.au/tools/](https://files.rfdesign.com.au/tools/)

2. **Asynchronous Firmware**  
   Download from: [https://files.rfdesign.com.au/firmware/](https://files.rfdesign.com.au/firmware/)  
   Look for: `RFDAsync V4.03 rfd900x2.gbl` or latest version

#### Flash Procedure

1. **Connect Radio**
   - Use FTDI cable to connect RFD900x to Windows PC
   - Radio usually appears as COM4 (may vary)
   - Default baud: 57,600 bps

2. **Open RFD Modem Tools**
   - Launch application
   - Click "Scan" to detect radio automatically
   - Or manually select COM port and baud rate

3. **Upload Firmware**
   - Go to **Firmware Upload** tab
   - Click **Browse** and select `.gbl` file
   - Click **Upload**
   - Wait for erase, flash, and verify (1-2 minutes)

4. **Verify**
   - Go to **Terminal** tab
   - Type: `ATI` (then Enter)
   - Should show: `RFD900x Async V4.03` or similar
   - Confirms successful flash

### Step 2: Configure Radio Parameters

After flashing firmware, configure for mesh networking.

#### Understanding Key Parameters

| Parameter | AT Command | Description | Recommended Value |
|-----------|------------|-------------|-------------------|
| Node ID | ATS10 | Unique ID for this radio | 1, 2, 3, ... (unique per UAV) |
| Destination ID | ATS11 | Target for messages | 65535 (broadcast for mesh) |
| TX Power | ATS12 | Transmit power in dBm | 30 (maximum, adjust as needed) |
| Antenna Mode | ATS23 | Antenna selection | 2 (auto-diversity) |
| Baud Rate | ATS1 | Serial baud rate | 57 (57,600 bps) |

#### View Current Settings

To see all current parameters:
```
ATI5
```

#### Configuration via Windows (RFD Modem Tools)

In the **Terminal** tab:

```
+++                 (Wait 2 seconds before this command, enter command mode)
ATS10=1            (Set Node ID - unique per radio!)
ATS11=65535        (Set Destination to broadcast)
ATS12=30           (Set TX power to 30 dBm)
ATS23=2            (Set antenna mode to auto-diversity)
AT&W               (Write settings to EEPROM)
ATZ                (Reboot to apply changes)
```

Reconnect and verify with `ATI5`.

#### Configuration via macOS/Linux

**Install picocom:**
```bash
brew install picocom        # macOS
sudo apt install picocom    # Linux
```

**Find Device:**
```bash
ls /dev/tty.*              # macOS
ls /dev/ttyUSB*            # Linux
```

Example: `/dev/tty.usbserial-BG00FTB3`

**Connect:**
```bash
picocom -b 57600 /dev/tty.usbserial-BG00FTB3
```

**Configure:**
```
(Wait 2 seconds)
+++                 (No Enter - enters command mode)
(Wait for "OK")
ATS10=1
ATS11=65535
ATS12=30
ATS23=2
AT&W
ATZ
```

**Exit picocom:**
```
Ctrl-A  Ctrl-X
```

**Verify:**
Reconnect and type `ATI5` to confirm settings.

### Step 3: Multiple Radio Configuration

For each UAV in your swarm:

1. Flash firmware on all radios (one time)
2. Configure each with unique Node ID:
   - UAV 1: `ATS10=1`
   - UAV 2: `ATS10=2`
   - UAV 3: `ATS10=3`
   - Ground station: `ATS10=100` (or any unique ID)
3. All radios use same Destination ID: `ATS11=65535`
4. Same TX power and other settings

### Step 4: Connect to Aircraft

**Ground Radio:**
- USB to laptop/ground station
- External antenna
- Used by QGroundControl for telemetry

**Air Radio (Option A - Direct to Autopilot):**
- Connect to CubeOrange TELEM1 port
- Provides MAVLink telemetry to ground

**Air Radio (Option B - Via Raspberry Pi for Mesh):**
- USB to Raspberry Pi
- Requires PPP/SLIP for IP networking
- Used for MOOS mesh communication

### Step 5: Test Radio Link

**Ground Range Test:**

1. Power both radios
2. Start at 1 meter separation
3. Open serial terminal on ground radio
4. Type characters - should see echoed from air radio
5. Walk increasing distances (10m, 50m, 100m, 500m, 1km+)
6. Monitor signal strength and packet loss

**Expected Performance:**
- 1 km: Excellent signal
- 5 km: Good signal  
- 10-20 km: Moderate signal (depends on antennas)
- 40+ km: Possible with high-gain directional antennas

## Part 2: Wi-Fi Mesh (Rocket M5) Setup

For high-bandwidth operations (video, maps, large data transfers).

### Prerequisites

- Ubiquiti Rocket M5 or similar 5.8 GHz radio
- OpenWrt or AirOS firmware
- Network access for configuration

### Step 1: Flash OpenWrt (if needed)

Many CPE radios come with vendor firmware. OpenWrt provides better mesh support.

1. Download OpenWrt for your hardware
2. Flash via web interface or TFTP
3. Configure basic network access
4. Enable SSH access

### Step 2: Install Mesh Protocol

**Option A: OLSR (Optimized Link State Routing)**

```bash
opkg update
opkg install olsrd olsrd-mod-arprefresh olsrd-mod-txtinfo

# Edit /etc/config/olsrd
# Add interfaces and configure
/etc/init.d/olsrd enable
/etc/init.d/olsrd start
```

**Option B: BATMAN-adv (Layer 2 mesh)**

```bash
opkg update
opkg install kmod-batman-adv batctl

# Load kernel module
modprobe batman-adv

# Configure mesh interface
batctl if add wlan0
ip link set up dev bat0
```

### Step 3: Configure Radio

Example OLSR configuration (`/etc/config/olsrd`):

```
config olsrd
    option IpVersion '4'
    option FIBMetric 'flat'

config Interface
    option interface 'wlan0'
    option Mode 'mesh'
```

### Step 4: Network Integration

Connect to Raspberry Pi via Ethernet:
- Configure static IP or DHCP
- Add routing for MOOS traffic
- Test connectivity between nodes

### Step 5: Test Mesh

```bash
# Check OLSR topology
echo "/topology" | nc 127.0.0.1 2006

# Check BATMAN-adv neighbors (if using BATMAN)
batctl o

# Ping test to other mesh nodes
ping <node-ip>

# Bandwidth test
iperf3 -s                    # On one node
iperf3 -c <node-ip>          # From another
```

## Part 3: Hybrid Mesh Architecture

Recommended setup for research and operational use:

### Architecture

```
UAV System:
├── RFD900x (Primary)
│   ├── Long-range backup
│   ├── Low-bandwidth C2
│   └── Encrypted telemetry
│
└── Rocket M5 (Secondary)
    ├── High-bandwidth data
    ├── Video streaming
    └── Map synchronization
```

### Implementation

1. **RFD900x**: Connected to CubeOrange TELEM1
   - Always-on link to ground
   - Emergency communications
   - Heartbeats and status

2. **Rocket M5**: Connected to Raspberry Pi Ethernet
   - MOOS inter-vehicle communication
   - Payload data
   - Optional: GCS monitoring

### Failover Logic

In MOOS missions, implement logic:
- Primary comms: Wi-Fi mesh (high rate)
- If Wi-Fi fails: Fall back to RFD900x
- If both fail: Execute RTL (Return to Launch)

## Mesh Networking for UAV Swarms

### Why Mesh?

- ✅ No central infrastructure required
- ✅ Self-healing (routes around failures)
- ✅ Multi-hop extends range
- ✅ Scalable to many nodes
- ✅ Resilient to individual node failures

### RFD900x Mesh Pattern

For telemetry radios (serial pipes):
1. Wrap in PPP or SLIP (create network interface)
2. Run BATMAN classic (userspace, Layer 3)
3. Each UAV sees others via IP
4. MOOS pShare communicates over mesh

### Wi-Fi Mesh Pattern

For broadband radios:
1. Configure OLSR or BATMAN-adv
2. All nodes join mesh automatically
3. MOOS applications communicate via UDP multicast
4. Video can stream peer-to-peer

### Practical Swarm Example

**3-UAV Swarm:**

```
UAV-1 (Node 1) ←──── Mesh ────→ UAV-2 (Node 2)
      ↑                              ↑
      └──────── Mesh ────────────────┘
                 ↕
            UAV-3 (Node 3)
                 ↕
          Ground Station (Node 100)
```

All nodes share:
- Position and heading
- Mission status  
- Coordination commands
- Payload data (if bandwidth permits)

## Troubleshooting

### RFD900x Issues

**Can't Enter Command Mode:**
- Wait full 2 seconds before `+++`
- Don't press Enter after `+++`
- Check baud rate matches radio setting

**No Communication:**
- Verify Node IDs are unique
- Check Destination ID = 65535 on all radios
- Test antennas (try different antenna)
- Check power levels

**Short Range:**
- Increase TX power (up to 30 dBm)
- Use better antennas
- Check for RF interference
- Ensure antenna has clear view

### Wi-Fi Mesh Issues

**Can't See Neighbors:**
- Verify same SSID/channel
- Check mesh protocol running
- Verify IP addressing
- Check firewall rules

**Poor Performance:**
- Reduce TX power if too close
- Check channel congestion
- Verify antenna orientation
- Test LOS (line of sight)

## Security Considerations

### RFD900x
- ✅ Built-in AES encryption (transparent)
- No additional configuration needed

### Wi-Fi Mesh
- Use WPA2/WPA3 encryption
- Optional: Add VPN layer (WireGuard recommended)
- Firewall rules to limit access

## Performance Optimization

### RFD900x
- Lower baud rate = longer range
- Higher TX power = longer range (more battery)
- Async mode handles multiple nodes better than sync

### Wi-Fi Mesh
- OLSR: Better routing visibility, more overhead
- BATMAN-adv: Lower overhead, less visibility
- Choose based on network size and requirements

## Next Steps

After radio configuration:

1. **[Quick Start Guide](quickstart.md)** - Integrate with MOOS
2. **[Usage Guide](usage.md)** - Multi-vehicle operations
3. **[Troubleshooting](troubleshooting.md)** - Solve issues

## Additional Resources

- [RFD900x User Manual](https://rfdesign.com.au/wp-content/uploads/2024/04/RFD900x-Asynchronous-User-Manual-V1.1.pdf)
- [OpenWrt Documentation](https://openwrt.org/docs/start)
- [OLSR Documentation](https://www.olsr.org/)
- [BATMAN-adv Documentation](https://www.open-mesh.org/projects/batman-adv/wiki)
- Detailed guides in `Info/` directory:
  - `Info/software_installation/RFD900x_Asynchronous_Mesh_Setup_Guide.md`
  - `Info/design_decisions/UAV_Swarm_Mesh_Guide.md`
  - `Info/design_decisions/telemetry_vs_wi_fi_comparison.md`

---

**Previous**: [Hardware Guide](hardware.md) | **Next**: [Firmware Guide](firmware.md)
