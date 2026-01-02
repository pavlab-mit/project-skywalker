# Autonomous Flight Instructions (Manual Launch)

## Purpose
These instructions describe how to safely transition a fixed-wing UAV from a **manual hand launch** to a **fully autonomous mission**, without using autonomous launch.

---

## Pre-Launch Checklist
Before launching the aircraft:

1. **Verify Ground Station Connectivity**
   - Confirm that your ground station is successfully receiving telemetry from the aircraft.
   - Ensure position data (GPS fix, heading, altitude) is updating correctly in pMarineViewer.

2. **Prepare Initial Autonomous Directive**
   - In pMarineViewer, pre-configure an initial behavior:
     - Either **go to the first waypoint**, or  
     - **loiter above the operator / launch point**
   - Do *not* engage the mode yet—only ensure the directive is ready to activate.

3. **Confirm Transmitter Mode Mapping**
   - Verify that one transmitter switch position corresponds to **GUIDED mode** (or the equivalent autonomous control mode).
   - Confirm that manual control remains available in the default switch position.

---

## Launch and Transition Procedure

1. **Perform Manual Launch**
   - Hand-launch the aircraft using standard manual procedures.
   - Maintain manual control immediately after launch to ensure stable airspeed and climb.

2. **Set Autonomous Directive**
   - Once the aircraft is airborne and stable:
     - Use pMarineViewer to **activate** the chosen directive (first waypoint or loiter).

3. **Engage Autonomous Mode**
   - Flip the transmitter switch to the position corresponding to **GUIDED mode**.
   - The aircraft should immediately begin executing the selected autonomous behavior.

4. **Monitor Aircraft Behavior**
   - Observe flight path, altitude, and responsiveness.
   - Be ready to switch back to manual control if behavior deviates unexpectedly.

---

## Recommended Testing Practice

- **Start with Autonomous Loiter**
  - Using a loiter command as the first autonomous behavior is strongly recommended for testing.
  - This allows you to:
    - Verify reliable command and telemetry links
    - Confirm that coordinate frames and positioning between the aircraft and ground station are consistent
    - Observe stable autonomous control before committing to longer missions

- Once loiter behavior is confirmed to be stable and predictable, proceed to waypoint-based missions.

---

## Operator Reminder
If anything appears incorrect—unexpected turns, altitude changes, or loss of telemetry—**immediately revert to manual mode** and regain control.
