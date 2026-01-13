# Installation Guide

This guide walks you through setting up a Raspberry Pi with MOOS-IvP for the Skywalker X8 UAV platform.

## Overview

The installation process involves:
1. Preparing your development machine (macOS, Linux, or Windows)
2. Flashing Raspberry Pi OS to an SD card
3. Configuring SSH and network access
4. Installing MOOS-IvP and dependencies
5. Building custom UAV applications

Estimated time: 2-3 hours

## Prerequisites

### Hardware Required
- Raspberry Pi 4B (4GB or 8GB RAM recommended)
- MicroSD card (32GB or larger, Class 10)
- SD card reader
- USB-C power supply for Raspberry Pi
- Development computer (macOS, Linux, or Windows)

### Software Required
- Raspberry Pi Imager
- SSH client (built into macOS/Linux, or PuTTY for Windows)
- Git
- Text editor

## Part 1: Development Machine Setup (macOS)

### Install Homebrew (macOS)

If you don't have Homebrew installed:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Install Required Tools

```bash
brew install ssh-copy-id git
```

### Install Raspberry Pi Imager

Download from [raspberrypi.com](https://www.raspberrypi.com/software/) or install via Homebrew:

```bash
brew install --cask raspberry-pi-imager
```

## Part 2: Flash Raspberry Pi OS

### 1. Prepare SD Card

1. Insert the SD card into your card reader
2. Open Raspberry Pi Imager
3. Click "Choose OS"
4. Select **"Raspberry Pi OS Lite (64-bit)"** (headless operation, no desktop environment)

### 2. Configure Advanced Settings

1. Click the gear icon (⚙) or press `Cmd+Shift+X` to open advanced settings
2. Configure the following:

**Hostname**
```
skywalker1.local
```
*Note: Change this to match your UAV identifier (skywalker2, skywalker3, etc.)*

**Enable SSH**
- ✅ Enable SSH
- Use password authentication (for initial setup)

**Set Username and Password**
- Username: `uav`
- Password: (choose a strong password)

**Configure Wireless LAN**
- SSID: Your Wi-Fi network name
- Password: Your Wi-Fi password
- Wireless LAN country: `US` (or your country code)

**Locale Settings**
- Time zone: Your timezone (e.g., `America/New_York`)
- Keyboard layout: `us`

### 3. Write Image

1. Click "Choose Storage" and select your SD card
2. Click "Write"
3. Wait for the process to complete (5-10 minutes)
4. Eject the SD card

## Part 3: First Boot and SSH Setup

### 1. Boot the Raspberry Pi

1. Insert the SD card into your Raspberry Pi
2. Connect power
3. Wait 1-2 minutes for first boot

### 2. Connect via SSH

From your development machine:

```bash
ssh uav@skywalker1.local
```

Enter the password you set during imaging.

**Troubleshooting**: If connection fails:
- Wait a bit longer (first boot can take time)
- Check that both devices are on the same network
- Try the IP address instead: `ssh uav@<ip-address>`

### 3. Generate SSH Key (Recommended)

Create a dedicated SSH key for your Pi:

```bash
ssh-keygen -t ed25519 -C "pi-skywalker" -f ~/.ssh/id_ed25519_skywalker1
```

You can optionally set a passphrase for additional security.

### 4. Copy Public Key to Pi

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_skywalker1.pub uav@skywalker1.local
```

### 5. Configure SSH Convenience

Edit `~/.ssh/config` on your development machine:

```bash
Host skywalker1
    HostName skywalker1.local
    User uav
    IdentityFile ~/.ssh/id_ed25519_skywalker1
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes
```

Add key to SSH agent (macOS):

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_skywalker1
```

### 6. Test Passwordless Login

```bash
ssh skywalker1
```

You should connect without entering a password.

**Troubleshooting**: If password is still required, fix permissions on the Pi:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R uav:uav ~/.ssh
```

## Part 4: Update System and Install Dependencies

### 1. Update Package Lists

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

### 2. Disable Package Kit (Optional)

This prevents automatic updates during manual installations:

```bash
sudo systemctl disable --now packagekit
```

### 3. Install Essential Tools

```bash
sudo apt-get --assume-yes install \
    emacs \
    wget \
    cmake \
    subversion \
    screen \
    ncdu \
    watch \
    git \
    build-essential \
    curl \
    file \
    ninja-build \
    pkg-config
```

### 4. Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

## Part 5: Install MOOS-IvP Stack

### 1. Install uav-common

**Note**: This is a private repository. Contact chbenj36@gmail.com for access.

```bash
cd ~
git clone https://github.com/cbenjamin23/uav-common.git
cd ~/uav-common
./build.sh
```

Add to `~/.bashrc`:

```bash
echo 'git -C ~/uav-common pull --quiet --ff-only && source ~/uav-common/bashrc_common.sh' >> ~/.bashrc
source ~/.bashrc
```

### 2. Install MOOS-IvP Core

```bash
cd ~
git clone https://github.com/moos-ivp/moos-ivp.git
cd ~/moos-ivp
./build.sh
```

**Note**: On Raspberry Pi, use `-j2` flag for parallel compilation to avoid memory issues:

```bash
./build.sh -j2
```

Build time: ~30-45 minutes on Raspberry Pi 4.

### 3. Install moos-ivp-swarm (Private Repository)

**Note**: This is a private repository. Contact the team for access.

```bash
cd ~
git clone git@github.com:pavlab-MIT/moos-ivp-swarm.git
cd ~/moos-ivp-swarm
./build.sh -j2
```

### 4. Install moos-ivp-uav-base with MAVSDK

**Note**: This is a private repository. Contact chbenj36@gmail.com for access.

```bash
cd ~
git clone git@github.com:cbenjamin23/moos-ivp-uav-base.git
cd ~/moos-ivp-uav-base
git submodule update --init --recursive
chmod +x ~/moos-ivp-uav-base/scripts/setup_bash_aliases_moos.sh
```

#### Build MAVSDK

MAVSDK provides the communication layer between MOOS-IvP and ArduPilot:

```bash
cd ~/moos-ivp-uav-base/MAVSDK
cmake -S . -B build/default -G Ninja \
    -DSUPERBUILD=ON \
    -DCMAKE_BUILD_TYPE=Debug \
    -DBUILD_SHARED_LIBS=ON \
    -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

cmake --build build/default -j2
```

Build time: ~45-60 minutes on Raspberry Pi 4.

#### Build UAV MOOS Applications

```bash
cd ~/moos-ivp-uav-base
./build.sh -j2
```

## Part 6: Verify Installation

Run these commands to verify your installation:

```bash
# Check MOOS-IvP binaries
which MOOSDB || echo "MOOSDB not in PATH"
which pAntler || echo "pAntler not in PATH"

# Check UAV build
test -d ~/moos-ivp-uav-base/build || echo "UAV build missing"

# Check MAVSDK
ls ~/moos-ivp-uav-base/MAVSDK/build/default || echo "MAVSDK missing"

# Test MOOS-IvP
MOOSDB --help
```

Expected output should show version information and help text for each command.

## Installation Notes

### Raspberry Pi vs. macOS Build Differences

**Raspberry Pi (ARM64)**
- Use `-j2` flag for parallel compilation (avoid memory exhaustion)
- Use `minrobot` build configuration when available
- Expect longer build times (30-60 min per major component)
- May need swap space for large builds

**macOS (Intel/Apple Silicon)**
- Can use more parallel jobs: `-j4` or `-j8`
- Install dependencies via Homebrew
- Faster build times
- Different paths and library locations

### Common Build Issues

**Out of Memory**
- Reduce parallel jobs: `-j1` or `-j2`
- Add swap space if needed
- Close other applications during build

**Missing Dependencies**
- Run `apt-get update && apt-get upgrade`
- Check that all packages from Part 4 are installed
- Some tools may require additional system libraries

**Permission Errors**
- Ensure you're not running as root
- Check file ownership: `ls -la ~/`
- Fix if needed: `chown -R uav:uav ~/moos-ivp*`

## Next Steps

After successful installation:

1. **[Quick Start Guide](quickstart.md)** - Run your first MOOS mission
2. **[Radio Setup](radio-setup.md)** - Configure telemetry and mesh networking
3. **[Firmware Guide](firmware.md)** - Flash ArduPilot to your autopilot
4. **[Usage Guide](usage.md)** - Learn to operate the system

## Repository Access

Several repositories mentioned in this guide are private. To request access:
- **uav-common**: chbenj36@gmail.com
- **moos-ivp-uav-base**: chbenj36@gmail.com
- **moos-ivp-swarm**: team@pavlab-mit.org

## Additional Resources

- [MOOS-IvP Documentation](https://oceanai.mit.edu/moos-ivp/)
- [ArduPilot Documentation](https://ardupilot.org/ardupilot/)
- [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)

---

**Previous**: [Documentation Home](index.md) | **Next**: [Quick Start Guide](quickstart.md)
