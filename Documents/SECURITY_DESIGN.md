# Security Design — Apex Technologies

## Objective

Protect a simulated enterprise that has privileged Management and IT segments, standard business departments, a Guest network, internal services, and public-facing connectivity. This is the **intended policy**, not proof that enforcement was successfully deployed.

## Trust model

| Source / zone | Intended access | Intended restriction |
|---|---|---|
| Management VLAN 60 | Authorized traffic to all corporate VLANs | No equivalent broad access initiated from ordinary departments |
| IT VLAN 50 | Approved administration and infrastructure troubleshooting | Least privilege where feasible |
| Sales VLAN 10 | Assigned business services, DNS, DHCP, intranet, permitted external resources | No lateral access to Finance/HR/Management |
| HR VLAN 20 | Assigned business services | No lateral access to unrelated departments/Management |
| Finance VLAN 30 | Assigned business services | No lateral access to unrelated departments/Management |
| Engineering VLAN 40 | Assigned business services | No lateral access to sensitive departments/Management |
| Guest VLAN 70 | Public/external resources and minimal supporting DNS/DHCP | No corporate VLAN access |
| Servers VLAN 80 | Deliver approved shared services | No uncontrolled initiation into client segments |
| Printers VLAN 90 | Permit authorized internal printing and administration (design objective) | Printer-to-user lateral traffic to be restricted in a future ACL policy |
| Untrusted/public area | Approved services only | No direct route to internal private VLANs |

## Device placement in latest topology

`R-ISP → FW-ASA → {R1-EDGE, R2-EDGE} → {DSW1, DSW2} → access VLANs`.

The pictured ASA is **inline**, not a separate firewall-protected DMZ branch. This is meaningful because all working internet-bound corporate traffic would have to cross the firewall, once routing, NAT and inspection are operational. The single ASA is a perimeter availability bottleneck until an HA design is established.

## Control choices

- **Extended ACLs on SVIs:** Useful for explicit inter-VLAN deny/permit rules. Cisco interface ACLs are generally stateless; return traffic and rule ordering must be accounted for. HSRP peers need equivalent policies.
- **Stateful perimeter firewall:** Intended for connection tracking, inbound/outbound filtering, NAT/PAT, and security event logging. Rules cannot be considered enforced until both permitted and denied traffic are tested and log events captured.
- **Logging:** Proposed ASA buffered/syslog logging for allowed and denied test flows, ideally with a dedicated collector in a future revision. No functional log collector is evidenced yet.
- **DMZ:** A distinct public-facing network is a **future design option**. The external test server connected to R-ISP is not evidence of an isolated DMZ.

## Deployment truth table

| Control | Intended | Verified |
|---|---:|---:|
| VLAN segmentation | Yes | VLANs and addressing configured |
| OSPF route learning | Yes | Yes, observed |
| Management privilege model | Yes | No complete access-matrix verification |
| Department isolation ACLs | Yes | No; previous ACLs were rolled back |
| Guest-to-corporate isolation | Yes | Not confirmed |
| ASA inline placement | Yes | Present in topology image |
| Firewall permitted/denied traffic tests | Yes | Not yet |
| ASA logging and inspection evidence | Yes | Not yet |
| DMZ isolation | Future | Not built/verified |

## Suggested minimal test plan for future iteration

For each policy rule, collect a positive test (allowed application connection) and a negative test (blocked connection) from the appropriate source VLAN. Test Management→Finance/HR, Sales→Finance, Guest→internal server, Guest→external web, internal DHCP/DNS, and external→corporate. Record firewall/ACL counters and logs; verify routing/NAT and return packets before treating a dropped ping as proof of filtering. Preserve a working project backup before changes.

## Printer segmentation update

The two shared printers were moved to VLAN 90 (`10.40.218.0/27`), with static IPs `10.40.218.10` and `10.40.218.11` and gateway `10.40.218.1`. This separates printers at Layer 2 and prepares for future printer access policies; **printer-specific ACL enforcement is not claimed**.
