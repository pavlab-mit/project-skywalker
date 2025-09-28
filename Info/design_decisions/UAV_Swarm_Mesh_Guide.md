# 🚁 Mesh Networking for UAV Swarms --- Beginner's Guide

When building a UAV swarm, each drone needs to share data with the
others --- telemetry, commands, even video. The radios you choose
(long-range **serial telemetry radios** like the **RFD900x**, or
broadband **Wi-Fi radios** like the **Rocket M5**) only provide the
physical link. The real challenge is: **how do we route data across many
drones without a central ground station?**

This is solved with **mesh networking protocols**.

------------------------------------------------------------------------

## 🔹 Layer 2 vs Layer 3 Mesh

-   **Layer 2 (Data Link Layer):**\
    Works like a giant Ethernet switch. Everyone is in one big "room,"
    and frames are forwarded until they reach the right node.
    -   Example: **802.11s**, **BATMAN-adv** (kernel L2 mode).\
    -   ✅ Pros: Transparent to applications; easy to drop in.\
    -   ❌ Cons: Broadcast traffic floods the whole mesh; doesn't scale
        well in mobile UAV networks.
-   **Layer 3 (Network Layer):**\
    Works like the postal system. Each drone has an IP address, and a
    routing protocol decides the best path for every message.
    -   Example: **BATMAN classic**, **OLSR**, **AODV**.\
    -   ✅ Pros: Scales better, adapts to mobility, gives routing
        visibility.\
    -   ❌ Cons: Slightly more complex setup.

👉 For UAV swarms, **Layer 3 is usually the better choice** because it
handles mobility and scaling more gracefully.

------------------------------------------------------------------------

## 🔹 Radios and Best-Fit Protocols

### 1. **Telemetry Radios (RFD900x, 900 MHz sub-GHz)**

-   Nature: Long-range, low-bandwidth (tens of kbps), appear as **serial
    pipes**.\
-   To use with IP: Must wrap traffic in **PPP or SLIP** → creates a
    network interface (`ppp0`) → then run routing protocol.\
-   Best protocol: **BATMAN classic** (userspace, Layer 3).
    -   Lightweight: doesn't flood full topology.\
    -   Suits constrained links where every byte matters.\
-   ✅ **Use case:** Send swarm telemetry/heartbeats/commands reliably
    over long distances.

------------------------------------------------------------------------

### 2. **Wi-Fi Radios (Rocket M5, 5.8 GHz)**

-   Nature: Medium-range, high-bandwidth (tens--hundreds of Mbps),
    expose **Ethernet/IP directly**.\
-   Best protocols:
    -   **OLSR:** Proactive, full-topology routing → optimized multi-hop
        paths, easy debugging.\
    -   **BATMAN-adv:** Layer 2 variant → swarm appears as one LAN,
        great for MAVLink/ROS2.\
-   ✅ **Use case:** Share video, maps, and high-rate swarm
    coordination.

------------------------------------------------------------------------

## 🔹 Practical Swarm Pattern (single aircraft setup)

-   **Primary radio (Wi-Fi mesh, Rocket M5 + OLSR/BATMAN-adv):**
    High-bandwidth, short/medium-range, handles video + data sync.\
-   **Backup radio (RFD900x + PPP + BATMAN classic):** Long-range,
    resilient, low-rate control/telemetry link if Wi-Fi drops.

This **hybrid model** gives you both throughput and resilience.

------------------------------------------------------------------------

## ✅ Rule of Thumb

-   **RFD900x (serial telemetry):** → **BATMAN classic** over PPP/SLIP.\
-   **Rocket M5 (Wi-Fi broadband):** → **OLSR** (for routing visibility
    & optimized paths) or **BATMAN-adv** (if you want plug-and-play LAN
    feel).
