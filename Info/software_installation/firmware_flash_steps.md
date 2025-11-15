# Correct Steps for Flashing ArduPlane Firmware onto CubeOrange / CubeOrange+

## 1. Disconnect Your Cube from USB
Ensure the Cube is not powered before starting the flashing process.

---

## 2. Open QGroundControl
Launch QGC, then navigate to:

**Vehicle Setup → Firmware**

You should see the message:  
*“Plug in your device via USB to start firmware upgrade.”*

---

## 3. Plug in the Cube via USB (Directly)
Connect your CubeOrange / CubeOrange+ directly to your computer via USB (no hubs).

QGC should detect:

- **Found device: CubePilot**
- **Bootloader Version: 5**
- **Board ID: 1063**

This confirms it is in bootloader mode.

---

## 4. Enable Advanced Settings
Check the following box:

✔️ **Advanced settings**

This is required to manually select the firmware file.

---

## 5. Select “Custom firmware file…”
At the bottom of the firmware setup panel, choose:

➡️ **Custom firmware file…**

This opens a file selection dialog.

---

## 6. Download the Correct Firmware
Download from one of the following directories:

### Recommended (latest build):
**https://firmware.ardupilot.org/Plane/latest/CubeOrangePlus/**

### Specific Version (e.g., 4.5.7):
**https://firmware.ardupilot.org/Plane/stable-4.5.7/CubeOrangePlus/**

Use the file:

### **`arduplane.apj`**

Avoid `.hex`, `.elf`, `.abi`, `.bin`, or broken CubeOrange folders.

---

## 7. QGC Verifies the Firmware File
After you select the `.apj` file, QGC will check:

```
Board ID: 1063
Successfully decompressed image
Erasing previous program...
```

A mismatch (e.g., 140 != 1063) means the wrong firmware — QGC will prevent flashing.

---

## 8. Flashing and Reboot
QGC will:

1. Erase previous firmware  
2. Upload the selected ArduPlane `.apj`  
3. Reboot the Cube automatically  

Once complete, QGC will reconnect and show the normal parameter menus.

---
