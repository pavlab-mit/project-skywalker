# Comparison: Telemetry radios (RFD900x-class) vs Wi‑Fi CPE (Rocket M5‑class)

| System | Good for comms‑denied (no external infra) | Can find each other by **IP** natively | Mesh (multi‑hop, dynamic) | Typical throughput | Typical LOS range (ballpark) | Encryption / security | Best use case |
|---|---:|:---:|:---:|---:|---:|---:|---|
| **Telemetry radio** (RFD900x / sub‑GHz FHSS modems) | ✅ Yes — self‑contained FHSS/sub‑GHz links, works without towers | ❌ Not natively — radios are serial pipes; run PPP/SLIP to carry IP | ⚠️ Limited natively — supports fixed forwarding / broadcast; dynamic routing must run above the radio (OLSR/BATMAN over PPP) | **Low** — tens → a few hundred kbps | **Long** — tens of km at low bitrates (20–40+ km in ideal LOS) | ✅ Built‑in AES at radio layer (transparent) | Long‑range, low‑bandwidth C2/backup; comms‑denied resilience |
| **Wi‑Fi CPE** (Rocket M5 / NanoStation / MikroTik SXT — 5 GHz class) | ❌ Not inherently — more easily jammed in contested RF; needs LOS/antenna planning | ✅ Yes — native Ethernet/IP interface; each node gets IP easily | ✅ Yes — supports true dynamic mesh when running OpenWrt/RouterOS + BATMAN‑adv or OLSR | **High** — tens → hundreds of Mbps | **Medium → Long** — 0.5–2 km omni; 5–15 km with small directional panels; 20–50+ km with high‑gain dishes & towers (requires careful alignment) | ⚠️ Use WPA2/WPA3 or an overlay VPN (WireGuard); no radio‑level AES by default | High‑bandwidth swarm coordination, map/video sync, large dynamic meshes |

---

## Short summary / tradeoffs
- **Resilience vs bandwidth:** telemetry radios win for jam/denial resistance and long range at low data rates. Wi‑Fi CPE radios win for throughput, native IP, and scalable dynamic mesh.  
- **How IP is carried:** telemetry radios = **serial** by design → you add PPP/SLIP and a routing daemon (OLSR/BATMAN) to make an IP mesh. Wi‑Fi CPEs expose an IP/Ethernet interface natively.  
- **Mesh behavior:** Wi‑Fi CPEs (with mesh firmware) provide **true dynamic, self‑healing multi‑hop mesh**. RFD‑class radios require either manual forwarding rules or an IP mesh layered on top (which consumes RF airtime).  
- **Scale:** For dozens of nodes prefer Wi‑Fi CPE / hierarchical cells. For small groups or long links, prefer telemetry radios.  
- **Common pattern:** Hybrid — Wi‑Fi mesh for primary high‑bandwidth coordination; sub‑GHz telemetry (RFD) as encrypted, long‑range backup for heartbeats/C2.

---

# What OLSR and BATMAN are (short descriptions)

## OLSR (Optimized Link State Routing)
- **Type:** Proactive **link‑state** routing protocol for ad‑hoc networks (runs at IP layer / L3).  
- **How it works (brief):** nodes periodically exchange **HELLO** and **TC (Topology Control)** messages to learn the network graph. Each node computes shortest paths (like Dijkstra) and installs full routing tables.  
- **Pros:** routes are precomputed so forwarding is low‑latency; widely implemented (e.g., `olsrd`).  
- **Cons:** control traffic (HELLO/TC) can be heavy on low‑bandwidth links; every node keeps full topology (scales less well on narrowband or very large meshes).

## BATMAN (Better Approach To Mobile Ad‑hoc Networking)
- **Type:** Mesh routing approach; **BATMAN‑adv** is a kernel/Layer‑2 implementation (treats mesh like a bridged LAN). BATMAN classic exists in userspace/L3.  
- **How it works (brief):** nodes send periodic **Originator Messages (OGMs)**. Instead of maintaining the whole topology, each node records the **best next‑hop** to reach a destination (statistics on OGMs). Routing decisions are based on link quality metrics and best‑next‑hop tables.  
- **Pros:** lower control overhead than link‑state (better for constrained links), simpler next‑hop routing, transparent L2 bridging (BATMAN‑adv) — good for protocols that expect a LAN.  
- **Cons:** less global topology visibility (you don’t get full graph), may be less featureful for some routing policies; typically expects an Ethernet‑like interface (works best on native Wi‑Fi CPEs, not raw serial modems).

---

If you want, I can also attach a minimal **config checklist** (AT commands + PPP/olsrd lines for telemetry radios; OpenWrt + batman‑adv stanzas for Wi‑Fi CPE) as a second file. Just say the word and I’ll add it.