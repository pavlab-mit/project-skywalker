# Comparison of Communication Options for UAVs

| System         | Good for Comms-Denied Communication (no external infra) | Ability to Find Each Other by IP Address | Mesh Communication |
|----------------|----------------------------------------------------------|------------------------------------------|---------------------|
| **RFD900x**    | ✅ Yes (self-contained, point-to-point radios)            | ❌ No (no IP stack, just serial telemetry) | ✅ Yes (supports mesh modes for multi-node MAVLink) |
| **Rocket M5**  | ✅ Yes (private Wi-Fi-style link, no towers/sats needed)  | ✅ Yes (Ethernet/IP-based, each node gets IP) | ❌ No (uses AP/client model, not a peer-to-peer mesh) |
| **Cellular (4G/5G dongle or tether)** | ❌ No (depends on carrier towers or satellites) | ✅ Yes (full IP addressing via SIM/carrier network) | ✅ Yes (via internet routing, but depends on provider infrastructure) |

---

## Summary

- **RFD900x** → Best for low-bandwidth, comms-denied environments, supports mesh, but not IP-based.  
- **Rocket M5** → Best for high-bandwidth private networks (video/data), self-contained, IP addressing works, but no mesh (all nodes connect to a ground AP).  
- **Cellular** → Gives global reach with IP/mesh via the internet, but relies on external infrastructure (not comms-denied).

