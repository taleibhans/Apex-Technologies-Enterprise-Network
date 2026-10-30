# Apex Technologies — Enterprise Network Engineering Lab

**Cisco Packet Tracer | VLANs · HSRP · OSPF · DHCP Relay · DNS · Perimeter Security Design**

A simulated multi-floor enterprise network built for a fictional organization. The lab combines nine segmented VLANs, dual distribution switches, redundant edge routing, centralized infrastructure services, dedicated printer segmentation, and an **ASA 5505 perimeter-firewall integration prototype**.

![Apex Technologies network topology](images/network-topology.png)

## At a glance

| Area | Implementation / evidence |
|---|---|
| Network segmentation | 9 VLANs covering departments, Guest, Management, Servers, and dedicated printers |
| Resilience | Dual-homed access switches, HSRP virtual gateways, OSPF Area 0 |
| Infrastructure services | Centralized DHCP with relay, internal DNS, HTTP intranet |
| Shared printers | VLAN 90 (`10.40.218.0/27`), static printer addresses, HSRP gateway; successful printer ping test |
| Routing validation | OSPF neighbors reached **FULL**; redundant learned routes observed |
| External testing | Simulated external HTTP server reachable **before** ASA cutover |
| Security | Departmental ACL policy designed; ASA 5505 added inline as a **work-in-progress** |

**Validation note:** Core routing and services were tested successfully during the build. ACL enforcement, firewall inspection/logging, and end-to-end connectivity **after** the firewall cutover were not conclusively verified. These are documented as limitations, not completed controls.

## Repository guide

- **[Technical report](docs/TECHNICAL_REPORT.md)** — architecture, addressing, implementation, verification, challenges, and next steps
- **[Security design](docs/SECURITY_DESIGN.md)** — intended access matrix, firewall placement, and current implementation status
- **[Evidence guide](docs/EVIDENCE_GUIDE.md)** — 10 screenshot combinations, including Engineering/Guest DHCP, Finance intranet, and Floor 4 VLANs
- **[Packet Tracer folder](packet-tracer/)** — place the `.pkt` project file here once you've chosen which saved version to publish

## Skills demonstrated

Enterprise subnetting and VLAN design · dedicated printer VLAN · Layer 2 trunking and STP · Layer 3 SVIs and DHCP relay · HSRP · OSPF adjacency and route verification · Packet Tracer PDU analysis · Network security architecture and troubleshooting.

*Built in Cisco Packet Tracer; all addressing and public-facing services are simulated.*
