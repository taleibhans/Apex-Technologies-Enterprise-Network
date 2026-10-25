# Screenshot Evidence Guide

Use the screenshots **already captured**. The device combinations below reflect the actual images selected, rather than the original suggestions. Place each combined screenshot in `images/` under the indicated filename. Do not retake a successful screenshot simply to use another client device.

| # | Filename | Devices and screen/window arrangement | What it demonstrates |
|---|---|---|---|
| 01 | `network-topology.png` | Entire enterprise topology (**already included**) | Architecture, redundant distribution, four floor switches, separate server switch, inline ASA prototype |
| 02 | `vlan-trunks.png` | **DSW1 CLI:** `sh vlan brief` plus `sh int trunk` side-by-side | VLAN definitions and trunking |
| 03 | `hsrp-status.png` | **DSW1 and DSW2 CLI:** `sh standby brief` side-by-side | Configured Active/Standby virtual gateways (only claim verified states visible) |
| 04 | `dhcp-engineering-guest.png` | **DHCP Server → Services → DHCP**, **Engineering PC3 → Desktop → IP Configuration**, **Guest PC1 → Desktop → IP Configuration** | DHCP pool configuration and client addressing in Engineering VLAN 40 and Guest VLAN 70 |
| 05 | `ospf-neighbors-routes.png` | **DSW1 CLI:** `sh ip ospf nei`; **R1-EDGE CLI:** `sh ip ro ospf` | FULL OSPF adjacencies and learned routes |
| 06 | `dns-finance-intranet.png` | **DNS Server → Services → DNS** plus **Finance PC1 → Desktop → Web Browser** at `http://intranet.apex.local` | Name resolution configuration and Finance VLAN client access to internal HTTP service |
| 07 | `external-web-sales-pre-asa.png` | **Sales PC1 → Desktop → Web Browser** at `http://203.0.113.10`; include earlier ping evidence if saved | Successful external HTTP test **before** the inline ASA cutover |
| 08 | `asa-config.png` | **FW-ASA CLI:** interface summary and route/configuration output (where supported) | ASA configured/in topology; **not** proof of successful filtering, NAT or logging |
| 09 | `floor4-access-vlans.png` | **Floor 4 access switch (Switch4) CLI:** `sh vlan brief`, optionally beside its connected endpoints/topology | IT (VLAN 50) and Management (VLAN 60) access-port segmentation on Floor 4 |

## Important labeling details

- **#04 is Engineering PC3 + Guest PC1**, not Sales + Management. Guest obtains its address from the Guest DHCP pool; VLAN 60 Management uses static addressing and is not a DHCP-pool example.
- **#06 is Finance PC1**, not Sales PC1.
- **#07 remains Sales PC1**, using the documented successful **pre-firewall** test.
- **#09 is Floor 4**, showing IT and Management port separation, **not** the Floor 2 HR/Finance switch.
- Screenshot #03 must show actual HSRP states before describing failover as successful; an Active/Standby display does not itself prove a failover test.
- The latest network image has an inline ASA, but post-ASA traffic inspection/logging and external reachability were not conclusively verified.

## Where to place the screenshots

Put your images in `images/` with the names above. You can also use different names, but then update the links in `docs/TECHNICAL_REPORT.md`. The README intentionally displays **only the topology** to keep it short. The technical report is the place for screenshot evidence.
