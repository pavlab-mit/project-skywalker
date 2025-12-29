# RFD900x → PPP → Headless IP Network (Mac ↔ Raspberry Pi)

## Objective

Create a **persistent, headless IP network** over **RFD900x telemetry radios** so that MOOS-IvP missions can run over standard IP networking **without Wi-Fi, Ethernet, MAVLink, or SSH access at boot**.

Note: This is a solution for single-vehicle ↔ ground-station communication. If your goal is scalable multi-vehicle missions, using the RFD900x serial radio can serve as an intermediate testing step before moving to network radios (layer 3) like mesh or wifi radios.

---

## System Architecture

```
Mac (ppp0: 10.0.0.1)
   ⇄ RFD900x radios ⇄
Pi  (ppp0: 10.0.0.2)
```

- Radios act as a **transparent serial link**
- PPP converts serial into a **network interface**
- The Pi **boots already network-ready**
- The ground station decides when the network exists

---

## Why PPP (`pppd`) Is Required

- RFD900x radios provide **serial**, not Ethernet
- MOOS-IvP requires **IP networking**
- PPP encapsulates IP packets over serial
- The OS exposes this as `ppp0`

Result: radios behave like a **point-to-point Ethernet cable**.

---

## Prerequisites

- 2× RFD900x radios
- Raspberry Pi (Bookworm or similar)
- Mac (macOS includes `pppd`)
- USB connection to each radio
- Radios configured with:
  - Same `NETID`
  - Same `SERIAL_SPEED` (115200 used here)
  - **MAVLINK disabled**

---

## Step 1 — Verify Raw Serial Transport

Before configuring PPP, confirm that the radios pass raw bytes.

### On the Pi
```bash
screen /dev/ttyUSB0 115200
```

### On the Mac
```bash
screen /dev/tty.usbserial-* 115200
```

Typing on one side must appear on the other.

Exit screen **completely**:
```
Ctrl+A → \ → y
```

> PPP will not work if `screen` is still attached.

---

## Step 2 — Install PPP

### On the Raspberry Pi
```bash
sudo apt update
sudo apt install ppp
```

macOS already includes `pppd`.

---

## Step 3 — Create PPP Peer Configurations

### Pi configuration

Create:
```bash
sudo nano /etc/ppp/peers/rfd
```

Paste:
```
/dev/ttyUSB0 115200
noauth
local
nodetach
passive
persist
maxfail 0
10.0.0.2:10.0.0.1
```

Save and exit.

---

### Mac configuration

Create directory if missing:
```bash
sudo mkdir -p /etc/ppp/peers
```

Create config:
```bash
sudo nano /etc/ppp/peers/rfd
```

Paste:
```
/dev/tty.usbserial-XXXX 115200
noauth
local
nodetach
passive
10.0.0.1:10.0.0.2
```

Replace `/dev/tty.usbserial-XXXX` with the actual device name.

Save and exit.

---

## Step 4 — Ensure Serial Port Is Free (Mac)

PPP requires exclusive access to the serial device.

```bash
screen -ls
```

If any sessions exist:
```bash
screen -S <id> -X quit
```

Verify:
```bash
lsof | grep tty.usbserial
```

There must be **no output**.

---

## Step 5 — Bring Up PPP Manually (Initial Test)

### Start PPP on the Pi first
```bash
sudo pppd call rfd
```

Expected:
```
Using interface ppp0
Connect: ppp0 <--> /dev/ttyUSB0
```

Leave running.

---

### Start PPP on the Mac
```bash
sudo pppd call rfd
```

Expected:
```
Using interface ppp0
local IP address 10.0.0.1
remote IP address 10.0.0.2
```

---

## Step 6 — Verify Connectivity

```bash
ifconfig ppp0
```

Test:
```bash
ping 10.0.0.2   # from Mac
ping 10.0.0.1   # from Pi
```

Optional:
```bash
ssh uav@10.0.0.2
```

---

## Step 7 — Make PPP Persistent on the Pi (Field Mode)

### Create a systemd service

```bash
sudo nano /etc/systemd/system/ppp-rfd.service
```

Paste:
```ini
[Unit]
Description=PPP over RFD900x
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/sbin/pppd call rfd
Restart=always
RestartSec=2

[Install]
WantedBy=multi-user.target
```

---

### Enable and start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable ppp-rfd.service
sudo systemctl start ppp-rfd.service
```

Stop any manually started PPP:
```bash
sudo pkill pppd
```

Verify:
```bash
systemctl status ppp-rfd.service
ifconfig ppp0
```

---

## Step 8 — Headless Field Test (No Infrastructure)

### Test conditions
- Pi Ethernet disconnected
- Pi Wi-Fi unavailable
- Mac Wi-Fi off
- No SSH access at boot

### Test procedure
1. Power off Pi
2. Power on Pi
3. Do nothing on the Pi
4. On the Mac:
   ```bash
   sudo pppd call rfd
   ```
5. Verify:
   ```bash
   ping 10.0.0.2
   ssh uav@10.0.0.2
   ```

Success indicates a fully headless radio network.

---

## Operational Notes

- Do **not** run `pppd` manually on the Pi once systemd is enabled
- Always use IP addresses (`10.0.0.x`), not hostnames
- MOOS apps should bind to `0.0.0.0` or `10.0.0.2`, not `localhost`

---

## Shortcuts / Quick Commands

### Mac (Day-to-Day Use)

Start radio network:
```bash
sudo pppd call rfd
```

Stop PPP cleanly:
```bash
sudo pkill pppd
```

Check interface:
```bash
ifconfig ppp0
```

SSH over radio:
```bash
ssh uav@10.0.0.2
```

Check who owns the serial port:
```bash
lsof | grep tty.usbserial
```

Kill stray screen sessions:
```bash
screen -ls
screen -S <id> -X quit
```

---

### Pi (Maintenance Only)

Check PPP service:
```bash
systemctl status ppp-rfd.service
```

Restart PPP service:
```bash
sudo systemctl restart ppp-rfd.service
```

Stop PPP service:
```bash
sudo systemctl stop ppp-rfd.service
```

Disable PPP on boot:
```bash
sudo systemctl disable ppp-rfd.service
```

Check interface:
```bash
ifconfig ppp0
```

---

## End State

- RFD900x radios function as **IP infrastructure**
- Pi boots **already network-ready**
- No MAVLink dependency
- MOOS-IvP runs over standard networking
- Ground station controls network presence

This matches real-world field robotics deployments.
