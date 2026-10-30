# Screenshot Evidence Guide



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
| 10 | `Printer-vlan.png` | **Floor 2 and Floor 3 switches:** Config → printer access port showing VLAN 90, plus **Finance Printer or Engineering Printer:** Config → FastEthernet0 showing its static IP | Dedicated VLAN 90 membership and printer addressing |
