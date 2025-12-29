# RFD900x Asynchronous Mesh Setup Guide

To set up the **RFD900x radios** from **RFDesign** for **asynchronous mesh communication**, there are two main steps:

1. **Flash the correct asynchronous firmware**  
2. **Configure the radio registers to the proper values**

---

## 1. Flashing the Asynchronous Firmware

Unfortunately, the provided FTDI cable and documentation are not sufficient for flashing on macOS or Linux — save yourself an hour of troubleshooting and use **Windows** instead.

### Windows Instructions

1. **Download and install RFD Modem Tools V4**  
   [https://files.rfdesign.com.au/tools/](https://files.rfdesign.com.au/tools/)

2. **Connect your RFD900x radio**  
   - Use the provided **FTDI-to-USB cable**.  
   - Plug it into your Windows computer — it usually appears as **COM4**, though this may differ.  
   - The default **baud rate is 57,600 bps**, but you can also click **Scan** in Modem Tools to automatically detect the correct port and baud rate.

3. **Download the correct asynchronous firmware**  
   [https://files.rfdesign.com.au/firmware/](https://files.rfdesign.com.au/firmware/)  
   Look for:  
   ```
   RFDAsync V4.03 rfd900x2.gbl
   ```

4. **Flash the firmware**
   - Open **RFD Modem Tools** → go to the **Firmware Upload** tab.  
   - Click **Browse**, select your `.gbl` file, and click **Upload**.  
   - The radio will erase, flash, and verify the firmware automatically.

5. **Verify the firmware version**
   - Once complete, reconnect to the modem in the **Terminal** tab.  
   - Type:
     ```
     ATI
     ```
     You should see output such as:
     ```
     RFD900x Async V4.03
     ```
     confirming a successful flash.

---

## 2. Configuring Radio Registers

At this point, you can configure the radio registers either on **Windows** using the **RFD Modem Tools** interface or directly on **macOS/Linux** using a serial terminal. If you work with a mac normally, skip to the **macOS/picocom** section to get used to editing registers with your mac.

The parameters and AT command syntax are described in the official **RFD900x Asynchronous User Manual (V1.1)**:  
[https://rfdesign.com.au/wp-content/uploads/2024/04/RFD900x-Asynchronous-User-Manual-V1.1.pdf](https://rfdesign.com.au/wp-content/uploads/2024/04/RFD900x-Asynchronous-User-Manual-V1.1.pdf)

### Checking Current Settings

To view the current configuration, type:
```
ATI5
```
This displays all register values including baud rate, node ID, destination ID, transmit power, and antenna mode.

---

### Windows – RFD Modem Tools

In the **Terminal** tab of RFD Modem Tools, enter the following commands:

To view all current settings at any time:
```
ATI5
```

```
+++
ATS10=1         (Node ID)
ATS11=65535     (Destination ID)
ATS12=30        (Transmit power in dBm)
ATS23=2         (Antenna mode: auto-diversity)
AT&W            (Save to EEPROM)
ATZ             (Reboot)
```

Reconnect and type ```ATI5``` again to verify that the values match the desired configuration.

If you have multiple radios, assign a unique Node ID (ATS10) to each one. All radios should share the same Destination ID (65535) to enable broadcast mesh communication.

### Configuring on macOS or Linux with picocom
Note: This has been a bit iffy so far. Please consider finding a friend with a windows computer if you find yourself stuck at the "+++" step.

1. **Install picocom**
   ```
   brew install picocom
   ```

2. **Connect the radio**
   - Plug in the FTDI cable and find the device name:
     ```
     ls /dev/tty.*
     ```
     Example: `/dev/tty.usbserial-BG00FTB3`

3. **Open the serial connection**
   ```
   picocom -b 57600 /dev/tty.usbserial-BG00FTB3
   ```
   You should see:
   ```
   Terminal ready
   ```

4. **Enter command mode**
   Wait two seconds, then type (no Enter key):
   ```
   +++
   ```
   You should receive `OK`.

5. **Set the parameters**
Note: See the Asynchronous User Manual in the references section at the bottom to understand parameter/value pairs
   ```
   ATS10=1
   ATS11=65535
   ATS12=30
   ATS23=2
   AT&W
   ATZ
   ```

7. **Disconnect from picocom**
   Type the following to exit picocom:
   ```
   Ctrl-A  Ctrl-X
   ```

8. **Confirm the configuration**
   Reconnect and type:
   ```
   ATI5
   ```
   to verify that the values match the desired configuration.

---

## References

- **RFD900x Asynchronous User Manual (V1.1)**  
  [https://rfdesign.com.au/wp-content/uploads/2024/04/RFD900x-Asynchronous-User-Manual-V1.1.pdf](https://rfdesign.com.au/wp-content/uploads/2024/04/RFD900x-Asynchronous-User-Manual-V1.1.pdf)
- **RFD Modem Tools (Windows)**  
  [https://files.rfdesign.com.au/tools/](https://files.rfdesign.com.au/tools/)
- **RFD Firmware Downloads**  
  [https://files.rfdesign.com.au/firmware/](https://files.rfdesign.com.au/firmware/)
