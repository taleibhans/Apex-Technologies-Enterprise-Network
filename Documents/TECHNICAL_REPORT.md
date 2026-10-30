# Apex Technologies — Technical Implementation Report

## 1. Executive summary

This report documents a simulated enterprise network for Apex Technologies, a fictional organization with segregated departments, central shared services, redundant network devices, and a planned secure public-network boundary. It records the **observed build and tests** and distinguishes functioning elements from attempted security enhancements. The companion [README](../README.md) is intentionally concise.

### Scope and delivery status

- **Built:** Four departmental/floor access-switch areas, one server access switch, two Layer 3 distribution switches, two edge routers, ISP router, centralized DHCP/DNS/HTTP services, representative endpoints, wireless access devices and printers, now assigned to dedicated VLAN 90.
- **Configured and tested:** Department DHCP, internal website name resolution and HTTP, inter-VLAN connectivity in successful tests, routed edge links, OSPF FULL adjacencies and learned routes, external test HTTP **prior to firewall changes**.
- **Configured but not fully validated:** HSRP failover, end-to-end resilience under physical fault, complete management accessibility under sustained testing.
- **Security exploration:** ACL revisions/rollback and insertion of an inline ASA 5505 between ISP and edge routers. The final security design was **not validated as an operational firewall policy**; post-cutover external reachability failed in at least one test.

## 2. Physical and logical topology

![Actual Packet Tracer topology](../images/network-topology.png)

The attached screenshot represents the **latest visible iteration**, including the ASA firewall. From north to south it includes:

1. **Public test area:** `R-ISP` (Cisco 2911) and `SRV-EXTERNAL` on the ISP's simulated external network.
2. **Perimeter:** Cisco **ASA 5505** `FW-ASA` placed inline between R-ISP and the company's two edge routers. Its interfaces and overall cabling are visible; functional traffic inspection and logs remain unverified.
3. **Edge:** `R1-EDGE` and `R2-EDGE`. Both retain connections toward the distribution tier.
4. **Distribution:** `DSW1` and `DSW2` (multilayer switches), linked to each other and dual-connected to access switches. SVIs supply VLAN routing and HSRP gateway addresses; the distribution layer participates in OSPF Area 0 with edge routers.
5. **Access:** Five access switches: Floor 1 Sales/Guest, Floor 2 HR/Finance, Floor 3 Engineering, Floor 4 IT/Management, plus a separate server switch.
6. **Servers and endpoints:** Dedicated DHCP, DNS and HTTP service hosts, an external test server, desktop/laptop endpoints, printers, a management IP phone, and wired wireless-access devices.

The dashed black links in the topology image depict inter-switch/cross-tier cabling, **not** firewall policy decisions. Access devices connect to both distribution switches; STP may block one redundant Layer 2 path per VLAN to avoid loops. An ASA at a common perimeter point simplifies policy enforcement but is also a **single point of failure** without a high-availability pair.

**No dedicated three-legged DMZ segment is verified in the screenshot.** The external server is attached to the ISP test router; it should not be described as a protected DMZ web server.

## 3. VLAN design and addressing

The enterprise uses the `10.40.0.0/16` private address allocation with VLSM subnets. Each VLAN is assigned a virtual gateway ending in `.1`; on the distribution layer, DSW1's SVI ends in `.2` and DSW2's ends in `.3`.

| VLAN | Name | Subnet | Mask | HSRP VIP | Client addressing |
|---:|---|---|---|---|---|
| 10 | Sales | `10.40.190.0/25` | `255.255.255.128` | `10.40.190.1` | DHCP |
| 20 | HR | `10.40.222.0/27` | `255.255.255.224` | `10.40.222.1` | DHCP |
| 30 | Finance | `10.40.221.0/26` | `255.255.255.192` | `10.40.221.1` | DHCP |
| 40 | Engineering | `10.40.126.0/25` | `255.255.255.128` | `10.40.126.1` | DHCP |
| 50 | IT | `10.40.189.0/26` | `255.255.255.192` | `10.40.189.1` | DHCP |
| 60 | Management | `10.40.220.0/27` | `255.255.255.224` | `10.40.220.1` | Static |
| 70 | Guest | `10.40.125.0/25` | `255.255.255.128` | `10.40.125.1` | DHCP |
| 80 | Servers | `10.40.219.0/26` | `255.255.255.192` | `10.40.219.1` | Static |
| 90 | Printers | `10.40.218.0/27` | `255.255.255.224` | `10.40.218.1` | Static |

Management client endpoints were configured as `10.40.220.10` and `10.40.220.11` with mask `/27`, virtual gateway `.1`, and internal DNS `10.40.219.11`. They initially failed DHCP because **VLAN 60 intentionally had no DHCP pool**. An APIPA address was therefore a symptom of missing DHCP service for that segment, not proof of ACL failure.

### Dedicated printer segment (VLAN 90)

As a later incremental upgrade, the Finance and Engineering printers were placed into dedicated VLAN 90 access ports on their respective floor switches, with VLAN 90 available across the distribution/uplink fabric. Each printer received a static address: **Finance Printer `10.40.218.10/27`** and **Engineering Printer `10.40.218.11/27`**, both using HSRP virtual gateway **`10.40.218.1`**. Distribution SVIs use `.2` and `.3` in this subnet. The user reported a successful ping after configuration, demonstrating tested IP reachability to a printer; the exact source/destination of that ping was not recorded in the supplied evidence. No printer-specific ACL policy was verified. A distinct VLAN provides its own Layer 2 broadcast domain, but inter-VLAN routing still permits traffic without separate restrictions.

### DHCP pool ranges used in the lab

| VLAN | Start | Address count |
|---:|---|---:|
| 10 | `10.40.190.10` | 60 |
| 20 | `10.40.222.4` | 25 |
| 30 | `10.40.221.10` | 30 |
| 40 | `10.40.126.10` | 100 |
| 50 | `10.40.189.10` | 35 |
| 70 | `10.40.125.10` | 100 |

Distribution SVIs on client VLANs are configured with `ip helper-address 10.40.219.10` to relay DHCP requests to the server VLAN. VLANs 60, 80 and 90 use static addresses, with no printer DHCP pool.

## 4. Shared infrastructure services

| System | Address | Role |
|---|---|---|
| `SRV-DHCP` | `10.40.219.10/26` | DHCP leasing to six client VLANs |
| `SRV-DNS` | `10.40.219.11/26` | A record for `intranet.apex.local` |
| `SRV-WEB` | `10.40.219.12/26` | Internal HTTP service |
| `SRV-EXTERNAL` | `203.0.113.10/24` | Simulated external HTTP testing |

The server-side virtual gateway is `10.40.219.1`. DNS maps `intranet.apex.local` to `10.40.219.12`. Departmental DHCP clients were expected to receive DNS `10.40.219.11`.

**DHCP incident:** An initial Sales client DHCP attempt failed. Packet Tracer Simulation Mode showed DHCP traffic arriving at the correct server on UDP port 67 but the server reporting that no service was listening. After enabling the DHCP service, lease acquisition worked. This isolated the issue to the server service rather than trunk/SVI routing.

## 5. Routing, high availability, and transit addressing

### Layer 2 redundancy and HSRP

Each floor/server access switch has uplinks to both distribution switches. STP prevents forwarding loops and explains why Packet Tracer sometimes displayed frame copies being dropped on blocked ports. The two distribution devices were configured with VLAN SVIs, Layer 3 routing, and HSRP virtual gateways. Active/standby priorities were divided by VLAN for load distribution. A formal HSRP gateway failover test was **not completed**.

### OSPF Area 0

Four distribution-to-edge point-to-point links used:

| Link | Distribution | Edge |
|---|---|---|
| DSW1—R1-EDGE | `10.40.250.1/30` | `10.40.250.2/30` |
| DSW1—R2-EDGE | `10.40.250.5/30` | `10.40.250.6/30` |
| DSW2—R1-EDGE | `10.40.250.9/30` | `10.40.250.10/30` |
| DSW2—R2-EDGE | `10.40.250.13/30` | `10.40.250.14/30` |

OSPF router IDs:

- DSW1: `1.1.1.1`
- DSW2: `2.2.2.2`
- R1-EDGE: `3.3.3.3`
- R2-EDGE: `4.4.4.4`

Area 0 adjacency outputs showed `FULL` between DSW1 and both edge routers and between R1-EDGE and both distribution switches. The edge routing table contained `O` routes to corporate subnets, sometimes with **two equal-cost next hops**. This verifies control-plane neighbor formation and route exchange, not actual reconvergence after a failure.

### ISP and external testing

Before the firewall redesign, R-ISP linked directly to both edge routers using `172.20.0.0/30` and `172.20.0.4/30`. The ISP's external test network was `203.0.113.0/24`, with router interface `203.0.113.1` and web server `203.0.113.10`. Client browsers successfully loaded the external test webpage. Initial ICMP replies were inconsistent (one captured attempt returned 1/4), so the project does **not** claim uninterrupted external packet delivery.

After insertion of the ASA, addressing and default routes were redesigned around firewall inside/outside interfaces. Connectivity from Sales to the external server subsequently failed. The firewall cutover therefore remains a prototype; the old topology's successful HTTP test is **not** evidence that the final firewall path works.

## 6. Security objectives and implementation status

See the standalone [security design](SECURITY_DESIGN.md) for the proposed policy matrix and control limitations.

- Department VLAN separation is a **logical segmentation boundary**, not proof of traffic isolation while unrestricted inter-VLAN routing is enabled.
- Extended Cisco ACLs were drafted and tested through multiple revisions, but stable intended enforcement was not established. All tested ACL assignments were subsequently rolled back; no working end-state ACL policy is claimed.
- A Cisco ASA 5505 was inserted inline between R-ISP and both edge routers in the latest visible network. Interface addressing, routing, NAT and ICMP inspection were attempted. **Complete traffic inspection, log collection, and successful post-cutover end-to-end connectivity are not verified.**
- No separate DMZ subnet is demonstrated in the provided topology; a DMZ is a future enhancement, not an accomplished component.

## 7. Test record and evidence grading

| Test | Observed result | Claim supported |
|---|---|---|
| DHCP client addressing | Six DHCP-enabled departments reported working | DHCP and helper functionality in tested state |
| Internal DNS + HTTP | `intranet.apex.local` loaded | Internal service access in tested state |
| Distribution-to-edge ICMP | Four point-to-point pings each returned 4/5 on initial attempt | All four routed links reachable |
| OSPF neighbor formation | `FULL` adjacency state on DSW1 and R1-EDGE | Neighbor establishment |
| OSPF route exchange | R1 learned departmental routes with `O` entries and redundant next hops | Dynamic route exchange |
| ISP-to-edge ICMP | Two initial pings returned 4/5 | ISP transit links reachable in old topology |
| External HTTP | Webpage displayed at `203.0.113.10` before ASA | Pre-firewall external HTTP connectivity |
| Management to Finance | Some runs 4/4, others timed out | Intermittent, not consistently reliable |
| ACL enforcement | Intended result not consistently demonstrated | **Not validated** |
| Firewall traffic and logging | External traffic failed after cutover; no confirmed log proof | **Not validated** |
| Failover resilience | No complete unplug/reconvergence test documented | **Not validated** |
| Printer VLAN upgrade | Static printer settings and successful ping reported | Basic printer reachability; printer-specific ACLs not tested |

### Packet Tracer simulation observations

- **STP blocked port:** A dropped redundant frame is expected if the port is not forwarding.
- **ARP resolution:** A first ICMP packet may be dropped while an address is resolved; this can explain some initial loss but does not prove the cause of every timeout.
- **ICMP administrative prohibition:** A captured `Type 3 / Code 13` message showed DSW1 generating an administratively prohibited response toward Finance. Screenshots of SVI VLAN 30 and 60 later showed no inbound or outbound access list, so the precise source of that earlier event was not conclusively resolved.
- **Missing VLAN-tag error:** Later Simulation Mode displayed a frame discarded at an access switch because the local VLAN did not match the 802.1Q tag; it was not established whether this represented an incidental flooded copy or an actual traffic-path failure.

## 8. Engineering decisions and lessons learned

1. Verify control-plane state with `show ip ospf neighbor` and `show ip route ospf`, not only successful pings.
2. Test with a defined known-good baseline before applying an ACL or perimeter change.
3. Treat an ASA cutover as a routing/NAT/inspection migration, not a drop-in cable change.
4. Distinguish observed working states from aspirational policy goals and prototype configurations.
5. Keep saved Packet Tracer milestones, including pre-firewall snapshots, to isolate regressions.

## 9. Future engineering work

1. Stabilize sustained intra-VLAN/inter-VLAN reachability and audit STP, ARP, VLAN trunk allow-lists, and HSRP roles.
2. Verify routed return paths and address translation across the ASA, followed by rule-hit and syslog evidence.
3. Establish a separate DMZ segment only when configured and tested.
4. Implement and verify departmental and Guest isolation with a controlled policy matrix and positive/negative tests.
5. Exercise HSRP, OSPF, and perimeter link/device failover and record restoration behaviour.
6. Add management-plane controls (SSH, least-privilege accounts, administrative VLAN hardening) once tested.

## Screenshot evidence index

The following are the selected device combinations for this project. Image files other than the topology should be placed in `images/`; links become active once those screenshots have been added.

| Screenshot | Device(s) shown | Evidence focus |
|---|---|---|
| [01 — Topology](../images/network-topology.png) | Full network | Architecture and firewall placement |
| [02 — VLANs/trunks](../images/vlan-trunks.png) | DSW1 | VLAN database and trunks |
| [03 — HSRP](../images/hsrp-status.png) | DSW1 + DSW2 | Gateway redundancy configuration |
| [04 — DHCP](../images/dhcp-engineering-guest.png) | DHCP server + **Engineering PC3 + Guest PC1** | Two DHCP client VLANs |
| [05 — OSPF](../images/ospf-neighbors-routes.png) | DSW1 + R1-EDGE | OSPF neighbors/routes |
| [06 — DNS/intranet](../images/dns-finance-intranet.png) | DNS server + **Finance PC1** | Internal DNS and HTTP |
| [07 — External HTTP](../images/external-web-sales-pre-asa.png) | **Sales PC1** | Pre-ASA external access |
| [08 — ASA](../images/asa-config.png) | FW-ASA | Configuration only; security effectiveness unverified |
| [09 — Floor 4 switch](../images/floor4-access-vlans.png) | **Floor 4 access switch** | IT/Management access segmentation |
| [10 — Printer VLAN](../images/10-printer-vlan.png) | Floor 2 + Floor 3 switches and printer IP configuration | VLAN 90 printer port membership/static addressing |

See [the Evidence Guide](EVIDENCE_GUIDE.md) for page names, commands, and precise captions.

## 10. Reproducibility and artifacts

This repository contains the current topology image and documentation. A Packet Tracer `.pkt` file can be added to `packet-tracer/` after selecting the intended saved version. The screenshot is of the **ASA-inclusive iteration**, which is not equivalent to the verified pre-ASA operational baseline. Do not infer final firewall success from the topology image alone.

See [Evidence Guide](EVIDENCE_GUIDE.md) for filenames and instructions that avoid creating new or misleading evidence.
