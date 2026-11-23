# University Campus Network — CCNA Packet Tracer Project

**Engineer:** Kabin Rijal  
**Platform:** Cisco Packet Tracer  
**Session:** Day 2 — November 22, 2026  
**Status:** Day 2 — work in progress — access layer and VLANs tested; uplink troubleshooting in progress

## Project scenario

A fictional university is expanding and needs a secure, reliable campus network for students, faculty, administration, IT, guests, and central services. The campus has four existing buildings and plans to expand. The design will eventually incorporate hierarchical campus architecture, inter-VLAN routing, DHCP, DNS, ACLs, wireless, redundancy, and troubleshooting.

I am approaching this as a network engineering project: identify customer requirements, select equipment, design and assign networks, configure, verify, and troubleshoot. This second lab session focuses on the access layer in Building B, Floor 1, and its connection to a distribution switch.

## Lab topology (current)

```text
             BLDB-DSW01
       Cisco 3560 multilayer switch
                Gi0/1
                  |
        802.1Q trunk on DSW01
        (2960 side not fixed yet)
                  |
                Gi0/1
             BLDB-F1-ASW01
            Cisco 2960 switch
             /    /    \    \
          Fa0/1 Fa0/2 Fa0/3 Fa0/4
           |      |      |     |
         Student Student Faculty Faculty
           01     02     01     02
```

## VLAN and IP addressing plan

| Role | VLAN | Subnet | Mask | Interface(s) on access switch |
|---|---:|---|---|---|
| Students | 10 | `10.20.0.0/23` | `255.255.254.0` | Fa0/1, Fa0/2 |
| Faculty | 20 | `10.20.2.0/25` | `255.255.255.128` | Fa0/3, Fa0/4 |

### End-device IPs

| Device | IP address | Prefix | Gateway |
|---|---|---|---|
| BLDB-F1-STUDENT1 | `10.20.0.50` | /23 | Not configured yet |
| BLDB-F1-STUDENT2 | `10.20.1.80` | /23 | Not configured yet |
| BLDB-F1-FACULTY1 | `10.20.2.10` | /25 | Not configured yet |
| BLDB-F1-FACULTY2 | `10.20.2.20` | /25 | Not configured yet |

**Subnet reasoning:** A /23 offers 512 total addresses (510 usable), enough for the 450 projected student devices in Building B. A /25 offers 128 total addresses (126 usable), enough for 100 faculty devices plus a gateway. These represent capacity targets, not all physically modeled clients.

## Tests performed

| Test | Observed result | Interpretation |
|---|---|---|
| Student PC 1 → Student PC 2 | Ping successful, 0% packet loss | Same VLAN and same /23 subnet |
| Faculty PC 1 → Faculty PC 2 | Ping successful, 0% packet loss | Same VLAN and same /25 subnet |
| Faculty PC 1 → Student PC 1 | Ping timed out | Different VLANs/subnets; no inter-VLAN routing or gateways configured |
| Distribution switch `show interfaces trunk` | Gi0/1 shows 802.1Q trunking | Trunk is operational on the 3560 side |
| Distribution switch allowed VLAN list | `10,20` | Only the two required VLANs are permitted on that uplink |
| Access switch `show interfaces trunk` | No trunk listed | 2960 side is not operating as a trunk |

## Troubleshooting log — unresolved uplink mismatch

**Symptom:** `show interfaces trunk` on the access switch returned no trunk interface, while the distribution switch reported `Gi0/1` as trunking.

**Evidence:** On access switch `BLDB-F1-ASW01`, `show interfaces gigabitEthernet 0/1 switchport` showed:

```text
Administrative Mode: dynamic auto
Operational Mode: static access
```

The switch also generated an STP inconsistency warning regarding port type on Gi0/1.

**Working hypothesis:** The switches disagree on the uplink's operational mode. The 3560 is set to a manual trunk, but the 2960 is still operating as an access port.

**Planned next action:** Explicitly configure Gi0/1 as an 802.1Q trunk on the 2960, verify operational trunking on both ends, and verify that only VLANs 10 and 20 are allowed. Then proceed with Layer 3 gateways and inter-VLAN routing on the 3560.

**Important:** The issue has been diagnosed, **not yet fixed**. This log preserves a real troubleshooting milestone.

## Commands and concepts practiced

- `enable`, `configure terminal`, `hostname`, `interface`, `vlan`, `name`
- `switchport mode access`, `switchport access vlan`, `switchport mode trunk`
- `switchport trunk allowed vlan 10,20`
- `show vlan brief`, `show interfaces status`, `show interfaces trunk`
- `show interfaces gigabitEthernet 0/1 switchport`
- `do` for running show commands from configuration mode; `?` for context help
- Layer 2 and Layer 3 differences; access/distribution/core roles
- ARP, ICMP ping tests, duplex, bandwidth, subnet masks, VLSM, 802.1Q, DTP, STP, and EtherChannel concepts

## Lessons learned

1. Creating a VLAN does not automatically assign switch ports to it.
2. Being in the same VLAN is not enough for IP communication; addressing also has to be compatible.
3. A /23 can span two adjacent /24-sized address ranges: `10.20.0.0` through `10.20.1.255`.
4. A trunk carries multiple VLANs but does **not** route between them.
5. An accepted configuration command is not the same as verified end-to-end operation.
6. I should compare *administrative mode* and *operational mode* when diagnosing trunk problems.

## Next session

- [ ] Correct the access-switch Gi0/1 trunk mode mismatch.
- [ ] Verify allowed VLANs and STP state on both ends.
- [ ] Configure Layer 3 SVIs for VLAN 10 and VLAN 20 on the 3560.
- [ ] Enable IP routing and set PC default gateways.
- [ ] Repeat inter-VLAN connectivity tests.
- [ ] Add ACLs to apply the university's access policy.
- [ ] Expand to additional floors and buildings; add DHCP and other services as the design grows.

## Screenshot evidence

The real screenshots from the November 22 Packet Tracer session were prepared and cropped to show the topology, VLAN assignments, student/faculty ping tests, 3560 trunk configuration, and the 2960 trunk mismatch. The screenshots have  they are supplied separately as a ready-to-upload ZIP. Do not interpret the screenshots as proof of successful inter-VLAN routing; that part of the project remains unfinished.


<img width="1018" height="477" alt="06_trunk_mismatch_on_2960" src="https://github.com/user-attachments/assets/cb7d2ca3-3e0b-4c92-ae31-04654a4585a6" />
<img width="1018" height="375" alt="05_trunk_configured_on_3560" src="https://github.com/user-attachments/assets/ff17fb92-0bf9-464b-ab08-332b883ad691" />
<img width="948" height="420" alt="04_faculty_ping_and_isolation" src="https://github.com/user-attachments/assets/a852220c-54d0-421e-8d97-b17bbe7b120b" />
<img width="948" height="330" alt="03_student_ping" src="https://github.com/user-attachments/assets/d4773478-6ec4-4747-954e-5c4006b717dd" />
<img width="1018" height="435" alt="02_vlan_assignments" src="https://github.com/user-attachments/assets/3b0523e9-564b-45c1-b6a7-f3382a31c6e8" />
<img width="1018" height="787" alt="01_topology" src="https://github.com/user-attachments/assets/b08227ff-141b-4d3b-a4d1-da32812a2ec2" />


