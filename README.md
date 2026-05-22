# Cisco Router NAT and ACL Advanced Troubleshooting Lab

An advanced Cisco Packet Tracer enterprise networking lab focused on diagnosing, isolating, and resolving misconfigurations within Dynamic NAT (PAT Overload), Extended/Standard Access Control Lists (ACLs), and asymmetric static routing. 

##  Topology Overview
The network infrastructure connects a corporate LAN environment containing multiple subnets to an external Web Server hosting services on `8.8.8.8` via serial point-to-point links. The core engineering tasks involved implementing strict security access matrices while facilitating public internet access.

---

##  Phase 1: Troubleshooting & Root Cause Analysis

### 1. Initial Web Browser Connection Failure
Attempting to access the HTTP web server (`8.8.8.8`) from PC1 initially resulted in a continuous `Request Timeout` error.
* **Root Cause:** An incorrectly structured Extended Access List (ACL 100) on Router R1 was implicitly dropping return traffic, preventing the completion of the TCP three-way handshake.



### 2. Misconfigured Server Network Parameters
During initial isolation, the target Web Server (Server0) was found to have an improper subnet mask and a completely missing gateway configuration (`0.0.0.0`).
* **Root Cause:** The server was entirely incapable of routing return traffic out of its local segment back to the gateway router interface.



### 3. NAT Translation Validation Before Remediation
Executing `show ip nat translations` on Router R1 yielded empty translation tables during live traffic stimulation.
* **Root Cause:** Access List 1 was erroneously mapped to a non-existent network (`192.168.50.0`), and the PAT overload process was incorrectly bound to an inside interface instead of the outside serial link interface.



---

##  Phase 2: Configuration & Remediation Stage

### 4. Reconfiguring NAT Overload Boundaries on R1
The stale configurations were completely flushed, and Access List 1 was rewritten to match the active local LAN subnets. The `Serial0/1/1` interface was properly designated as the outside NAT boundary, initiating valid dynamic translations.

```baseline
R1(config)# no ip nat inside source list 1 interface Gig0/0/0 overload
R1(config)# access-list 1 permit 192.168.10.0 0.0.0.255
R1(config)# access-list 1 permit 192.168.20.0 0.0.0.255
R1(config)# ip nat inside source list 1 interface Serial0/1/1 overload
