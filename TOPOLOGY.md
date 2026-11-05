# Rack 4E topology reference

Built from `HANDOVER.md` (4 Nov) and the interface descriptions and IPs in the `.ttl` macros (5 Nov). Use this file for redrawing the topology diagram. Config detail per device is in `plan.md`.

**5 Nov redesign:** the firewall links to the DSWs are now routed `/30`s (ASA E1/3 + E1/4, PA eth1/6 + eth1/7), all up. The server-subnet backups (PA `.78` on VLAN 69, ASA `.86` on VLAN 92) are removed. VLAN 100 is only the DSW1 <-> DSW2 transit over Po5. Live devices still run the old design until the macros are re-applied (`HANDOVER.md` section 10).

## 1. Devices

| Device | Model / role | OSPF router-ID | OOB mgmt IP (VLAN 99) | Mgmt port |
|---|---|---|---|---|
| R1 | Edge router, WAN to ISP1 | 1.1.1.1 | 172.16.0.33 (OOB gateway) | G0/1/0 |
| R2 | Backup transit router | 2.2.2.2 | 172.16.0.34 | G0/1/0 |
| DSW1 | Distribution, 9300L | 3.3.3.3 | 172.16.0.35 | SVI Vlan99 |
| DSW2 | Distribution, 9300L | 4.4.4.4 | 172.16.0.36 | SVI Vlan99 |
| ASA | Firepower 1010 (internal firewall) | 5.5.5.5 | 172.16.0.42 | Management1/1 |
| PA | PA-440 (edge firewall, only NAT device) | 6.6.6.6 | 172.16.0.41 | MGT |
| SW1 | Access, Staff | n/a | 172.16.0.37 | SVI Vlan99 |
| SW2 | Access, ATM | n/a | 172.16.0.38 | SVI Vlan99 |
| SW3 | Access, HR | n/a | 172.16.0.39 | SVI Vlan99 |
| SW4 | Access, IT | n/a | 172.16.0.40 | SVI Vlan99 |
| Proxmox | Supermicro host | n/a | 172.16.0.43 | bond0 / vmbr3 |
| ISP1 | Lab ISP | n/a | n/a | n/a |

## 2. Physical links (cable list)

State: **up** = carrying traffic, **down** = cabled but admin-down / shut, so draw it dashed.

### WAN, routers, firewalls

| # | End A | End B | Subnet | A IP | B IP | OSPF cost | State |
|---|---|---|---|---|---|---|---|
| 1 | R1 G0/0/0 | ISP1 | 172.17.9.92/30 | .93 | .94 | n/a | up |
| 2 | R1 G0/0/1 | PA eth1/5 (VLAN 10) | 172.16.0.88/29 | .90 | .92 (vlan.10) | 10 | up |
| 3 | R2 G0/0/1 | PA eth1/2 (VLAN 10) | 172.16.0.88/29 | .91 | .92 (vlan.10) | 20 | up |
| 4 | R1 G0/1/1 | R2 G0/1/1 | 172.16.0.108/30 | .109 | .110 | 5 | up |
| 5 | PA eth1/3 | ASA E1/1 (`outside`) | 172.16.0.104/30 | .105 | .106 | 10 | up |
| 6 | R2 G0/0/0 | ASA E1/2 (`wan-backup`) | 172.16.0.112/30 | .113 | .114 | 500 | up (bypass, not used normally) |

PA eth1/2 and eth1/5 are both L2 members of PA VLAN `OUTSIDE`, so R1, R2 and the PA share one /29 (draw as a small segment, or two links into "PA vlan.10 .92").

### Firewalls to distribution switches

All four firewall-to-DSW transit links are routed point-to-point `/30`s (no VLAN). Each link is its own subnet, so there is no overlap and no Layer 2 loop.

| # | End A | End B | Subnet | A IP | B IP | OSPF cost | State |
|---|---|---|---|---|---|---|---|
| 7 | ASA E1/3 (`inside`) | DSW1 Gi1/0/11 | 172.16.0.96/30 | .97 | .98 | 10 | up |
| 8 | ASA E1/4 (`inside2`) | DSW2 Gi1/0/11 | 172.16.0.100/30 | .101 | .102 | 10 | up |
| 9 | PA eth1/6 | DSW1 Gi1/0/12 | 172.16.0.116/30 | .117 | .118 | 500 | up (ASA bypass, not used normally) |
| 10 | PA eth1/7 | DSW2 Gi1/0/12 | 172.16.0.120/30 | .121 | .122 | 500 | up (ASA bypass, not used normally) |

Server gateways (Layer 2, one firewall per server zone):

| # | End A | End B | VLAN | Subnet | IPs | State |
|---|---|---|---|---|---|---|
| 11 | ASA E1/6 (switchport) | DSW1 Gi1/0/14 | 69 | 172.16.0.72/29 | ASA SVI Vlan69 `server` .73 | up |
| 13 | PA eth1/4 | DSW2 Gi1/0/14 | 92 | 172.16.0.80/29 | PA vlan.92 .81 | up |

Removed 5 Nov (unplug, don't draw): #12 PA eth1/8 <-> DSW1 Gi1/0/15 (old PA `.78` in VLAN 69) and #14 ASA E1/7 <-> DSW2 Gi1/0/15 (old ASA `.86` in VLAN 92). Those ports are now unused and shut.

### Distribution and access switches

All trunks: LACP active, native VLAN 821, nonegotiate.

| # | End A | End B | Port-channel | Allowed VLANs |
|---|---|---|---|---|
| 15 | DSW1 Gi1/0/9-10 | DSW2 Gi1/0/9-10 | Po5 | 23, 46, 61, 67, 69, 92, 99, 100 (VLAN 100 = DSW1 .125 <-> DSW2 .126 routed transit, 172.16.0.124/30) |
| 16 | DSW1 Gi1/0/1-2 | SW1 Gi1/0/1-2 | Po1 (both ends) | 23, 99 |
| 17 | DSW2 Gi1/0/1-2 | SW1 Gi1/0/3-4 | Po1 on DSW2 / Po2 on SW1 | 23, 99 |
| 18 | DSW1 Gi1/0/3-4 | SW2 Gi1/0/1-2 | Po2 / Po1 | 46, 99 |
| 19 | DSW2 Gi1/0/3-4 | SW2 Gi1/0/3-4 | Po2 / Po2 | 46, 99 |
| 20 | DSW1 Gi1/0/5-6 | SW3 Gi1/0/1-2 | Po3 / Po1 | 61, 99 |
| 21 | DSW2 Gi1/0/5-6 | SW3 Gi1/0/3-4 | Po3 / Po2 | 61, 99 |
| 22 | DSW1 Gi1/0/7-8 | SW4 Gi1/0/1-2 | Po4 / Po1 | 67, 99 |
| 23 | DSW2 Gi1/0/7-8 | SW4 Gi1/0/3-4 | Po4 / Po2 | 67, 99 |

### Proxmox (Supermicro)

| # | Proxmox port | Bridge | Switch port | VLAN |
|---|---|---|---|---|
| 24 | eno2 (LAN 2) | vmbr1 | DSW1 Gi1/0/13 | 69 internal servers |
| 25 | eno5 (LAN 3) | vmbr2 | DSW2 Gi1/0/13 | 92 DMZ |
| 26 | eno6 (LAN 4), bond0 primary | vmbr3 | DSW1 Gi1/0/24 | 99 OOB |
| 27 | eno1 (LAN 1), bond0 backup | vmbr3 | DSW2 Gi1/0/24 | 99 OOB |

### OOB management cabling (VLAN 99 access ports)

| # | Device port | Switch port |
|---|---|---|
| 28 | R1 G0/1/0 (.33) | DSW1 Gi1/0/22 |
| 29 | PA MGT (.41) | DSW1 Gi1/0/23 |
| 30 | R2 G0/1/0 (.34) | DSW2 Gi1/0/22 |
| 31 | ASA Management1/1 (.42) | DSW2 Gi1/0/23 |
| 32 | Mgmt laptop (.46/28, gw .33) | DSW1 Gi1/0/21 (live only, not in macro) |

Access switches and DSWs carry VLAN 99 in-band over the port-channels (no G0/0 cabling; G0/0 is shut on every switch).

### End-user access ports

| Switch | Ports | VLAN | Port security |
|---|---|---|---|
| SW1 | Gi1/0/5-16 | 23 Staff | max 2 MAC |
| SW2 | Gi1/0/5-7 | 46 ATM | max 1 MAC |
| SW3 | Gi1/0/5-6 | 61 HR | max 2 MAC |
| SW4 | Gi1/0/5-7 | 67 IT | max 2 MAC |

Unused ports go to VLAN 727 (blackhole): SW1 Gi1/0/17-24, SW2/SW4 Gi1/0/8-24, SW3 Gi1/0/7-24, DSW1/DSW2 Gi1/0/15-21 (except DSW1 Gi1/0/21 live). Unused firewall ports: PA eth1/1, eth1/8, ASA E1/5, E1/7, E1/8.

## 3. Addressing table (by subnet)

| VLAN / link | Subnet | Gateway | Assignments |
|---|---|---|---|
| Staff VLAN 23 | 172.16.0.0/27 | VIP .1 | DSW1 .2 (HSRP active), DSW2 .3; DHCP .4-.30 |
| OOB VLAN 99 (VRF OOB) | 172.16.0.32/28 | R1 .33 | R2 .34, DSW1 .35, DSW2 .36, SW1-4 .37-.40, PA .41, ASA .42, Proxmox .43, TACACS1/Jump1 .44, TACACS2/Jump2 .45, mgmt laptop .46 |
| ATM VLAN 46 | 172.16.0.48/29 | VIP .49 | DSW1 .50 (active), DSW2 .51; DHCP .52-.54 |
| HR VLAN 61 | 172.16.0.56/29 | VIP .57 | DSW1 .58, DSW2 .59 (active); .60 reserved; DHCP .61-.62 |
| IT VLAN 67 | 172.16.0.64/29 | VIP .65 | DSW1 .66, DSW2 .67 (active); DHCP .68-.70 |
| Internal servers VLAN 69 | 172.16.0.72/29 | ASA .73 | File VM103 .74, Accounts VM104 .75, Txn VM105 .76, DHCP VM107 .77 (.78 unused) |
| DMZ VLAN 92 | 172.16.0.80/29 | PA .81 | Web VM100 .82, DNS VM101 .83, Jump1 VM102 .84, Jump2 VM106 .85 (.86 unused) |
| Router-PA transit (PA VLAN 10) | 172.16.0.88/29 | n/a | R1 .90, R2 .91, PA .92 |
| ASA-DSW1 | 172.16.0.96/30 | n/a | ASA E1/3 .97, DSW1 Gi1/0/11 .98 |
| ASA-DSW2 | 172.16.0.100/30 | n/a | ASA E1/4 .101, DSW2 Gi1/0/11 .102 |
| PA-ASA | 172.16.0.104/30 | n/a | PA eth1/3 .105, ASA E1/1 .106 |
| R1-R2 | 172.16.0.108/30 | n/a | R1 .109, R2 .110 |
| R2-ASA bypass | 172.16.0.112/30 | n/a | R2 .113, ASA E1/2 .114 |
| PA-DSW1 bypass | 172.16.0.116/30 | n/a | PA eth1/6 .117, DSW1 Gi1/0/12 .118 |
| PA-DSW2 bypass | 172.16.0.120/30 | n/a | PA eth1/7 .121, DSW2 Gi1/0/12 .122 |
| DSW1-DSW2 transit (VLAN 100 on Po5) | 172.16.0.124/30 | n/a | DSW1 Vlan100 .125, DSW2 Vlan100 .126 |
| WAN | 172.17.9.92/30 | ISP1 .94 | R1 .93 |
| Public NAT block (on PA) | 203.149.210.184/29 | n/a | .185 outbound PAT + jump SSH tcp/2222 -> .84:22; .189 Web (.82, 80/443); .190 DNS (.83) |

Other VLANs: 727 blackhole (unused ports), 821 native on all trunks.

HSRP v2 groups use the VLAN number (23, 46, 61, 67). DSW1 priority 110 for 23/46, DSW2 priority 110 for 61/67. Each DSW tracks its own ASA link (`track 10`, decrement 30). Each DSW advertises its Active VLANs into OSPF at cost 10 and the others at cost 20, so the ASA returns traffic to the Active DSW.

## 4. Servers (VMs on Proxmox)

| VM | Role | Bridge | IP | Default gateway |
|---|---|---|---|---|
| 100 | Web (GruppeSechs) | vmbr2 | 172.16.0.82/29 | .81 PA |
| 101 | DNS (BIND) | vmbr2 | 172.16.0.83/29 | .81 PA |
| 102 | TACACS1 / Jump1 | vmbr3 + vmbr2 | .44/28 + .84/29 | .81 (ens19) |
| 103 | File (Samba) | vmbr1 | 172.16.0.74/29 | .73 ASA |
| 104 | Accounts | vmbr1 | 172.16.0.75/29 | .73 ASA |
| 105 | Transactions | vmbr1 | 172.16.0.76/29 | .73 ASA |
| 106 | TACACS2 / Jump2 | vmbr3 + vmbr2 | .45/28 + .85/29 | .81 (ens19) |
| 107 | DHCP | vmbr1 | 172.16.0.77/29 | .73 ASA |

The jump hosts are dual-homed (OOB + DMZ). Draw them touching both networks.

## 5. Firewall zones / nameifs (for labelling)

| Device | Interface | Zone / nameif | Security level |
|---|---|---|---|
| ASA | E1/1 | outside | 0 |
| ASA | E1/2 | wan-backup | 0 |
| ASA | E1/3 | inside | 100 |
| ASA | E1/4 | inside2 | 100 |
| ASA | Vlan69 | server | 60 |
| ASA | Management1/1 | management | 100 |
| PA | vlan.10 (eth1/2, eth1/5) | outside | n/a |
| PA | eth1/3, eth1/6, eth1/7 | inside | n/a |
| PA | vlan.92 (eth1/4) | dmz | n/a |

## 6. Normal traffic path (useful as an overlay)

User -> SW1-4 -> HSRP Active DSW (DSW1 for Staff/ATM, DSW2 for HR/IT) -> that DSW's own /30 to the ASA (E1/3 or E1/4) -> ASA E1/1 .106 -> PA eth1/3 .105 (PAT to .185) -> PA vlan.10 .92 -> R1 G0/0/1 .90 -> R1 G0/0/0 .93 -> ISP1 .94.

- Internal servers use the ASA (.73) only. DMZ servers use the PA (.81) only.
- Standby paths, drawn dashed but **up**: R2 <-> ASA (.112/30), PA eth1/6 and eth1/7 (.116/30, .120/30). Po5 / VLAN 100 is only used when a DSW loses its own ASA link.

### Automatic failover (no config changes)

| Failure | What happens | Lost |
|---|---|---|
| One DSW's ASA link | That DSW's track 10 fails: HSRP moves its VLANs to the other DSW, its static default is withdrawn | Nothing |
| ASA | Both DSWs' track 10 fail; OSPF default moves to the PA bypass (eth1/6/7, cost 500). The PA filters users with its `inside` rules | Internal servers (VLAN 69) |
| PA | The ASA's OSPF default moves to R2 (wan-backup, cost 500); the ASA's standby PAT translates to .185. R1/R2 track 2 (ping .92) fails, so return traffic comes back via R2 -> ASA | DMZ (VLAN 92) |
| One DSW | HSRP moves all VLANs to the other DSW; it still has its own ASA link | Nothing |

## 7. Known mismatches

`plan.md` was updated on 5 Nov to match the macros. These remain:

| Topic | Says | Actual (macros / live) |
|---|---|---|
| Server cabling | `supermicro setup.md`: eno2 -> ASA eth1/6, eno5 -> PA eth1/4 directly | eno2 -> DSW1 Gi1/0/13, eno5 -> DSW2 Gi1/0/13, firewalls hang off the DSWs |
| DSW1 Gi1/0/21 | `dsw1.ttl`: unused (VLAN 727, shut) | live: VLAN 99 mgmt laptop port |

Port numbers come from the macro interface descriptions. Confirm against the real cabling with `show cdp neighbors` (Cisco) and `show lldp neighbors all` (PA) before finalising a diagram.
