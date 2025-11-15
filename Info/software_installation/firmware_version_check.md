# Checking ArduPilot Firmware Version on macOS Using MAVProxy

This guide explains how to check the firmware version installed on a CubeOrange / CubeOrange+ using **MAVProxy** on macOS.

---

## 1. Install MAVProxy (If Not Already Installed)

Open Terminal and run:

```bash
pip3 install MAVProxy --user
```

This installs MAVProxy into your user Python environment.

---

## 2. Identify the Cube’s Serial Device

Run:

```bash
ls /dev/tty.usb*
```

You should see something like:

```
/dev/tty.usbmodem2101
```

This is the autopilot's serial port.

---

## 3. Connect to the Cube Using MAVProxy

Replace the serial device with your actual device name:

```bash
python3 -m MAVProxy.mavproxy --master=/dev/tty.usbmodem2101,115200
```

MAVProxy will begin waiting for a heartbeat.

---

## 4. Read the Firmware Version

Within a few seconds, you will see version information such as:

```
AP: ArduPlane V4.6.3 (3fc7011a)
AP: ChibiOS: 88b84600
Board: CubeOrange (ID: 1063)
```

The line starting with:

```
AP: ArduPlane Vx.x.x
```

is the current firmware version installed on your Cube.

---

## 5. Exit MAVProxy

Press:

```
CTRL + C
```

Or simply close the Terminal window.

---

