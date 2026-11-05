# Punggol Bank Network: Handover Document

**Rack:** 4E (`gruppesechs.sitict.net`)
**Topology:** Full PTSO (Parallel Topologically, Serial Operationally)
**Last updated:** 4 November 2026

This file is the full context for continuing on another device. Chat history from the AI assistant does not transfer between machines, so everything needed is written here. Test results live in `TEST-PLAN.md`.

## Where things stand

- **Network:** end-to-end path works. HR and IT reach the internet through DSW -> ASA -> PA -> R1 -> ISP.
- **Servers:** all built. DHCP (VM107), DNS (VM101), File/Samba (VM103), Accounts portal (VM104), Txn (VM105), Web (VM100), TACACS/jump hosts (VM102/106).
- **Seen working (4 Nov):** DHCP leases (HR, IT, Staff), internet and browsing for HR/IT, internal DNS + other racks' names (e.g. `wkg00n.sitict.net`), HR file share, Staff -> Accounts portal, web server from inside and outside over HTTPS, outside SSH to the jump host (`.185:2222`).
- **Open now:**
  1. ~~Jump host can't reach R1's OOB address~~ **Resolved 4 Nov** (P19). Remaining Stage 7 work: SSH from both jump hosts to every device, check NTP on the remaining devices.
  2. ~~ATM DHCP lease~~ **Working 4 Nov** (`.52`).
  3. **Temporary internet rules for installs are still in place** (8.4). Remove once nothing else needs installing (Wazuh?).
  4. Most deny tests in `TEST-PLAN.md` haven't been run.

---

## 1. Repository layout

```
configs/
├── HANDOVER.md            <- this file
├── TEST-PLAN.md           <- connectivity test matrix with results (Stages 0-10)
├── plan.md                <- original design document (requirements source)
├── supermicro setup.md    <- Proxmox host/bridge/VM notes (see warnings in 8.2)
├── revision 2.md          <- lecture/lab notes (Lab 3.1 jump host, AAA, NTP, DoS)
├── README.md
└── config/                <- Tera Term macros, one per device
    ├── r1.ttl  r2.ttl
    ├── dsw1.ttl  dsw2.ttl
    ├── sw1.ttl  sw2.ttl  sw3.ttl  sw4.ttl
    ├── firepower1010.ttl
    ├── pa440.ttl
    └── pa440_cleanup.ttl
```

The `.ttl` files and `plan.md` contain cleartext credentials. Keep the GitHub repo **private**.

---

## 2. Data flow (normal operation)

```
HR/IT users -> SW3 (HR) / SW4 (IT)
  -> DSW1 / DSW2 (HSRP gateways, inter-VLAN ACLs, DHCP relay to .77)
  -> ASA-1010 inside (.97) -> ASA outside (.106)
  -> PA-440 eth1/3 (.105): App-ID + the single outbound PAT to 203.149.210.185
  -> PA vlan.10 (.92) -> R1 (.90) -> R1 WAN (.93) -> ISP1 (.94) -> Internet

Internal servers (VLAN 69): gateway ASA .73 only (PA .78 backup removed from routing)
DMZ servers (VLAN 92):      gateway PA .81 only  (ASA .86 backup admin-down, manual failover)
OOB management (VLAN 99):   R1 G0/1/0 .33 in VRF OOB; devices only accept SSH from .44/.45
```

- Only the **PA** does outbound NAT. The ASA and R1 don't NAT.
- **Both firewalls must see both directions of a connection.** Every "backup" interface that gave a router a second, directly connected path is shut down, because a connected route always beats OSPF and splits traffic across the two firewalls:
  - PA eth1/6, eth1/7, eth1/8: admin-down (eth1/6/7 also created a VLAN 100 loop).
  - PA vlan.69: removed from the virtual router and OSPF.
  - ASA Vlan92 (`dmz`, `.86`): shut down.
- Consequence: no automatic failover around a failed firewall (an ASA failure is a full outage). Manual failover only.

---

## 3. Addressing

### 3.1 Rack 4E ISP values (ISP appendix formula)
Row `r=4`, column `E=5`, `n=45`, `f = 45 - 16 - 6 = 23`.

| Item | Value |
|------|-------|
| WAN link (R1 <-> ISP1) | `172.17.9.92/30` |
| R1 WAN (G0/0/0) | `172.17.9.93` |
| ISP1 gateway | `172.17.9.94` |
| Public NAT block | `203.149.210.184/29` |

`plan.md` lists R1 as `.94` and the ISP as `.93`. That is **reversed**. The table above is correct.

### 3.2 Internal subnets

| Network | Subnet | Key IPs |
|---------|--------|---------|
| Staff VLAN 23 | 172.16.0.0/27 | VIP .1, DSW1 .2, DSW2 .3; DHCP .4-.30 |
| OOB VLAN 99 | 172.16.0.32/28 | R1 .33, R2 .34, DSW1 .35, DSW2 .36, SW1-4 .37-.40, PA .41, ASA .42, Proxmox .43, TACACS1 .44, TACACS2 .45, mgmt laptop .46 |
| ATM VLAN 46 | 172.16.0.48/29 | VIP .49, DSW1 .50, DSW2 .51; DHCP .52-.54 |
| HR VLAN 61 | 172.16.0.56/29 | VIP .57, DSW1 .58, DSW2 .59; .60 reserved; DHCP .61-.62 |
| IT VLAN 67 | 172.16.0.64/29 | VIP .65, DSW1 .66, DSW2 .67; DHCP .68-.70 |
| Internal servers VLAN 69 | 172.16.0.72/29 | ASA .73, File .74, Accounts .75, Txn .76, DHCP .77, PA .78 (unused) |
| DMZ VLAN 92 | 172.16.0.80/29 | PA .81, Web .82, DNS .83, Jump1 .84, Jump2 .85, ASA .86 (shut) |
| Routers <-> PA (PA vlan.10) | 172.16.0.88/29 | R1 .90, R2 .91, PA .92 |
| Inside transit (VLAN 100) | 172.16.0.96/29 | ASA .97, DSW1 .98, DSW2 .99, PA .100 (bypass, down) |
| PA <-> ASA serial | 172.16.0.104/30 | PA eth1/3 .105, ASA E1/1 .106 |
| R1 <-> R2 direct | 172.16.0.108/30 | R1 .109, R2 .110 |
| R2 <-> ASA bypass | 172.16.0.112/30 | R2 .113, ASA E1/2 .114 |

HSRP: DSW1 Active for VLANs 23/46, DSW2 Active for VLANs 61/67.

### 3.3 Public mappings (decided 4 Nov)

| Service | Public IP | Internal | Where |
|---------|-----------|----------|-------|
| Outbound PAT + jump SSH | 203.149.210.185 (tcp/2222 -> .84:22) | | PA `NAT-OUTBOUND`, `NAT-JUMP-SSH`; ASA standby NAT |
| Web | **203.149.210.189** (tcp/80 + tcp/443) | 172.16.0.82 | PA `PUB-WEB`, `NAT-WEB`, `NAT-WEB-HTTPS`; ASA standby NAT |
| DNS | **203.149.210.190** | 172.16.0.83 | PA `PUB-DNS`, `NAT-DNS`; ASA standby NAT |

The PA originally used `.186`/`.187`. Changed to `.189`/`.190` to match `plan.md` and the ASA standby NAT, so the addresses stay the same after a failover.

**How the name reaches outsiders:** the public `sitict.net` nameservers (`ns1.sitict.net`, 66.96.202.68) don't know `gruppesechs.sitict.net`. The lab DNS (`dns01.ict.siat.edu.sg`; lab Wi-Fi resolver `172.18.18.36`) handles rack subdomains itself and asks our server at `.190`. It caches answers for the record's TTL (the old `.186` answer stayed cached ~1 hour on 4 Nov). Before changing an IP again, lower the zone TTL to 300 a day ahead.

**ICT caching DNS (from the prof):** `172.18.4.32`. VM101 forwards to it first, which is what lets our users resolve other racks' names.

---

## 4. Device access and reload behaviour

- Logins: `bigDonkBonkAdmin` on all devices. Passwords are in `plan.md` and the `.ttl` files. Proxmox and IPMI credentials are in `supermicro setup.md`.
- `write memory` is **not allowed** (lab rule).
  - **PA-440 keeps** its committed config across reloads.
  - **Cisco devices and the ASA wipe** on reload. Rebuild by re-running the device's `.ttl` macro.
- PA rebuild: run `pa440_cleanup.ttl` first, then `pa440.ttl` (PAN-OS `set` is additive; stale objects pile up otherwise).
- Tera Term: the PA macro waits on `[edit]`, not `#`, to avoid command collisions.

### Things a macro can't restore
1. **Po5 cabling**: DSW1 Gi1/0/9-10 <-> DSW2 Gi1/0/9-10. If it's down, DSW2 (HR/IT's active gateway) can't reach the ASA.
2. **PA eth1/6/7/8 and ASA Vlan92 must stay down.** Don't bring them up as a "fix".
3. **R1 default must be `.94`**, never `.93` (R1's own IP).
4. **Proxmox LAN 4 (`eno6`, bond primary) must be cabled to DSW1 Gi1/0/24.** Plugging a laptop into LAN 4 makes the bond send all OOB traffic to the laptop. Use DSW1 Gi1/0/21 (VLAN 99, `.46/28`) for a management laptop instead.

### Lab image quirk: `Mgmt-vrf`
The school image puts some router ports into the factory `Mgmt-vrf`. On R1/R2 G0/1/1 this kept the R1-R2 link out of OSPF. Both macros run `no vrf forwarding Mgmt-vrf` before setting the IP. That command deletes the interface IP, so the IP must be set straight after. Correct IPs: R1 `.109`, R2 `.110`. Check other router ports (e.g. R1 G0/1/0 for the OOB problem) for the same quirk.

### CLI gotchas
- **PA:** `ping host X` uses the management port; use `ping source 172.16.0.92 host X`. Config-mode `show ...` commands need `configure`; in operational mode use `show config running | match <text>`. `test security-policy-match` needs base App-ID names (`dns-base`, not `dns`). Inbound sessions are listed under the **public** destination IP.
- **ASA:** `show running-config object network` (not `... object network <name>`). UDP/53 packet-tracer always ends in `inspect-dns-invalid-pak` (no DNS payload): read the egress interface instead.
- **Windows:** `Test-NetConnection` is slow when the laptop's DNS server isn't answering (reverse lookup first). `nslookup` prints `Server: UnKnown` when the DNS server has no PTR record for itself. That's not an error.

---

## 5. Root causes fixed (all in the macros)

| # | Problem | Fix | File |
|---|---------|-----|------|
| 1 | DAI dropped the static HR host | Static ARP ACL + `ip source binding` (now OPTIONAL, commented out: HR uses DHCP) | `sw3.ttl` |
| 2 | ASA factory NAT `obj_any` PAT'd HR to .106 | `no nat (any,outside) dynamic interface` | `firepower1010.ttl` |
| 3 | ASA OSPF `ip_multicast_ctl` errors on switchport SVIs | OSPF only on inside/outside/wan-backup; `redistribute connected subnets` | `firepower1010.ttl` |
| 4 | ASA command authorization locked us out (TACACS not built) | `aaa authorization command TACACS LOCAL` commented out | `firepower1010.ttl` |
| 5 | DSW default via the PA bypass skipped the ASA | Tracked static default via .97 | `dsw1.ttl`, `dsw2.ttl` |
| 6 | VLAN 100 Layer 2 loop through PA eth1/6/7 | `link-state down` on eth1/6/7/8 | `pa440.ttl` |
| 7 | Po5 physically disconnected | Cable reseated | physical |
| 8 | R1 default pointed at its own IP .93 | Correct `.94 track 1` (live typo only) | `r1.ttl` already correct |
| 9 | PA rulebase stale/misordered; NAT-OUTBOUND incomplete; `DSWs` group stale members | Cleanup macro + full NAT line | `pa440.ttl`, `pa440_cleanup.ttl` |
| 10 | R1-R2 link in `Mgmt-vrf` | `no vrf forwarding Mgmt-vrf` | `r1.ttl`, `r2.ttl` |
| 11 | PA live static default not in macro | `static-route DEFAULT-OUT` | `pa440.ttl` |
| 12 | PA kept `172.16.0.72/29` connected on dead vlan.69 (P9) | vlan.69 removed from VR and OSPF | `pa440.ttl` |
| 13 | ASA only allowed DHCP relay from .98/.99 (P2) | `DSW-SVIS` object-group + bootps permits in `INSIDE_IN` / `SERVER_IN` | `firepower1010.ttl` |
| 14 | Static HR binding blocked the DHCP lease (P12) | Static block removed live; OPTIONAL in macro | `sw3.ttl` |
| 15 | ASA delivered DMZ traffic directly, bypassing the PA (P1) | ASA `interface Vlan92` shut down | `firepower1010.ttl` |
| 16 | DNS server couldn't reach forwarders permanently (P5) | PA `DNS-SVR-FORWARD` + `DNS-SVR` in `NAT-OUTBOUND` | `pa440.ttl`, `pa440_cleanup.ttl` |
| 17 | Staff/ATM got a DNS server they aren't allowed to use | DHCP gives `.83` to HR/IT only | VM107 `dhcpd.conf` |
| 18 | Peer DSW's HSRP hellos dropped by `ACL_STAFF_IN` / `ACL_ATM_IN`; both DSWs Active for 23/46 (P10) | `permit udp <subnet> host 224.0.0.102 eq 1985` first | `dsw1.ttl`, `dsw2.ttl` |
| 19 | DHCP replies for the peer DSW's SVI crossed the user VLAN and were denied by the inbound user ACL (`denied udp 172.16.0.77(67) -> 172.16.0.51(67)`) (P18) | `permit udp host 172.16.0.77 eq bootps any eq bootps` near the top of all four user ACLs | `dsw1.ttl`, `dsw2.ttl` |
| 20 | Inbound HTTPS to the web server had no NAT (P4) | `NAT-WEB-HTTPS` | `pa440.ttl`, `pa440_cleanup.ttl` |
| 21 | PA public IPs (.186/.187) didn't match plan.md / ASA (.189/.190) | `PUB-WEB` .189, `PUB-DNS` .190 | `pa440.ttl` |
| 22 | Users couldn't resolve other racks' names (only on the lab DNS) | VM101 forwards to ICT DNS `172.18.4.32`; PA `ICT-DNS` added to `DNS-SVR-FORWARD` | `pa440.ttl`, `pa440_cleanup.ttl`, VM101 `named.conf.options` |

### Why `R1# ping 8.8.8.8` fails (don't re-diagnose)
R1 sources from `172.17.9.93` on the WAN `/30`, which the ISP doesn't route to the internet. Hosts PAT'd to `203.149.210.185` work. Test internet from an HR/IT host, never from R1.

### Static routes: required vs optional
Every static route in the macros has a `REQUIRED` or `OPTIONAL` comment.

| Route | Status |
|-------|--------|
| DSW1/2 `0.0.0.0/0 via .97 track 10` | OPTIONAL (OSPF provides it; kept to force the ASA path) |
| PA `DEFAULT-OUT 0.0.0.0/0 via .90` | OPTIONAL (OSPF provides it) |
| DSW1/2 `vrf OOB 0.0.0.0/0 via .33` | REQUIRED |
| R1 `0.0.0.0/0 via .94 track 1` | REQUIRED (source of the OSPF default) |
| R1/R2 `203.149.210.184/29 via .92 track 2` | REQUIRED |
| R1 (via .110) / R2 (via .114) AD 250 public-block routes | OPTIONAL (PA-failure backup) |
| ASA `route management 0.0.0.0/0 .33` | REQUIRED |
| PA management `default-gateway .33` | REQUIRED |
| SW1-4 `ip default-gateway .33` | REQUIRED |

---

## 6. Live state vs macros

| Item | Macro | Live | Action |
|------|-------|------|--------|
| PA `HR-IT-INTERNET` | tightened (`web-browsing ssl dns ping`) | loose (`any/any`) | Tighten at Stage 10 |
| PA `TEMP-SVR-UPDATES` / `TEMP-SVR-NAT`, ASA `SERVER_IN` www/https/8.8.8.8 lines | not in macro (on purpose) | **present** | Remove after the last install (8.4) |
| DSW HSRP permit (fix #18) | present | applied on both (seen in `ACL_ATM_IN`, 4 Nov) | None (check `ACL_STAFF_IN` too) |
| DSW DHCP peer-reply permit (fix #19) | present | applied on both (seen in `ACL_ATM_IN`, 4 Nov); ATM lease works | None |
| DSW1 Gi1/0/21 management port (VLAN 99) | **not in macro** | configured live (4 Nov) | Add to `dsw1.ttl` once confirmed working |
| DSW default route | tracked (`ip sla 10`) | plain static via .97 | None (same behaviour) |
| R1 extra `0.0.0.0/0 .94 250` | absent | present (harmless) | None |
| ASA live `INSIDE_IN` | `DSW-SVIS` bootps line only | old `DSWs` bootps line also present (redundant) | Optional: `no access-list INSIDE_IN extended permit udp object-group DSWs object DHCP-SVR eq bootps` |
| Temporary ICMP permits on DSW `ACL_HR_IN` / `ACL_IT_IN` | present, `TODO (FINAL HARDENING)` | present | Remove at Stage 10 |

---

## 7. Testing

Full matrix, results and diagnosis steps are in `TEST-PLAN.md`. Summary (4 Nov):

| Stage | Status |
|-------|--------|
| 0 Infrastructure | All pass; re-check HSRP (0.3) after fix #18 |
| 1 Addressing | HR, IT, Staff pass; ATM pending |
| 2 Internet | HR pass; IT and deny tests not run |
| 3 VLAN isolation | Not run |
| 4 Internal servers | File share, Staff -> Accounts pass; ATM and deny tests pending |
| 5 DMZ | Web + DNS from HR/IT pass (incl. other racks' names); jump/deny tests pending |
| 6 Inbound | HTTPS, DNS, jump SSH pass; HTTP (6.1), lab-resolver name (6.3a), deny tests pending |
| 7 OOB | **Blocked: jump host can't reach R1 .33** |
| 8 Server outbound | Installs + DNS forwarding pass; post-cleanup tests pending |
| 9, 10 | Not started |

### Open problems (numbers match `TEST-PLAN.md` Part 2)

| # | Problem | Next step |
|---|---------|-----------|
| P19 | ~~VM102 couldn't reach R1 `.33`; NTP to `.44` reach 0~~ | **Resolved 4 Nov.** Ping passes on retry (earlier failure was during the LAN 4 / bond disturbance). NTP to `.44`: chronyd hadn't loaded the edited config (not listening on udp/123); fixed by restarting chrony and resetting R1's association. R1 now reaches `.44` at stratum 4. |
| P3 | Txn server can't reach DNS `.83` (`SERVER_IN` line 1 denies first) | Decide if Txn needs names; if so add a permit above line 1 |
| P6 | R1 `WAN-IN` SSH/Telnet denies reference `.94` | Change to `.93` in `r1.ttl` and live |
| P11 | DHCP renewals from client IPs denied by ASA `INSIDE_IN` | Optional: permit udp from user subnets to `DHCP-SVR eq bootps` |
| P13 | TACACS lockout risk once tac_plus answers | Console session open; `test aaa group tac-group <user> <pw> legacy` before relying on it |
| P14 | ~~Clocks not synced until chrony runs on `.44/.45`~~ | **Fixed 4 Nov** for R1/R2 (chrony on both TACACS VMs). Confirm on DSWs, SWs, ASA (`show ntp status`) and PA (`show ntp`) |
| P15 | Internal servers have no DNS after the temporary rules go | Decide if needed |
| P17 | Wazuh agents on internal servers can't reach `.84/.85` | ASA `SERVER_IN` + PA `inside -> dmz` permits for tcp/1514-1515 |

---

## 8. Servers (Proxmox)

### 8.1 Proxmox host

| Bridge | Port | Network | Host IP |
|--------|------|---------|---------|
| vmbr1 | eno2 (LAN 2) | Internal servers VLAN 69 | none |
| vmbr2 | eno5 (LAN 3) | DMZ VLAN 92 | none |
| vmbr3 | bond0 = eno6 (LAN 4, primary, DSW1 Gi1/0/24) + eno1 (LAN 1, backup, DSW2 Gi1/0/24), active-backup | OOB VLAN 99 | 172.16.0.43/28, gw .33 |

Proxmox GUI: `https://172.16.0.43:8006` from the OOB network, e.g. a laptop on DSW1 Gi1/0/21 with `172.16.0.46/28`, gateway `.33`. Don't use LAN 4 for a laptop (see section 4).

### 8.2 Warnings about `supermicro setup.md`
- Its netplan blocks contain stray backslashes (`\-`, `\#`, `\[`, `\]`) from a document export -> `unknown key \network`. Use the templates in 8.3.
- The VM102 block has an invalid extra line `via 172.16.0.86`.
- The backup gateways `.78` and `.86` don't work (interfaces down/removed) and Linux doesn't fail over between metric routes. Use one gateway.
- "Linked Clone" needs a template. Use Full Clone.
- Its cabling description differs from `plan.md`. Either works if the VLAN joins the ports.

### 8.3 VM inventory and network

| VM | Role | Bridge(s) | IP | Default route | Nameservers |
|----|------|-----------|----|---------------|-------------|
| 100 | Web, hostname GruppeSechs | vmbr2 | 172.16.0.82/29 | .81 (PA) | 172.16.0.83, 8.8.8.8 |
| 101 | DNS (BIND) | vmbr2 | 172.16.0.83/29 | .81 (PA) | 127.0.0.1, 8.8.8.8 |
| 102 | TACACS1 / Jump1 | ens18 vmbr3, ens19 vmbr2 | .44/28, .84/29 | .81 on ens19 **only** | 172.16.0.83, 8.8.8.8 |
| 103 | File (Samba) | vmbr1 | 172.16.0.74/29 | .73 (ASA) | 172.16.0.83, 8.8.8.8 |
| 104 | Accounts (Nginx portal) | vmbr1 | 172.16.0.75/29 | .73 | same |
| 105 | Transactions | vmbr1 | 172.16.0.76/29 | .73 | same |
| 106 | TACACS2 / Jump2 | ens18 vmbr3, ens19 vmbr2 | .45/28, .85/29 | .81 on ens19 **only** | 172.16.0.83, 8.8.8.8 |
| 107 | DHCP | vmbr1 | 172.16.0.77/29 | .73 | same |

Check with `ip route`: exactly one `default via` line per VM.

**Internal server template** (`/etc/netplan/50-cloud-init.yaml`, change the IP per VM):
```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 172.16.0.77/29
      routes:
        - to: default
          via: 172.16.0.73
      nameservers:
        addresses: [172.16.0.83, 8.8.8.8]
```

**DMZ server template** (Web; DNS uses `[127.0.0.1, 8.8.8.8]`):
```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 172.16.0.82/29
      routes:
        - to: default
          via: 172.16.0.81
      nameservers:
        addresses: [172.16.0.83, 8.8.8.8]
```

**Dual-NIC jump/TACACS template** (VM102; VM106 uses .45 / .85):
```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 172.16.0.44/28
    ens19:
      dhcp4: false
      addresses:
        - 172.16.0.84/29
      routes:
        - to: default
          via: 172.16.0.81
      nameservers:
        addresses: [172.16.0.83, 8.8.8.8]
```

Apply: `sudo chmod 600 /etc/netplan/50-cloud-init.yaml && sudo netplan generate && sudo netplan apply`, then `ip -br a`, `ip route`, `ping -c 3 <gateway>`.

**Cloning** (internal servers were cloned from VM103 before Samba was installed): shut down the source -> Clone -> **Full Clone**. Before first boot, tick **Disconnect** on net0 (the clone still has the source IP). In the clone: fix the netplan IP, `sudo hostnamectl set-hostname <name>`, edit `/etc/hosts`, `sudo truncate -s 0 /etc/machine-id && sudo systemd-machine-id-setup`, `sudo rm /etc/ssh/ssh_host_* && sudo ssh-keygen -A && sudo systemctl restart ssh`. Untick Disconnect.

### 8.4 Temporary internet rules for package installs (STILL LIVE)

No permanent rule lets internal or DMZ servers reach the internet (only the Txn server inside its window, and the jump hosts for NTP/DNS). These **temporary** rules allow `apt install`. They're deliberately not in the macros.

Paths: internal server -> ASA `server` -> ASA `outside` -> PA eth1/3 (zone `inside`) -> outside. DMZ server -> PA vlan.92 (zone `dmz`) -> outside.

**Added (for reference):**
```
(ASA)
access-list SERVER_IN line 1 extended permit tcp object SERVERS any eq www
access-list SERVER_IN line 2 extended permit tcp object SERVERS any eq https
access-list SERVER_IN line 3 extended permit udp object SERVERS host 8.8.8.8 eq domain

(PA, configure mode)
set rulebase security rules TEMP-SVR-UPDATES from [ inside dmz ] to outside source [ SERVERS DMZ-SUBNET ] destination any source-user any category any application any service any action allow log-end yes
move rulebase security rules TEMP-SVR-UPDATES before DENY-ALL
set rulebase nat rules TEMP-SVR-NAT from [ inside dmz ] to outside source [ SERVERS DMZ-SUBNET ] destination any service any source-translation dynamic-ip-and-port translated-address PUB-PAT
commit
```

**Remove when installs are done** (install Wazuh first if you're using it):
```
(ASA)  no access-list SERVER_IN extended permit tcp object SERVERS any eq www
       no access-list SERVER_IN extended permit tcp object SERVERS any eq https
       no access-list SERVER_IN extended permit udp object SERVERS host 8.8.8.8 eq domain
(PA)   delete rulebase security rules TEMP-SVR-UPDATES
       delete rulebase nat rules TEMP-SVR-NAT
       commit
```
Then run `TEST-PLAN.md` 8.5 and 8.9 (DNS must still resolve through `DNS-SVR-FORWARD`).

### 8.5 DHCP server (VM107, 172.16.0.77) - built

- The DSWs relay with `ip helper-address 172.16.0.77` on VLANs 23, 46, 61, 67. **Both** DSWs relay every request, with `giaddr` = the receiving SVI (.2/.3, .50/.51, .58/.59, .66/.67). The server answers both.
- The ASA sends replies for a subnet to one DSW only, so replies meant for the other DSW's SVI cross the user VLAN. That's why every user ACL needs `permit udp host 172.16.0.77 eq bootps any eq bootps` (fix #19), and the ASA needs the `DSW-SVIS` permits (fix #13).
- DNS server `.83` is handed out to HR and IT only.

| VLAN | Subnet | Router | Range |
|------|--------|--------|-------|
| 23 Staff | 172.16.0.0/27 | .1 | .4 - .30 |
| 46 ATM | 172.16.0.48/29 | .49 | .52 - .54 |
| 61 HR | 172.16.0.56/29 | .57 | .61 - .62 (.60 reserved) |
| 67 IT | 172.16.0.64/29 | .65 | .68 - .70 |

`/etc/default/isc-dhcp-server`: `INTERFACESv4="ens18"`.

`/etc/dhcp/dhcpd.conf`:
```
authoritative;
default-lease-time 86400;
max-lease-time 172800;
option domain-name "gruppesechs.sitict.net";
# No global DNS server: only HR and IT may use DNS (plan.md).

# Server's own subnet. Required so dhcpd listens on ens18. No pool.
subnet 172.16.0.72 netmask 255.255.255.248 {
}

subnet 172.16.0.0 netmask 255.255.255.224 {
  range 172.16.0.4 172.16.0.30;
  option routers 172.16.0.1;
  option broadcast-address 172.16.0.31;
}

subnet 172.16.0.48 netmask 255.255.255.248 {
  range 172.16.0.52 172.16.0.54;
  option routers 172.16.0.49;
  option broadcast-address 172.16.0.55;
}

subnet 172.16.0.56 netmask 255.255.255.248 {
  range 172.16.0.61 172.16.0.62;
  option routers 172.16.0.57;
  option broadcast-address 172.16.0.63;
  option domain-name-servers 172.16.0.83;
}

subnet 172.16.0.64 netmask 255.255.255.248 {
  range 172.16.0.68 172.16.0.70;
  option routers 172.16.0.65;
  option broadcast-address 172.16.0.71;
  option domain-name-servers 172.16.0.83;
}
```
Check/restart: `sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf && sudo systemctl restart isc-dhcp-server`. Watch: `sudo journalctl -u isc-dhcp-server -f` or `sudo tcpdump -ni ens18 port 67 or port 68`.

If a laptop gets `169.254.x.x`: the DAI messages for that address are a side effect. Find where the exchange stops: tcpdump on VM107 (request arriving? offer sent?), DSW logs (`ACL_*_IN denied udp 172.16.0.77(67)` = fix #19 missing), ASA `show logging | include 172.16.0.77`.

### 8.6 Service requirements summary

| VM | Software | Requirement (plan.md) | Rules involved |
|----|----------|------------------------|----------------|
| 103 File | Samba | HR only, tcp/445 | DSW `ACL_HR_IN`, ASA `permit tcp HR FILE-SVR eq 445` |
| 104 Accounts | Nginx HTTPS portal | Staff + ATM only | DSW `permit ip`, ASA `permit ip STAFF-ATM ACCT-SVR` |
| 105 Txn | (portal, same pattern) | Staff + ATM; internet 80/443 only in TXN-WINDOW (weekdays 01:00-03:00, final time on demo day) | ASA `SERVER_IN` time-range, PA `TXN-INTERNET-TIMED` (P3, P14) |
| 100 Web | Nginx, hostname GruppeSechs | HR/IT internal; outside via `203.149.210.189` | PA `INTERNAL-TO-WEB`, `INBOUND-WEB`, `NAT-WEB`, `NAT-WEB-HTTPS` |
| 101 DNS | BIND9 | Authoritative for `gruppesechs.sitict.net`; recursive for HR/IT via the ICT DNS | PA `INTERNAL-TO-DNS`, `INBOUND-DNS`, `NAT-DNS`, `DNS-SVR-FORWARD` |
| 102/106 TACACS + jump | OpenSSH, tac_plus, chrony, NetFlow analyzer, Wazuh | Devices already point TACACS/NTP/syslog/NetFlow at .44/.45 | PA `JUMP-SSH-IN`, `IT-TO-JUMP-SSH`, `DMZ-OUTBOUND-NTP` |

### 8.7 File server (VM103, 172.16.0.74, Samba) - built

HR only, SMB over **tcp/445 only** (the ASA doesn't permit NetBIOS 137-139). Users: `hrmanager`, `hradmin`.

```bash
sudo apt install -y samba
sudo groupadd hrshare
sudo useradd -M -s /usr/sbin/nologin -G hrshare hrmanager
sudo useradd -M -s /usr/sbin/nologin -G hrshare hradmin
sudo smbpasswd -a hrmanager
sudo smbpasswd -a hradmin
sudo mkdir -p /srv/samba/hr
sudo chown root:hrshare /srv/samba/hr
sudo chmod 2770 /srv/samba/hr
```

`/etc/samba/smb.conf` (in `[global]`; `[printers]`/`[print$]` removed):
```
[global]
   workgroup = WORKGROUP
   server string = Punggol Bank File Server
   server role = standalone server
   security = user
   map to guest = never
   smb ports = 445
   disable netbios = yes
   server min protocol = SMB2_10
   server smb encrypt = desired
   hosts allow = 172.16.0.56/29 127.0.0.1
   hosts deny = 0.0.0.0/0
   interfaces = ens18 lo
   bind interfaces only = yes
   log file = /var/log/samba/log.%m
   max log size = 1000

[HR]
   path = /srv/samba/hr
   valid users = @hrshare
   read only = no
   browseable = yes
   create mask = 0660
   directory mask = 2770
```
```bash
testparm -s
sudo systemctl restart smbd
sudo systemctl disable --now nmbd
smbclient //127.0.0.1/HR -U hrmanager -c 'ls'
```
Client: `net use H: \\172.16.0.74\HR /user:hrmanager`. Mapping creates nothing: save a file, then check `sudo ls -la /srv/samba/hr` (needs sudo: the folder is 2770). Don't publish a `fileserver` record in the public zone (it's answered on the internet).

### 8.8 Accounts server (VM104, 172.16.0.75) - built

`plan.md` doesn't specify the software; chosen: Nginx HTTPS portal, self-signed certificate, HTTP basic auth. Staff/ATM use the IP (they get no DNS server).

```bash
sudo apt install -y nginx apache2-utils openssl
sudo mkdir -p /var/www/accounts
echo '<h1>Punggol Bank - Accounts Server</h1><p>Staff and ATM access only.</p>' | sudo tee /var/www/accounts/index.html
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/accounts.key -out /etc/ssl/certs/accounts.crt -subj "/CN=172.16.0.75" -addext "subjectAltName=IP:172.16.0.75"
sudo htpasswd -c /etc/nginx/.htpasswd-accounts staff1
sudo htpasswd /etc/nginx/.htpasswd-accounts atm1
```

`/etc/nginx/sites-available/accounts`:
```
server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name _;
    ssl_certificate     /etc/ssl/certs/accounts.crt;
    ssl_certificate_key /etc/ssl/private/accounts.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    allow 172.16.0.0/27;     # Staff
    allow 172.16.0.48/29;    # ATM
    allow 127.0.0.1;
    deny all;

    auth_basic "Punggol Bank Accounts";
    auth_basic_user_file /etc/nginx/.htpasswd-accounts;

    root /var/www/accounts;
    index index.html;
}
```
```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/accounts /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
curl -k -u staff1 https://127.0.0.1
```
`403` = client IP not in `allow`; `401` = wrong login. The Txn server (VM105) uses the same pattern with its own IP and names.

### 8.9 Web server (VM100, 172.16.0.82, hostname GruppeSechs) - built

Serves HTTP and HTTPS on both ports, no redirect. Reachable from HR/IT (`172.16.0.82`) and outside (`203.149.210.189`, `gruppesechs.sitict.net`).

**Getting files onto VM100:** download (`curl -L -o index.html <raw URL>`, needs 8.4), or through a jump host in one step from an outside PC:
```
scp -o ProxyJump=<jumpuser>@203.149.210.185:2222 index.html <vm100user>@172.16.0.82:/tmp/
```
(Two-step version: `scp -P 2222 index.html <jumpuser>@203.149.210.185:~/` from the PC, then `scp ~/index.html <vm100user>@172.16.0.82:/tmp/` **on the jump host**.) Use `-r` for a folder.

```bash
sudo hostnamectl set-hostname GruppeSechs        # also fix /etc/hosts
sudo apt install -y nginx openssl unzip
sudo systemctl disable --now apache2             # Apache was installed and held port 80 on 4 Nov
sudo mkdir -p /var/www/gruppesechs
sudo cp /tmp/index.html /var/www/gruppesechs/
sudo chmod -R a+rX /var/www/gruppesechs
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/gruppesechs.key -out /etc/ssl/certs/gruppesechs.crt -subj "/CN=gruppesechs.sitict.net" -addext "subjectAltName=DNS:gruppesechs.sitict.net,DNS:www.gruppesechs.sitict.net,IP:172.16.0.82,IP:203.149.210.189"
```
Paste `openssl` as **one line**: the multi-line `\` version broke in the console on 4 Nov and left no certificate.

`/etc/nginx/sites-available/gruppesechs`:
```
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    server_name _;

    ssl_certificate     /etc/ssl/certs/gruppesechs.crt;
    ssl_certificate_key /etc/ssl/private/gruppesechs.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    root /var/www/gruppesechs;
    index index.html;
    server_tokens off;
}
```
```bash
sudo rm -f /etc/nginx/sites-enabled/default      # only 'gruppesechs' may claim default_server
sudo ln -s /etc/nginx/sites-available/gruppesechs /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl enable --now nginx
curl -s http://127.0.0.1 | head
```
Troubleshooting seen on 4 Nov: `duplicate default server` (another site still enabled), `cannot load certificate` (certificate never created), `bind() to 0.0.0.0:80 failed (98: Address already in use)` (Apache; find with `sudo ss -tlnp | grep ':80 '`). If nginx failed, use `start`, not `reload`.

### 8.10 DNS server (VM101, 172.16.0.83, BIND) - built

- **Authoritative** for `gruppesechs.sitict.net`. Zone file path: `grep -A3 'zone "gruppesechs.sitict.net"' /etc/bind/named.conf.local`. A record -> `203.149.210.189`. Bump the SOA **serial** after every edit, then `sudo named-checkzone gruppesechs.sitict.net <zonefile> && sudo systemctl reload named`.
- **Recursive** for HR and IT only. `/etc/bind/named.conf.options`, inside `options { }`:
  ```
  listen-on { any; };
  allow-query { any; };
  allow-recursion { localhost; localnets; 172.16.0.56/29; 172.16.0.64/29; };
  forwarders { 172.18.4.32; 8.8.8.8; };
  forward first;
  ```
  Don't set `allow-recursion { any; }`: `.83` is published on the internet (`.190`), so it would become an open resolver.
- Checks: `dig @127.0.0.1 google.com +short`, `dig @127.0.0.1 wkg00n.sitict.net +short`, `dig @127.0.0.1 gruppesechs.sitict.net +short` (`.189`).
- If HR's queries time out: confirm ASA Vlan92 is still shut (`show route 172.16.0.80` -> via `.105`), and `tcpdump -ni ens18 port 53` on VM101.
- Optional: a BIND internal view returning `172.16.0.82` for inside users. Today inside users get `.189`, which also works (the traffic goes out to R1 and back in through the PA's inbound NAT).

### 8.11 Still to set up on VM102/106

| Item | Notes |
|------|-------|
| **chrony** (NTP for the OOB devices) | **DONE 4 Nov (VM102 checked; confirm VM106 config matches).** Check it's serving with `sudo ss -ulpn \| grep chronyd` (must show `0.0.0.0:123`), and `chronyc -N sources`. After editing the config, restart chrony and confirm port 123 is open; on a Cisco device, reset a stuck association with `no ntp server vrf OOB 172.16.0.44` then `ntp server vrf OOB 172.16.0.44 prefer`. Devices picking `.45` over `.44` is fine when VM102 syncs from VM106 (one stratum lower); add `noselect` to each VM's `server <other TACACS>` line to make both prefer the internet pool. Setup: **on both VM102 and VM106** (devices use `.44` prefer + `.45`). In `/etc/chrony/chrony.conf`: comment out the Ubuntu `pool ...` lines, add `pool 0.sg.pool.ntp.org iburst maxsources 4`, `server 172.16.0.45 iburst` (on VM106: `.44`), `allow 172.16.0.32/28`, `local stratum 10 orphan`; `sudo systemctl restart chrony`. Upstream NTP leaves via ens19 (PA `DMZ-OUTBOUND-NTP`, NAT `NAT-OUTBOUND`); devices query over ens18 (OOB). Tests: `TEST-PLAN.md` 7.13-7.16. Fixes P14. |
| **tac_plus** | Config in `plan.md` ("TACACS config"). **Read P13 first.** Keep a console session open on each device, then test with `test aaa group tac-group bigDonkBonkAdmin <pw> legacy`. Re-enable ASA `aaa authorization command TACACS LOCAL` only after it works. |
| **Wazuh** | Manager needs roughly 4 GB RAM. Install while the 8.4 temporary rules are up. Agents on internal servers need P17 rules (ASA `SERVER_IN` + PA `inside -> dmz`, tcp/1514-1515). |
| **NetFlow analyzer** | Devices already export to `.44`/`.45` udp/2055 over the OOB VRF. |

---

## 9. Next steps (in order)

1. ~~Fix the jump host -> R1 OOB problem (P19)~~ Resolved. **Finish Stage 7**: NTP works on R1 (check the other devices with `show ntp associations`), then SSH to every device from both jump hosts.
2. ~~Confirm fixes #18 and #19 live on both DSWs~~ **Confirmed 4 Nov**; ATM got `.52`. Still to do: ATM -> Accounts (`TEST-PLAN.md` 4.9) and re-check HSRP (0.3).
3. **Add DSW1 Gi1/0/21** (management port) to `dsw1.ttl` once confirmed.
4. **Set up chrony, then tac_plus (carefully), then Wazuh**, while the temporary rules are still up.
5. **Remove the temporary rules** (8.4) and run `TEST-PLAN.md` 8.5 and 8.9.
6. **Run the remaining tests** (Stages 2-6 deny tests, Stage 8 Txn window).
7. **Fix P6** (R1 `WAN-IN` `.94` -> `.93`) and decide on P3, P11, P15.
8. **Final hardening (Stage 10):** remove the temporary ICMP permits on both DSWs, tighten PA `HR-IT-INTERNET`, re-enable ASA command authorization (after TACACS works), re-test.
9. **Demo day:** set the final TXN-WINDOW (ASA `time-range`, PA `schedule`).
