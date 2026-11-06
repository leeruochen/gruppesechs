# Punggol Bank Network: Handover Document

**Rack:** 4E (`gruppesechs.sitict.net`)
**Topology:** Full PTSO (Parallel Topologically, Serial Operationally)
**Last updated:** 5 November 2026 (firewall-link redesign, second ISP and NAT on the routers are in the macros, not yet applied live: see section 10)

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
  -> HSRP Active DSW (DSW1: Staff/ATM, DSW2: HR/IT; ACLs, DHCP relay to .77)
  -> that DSW's own /30 to the ASA (DSW1 -> E1/3 .97, DSW2 -> E1/4 .101)
  -> ASA outside (.106)
  -> PA-440 eth1/3 (.105): App-ID / security policy (no NAT)
  -> PA vlan.10 (.92) -> R1 (.90): PAT to 203.149.210.185 -> R1 WAN (.93) -> ISP1 (.94) -> Internet
  (R1 or ISP1 down: PA -> R2 (.91): PAT to 129.126.142.185 -> R2 WAN (172.17.10.93) -> ISP2 (.94))

Internal servers (VLAN 69): gateway ASA .73 only (no backup gateway)
DMZ servers (VLAN 92):      gateway PA .81 only  (no backup gateway)
OOB management (VLAN 99):   R1 G0/1/0 .33 in VRF OOB; devices only accept SSH from .44/.45
```

- **NAT is on the routers** (5 Nov). Each router translates to its own ISP's block: R1 `203.149.210.184/29`, R2 `129.126.142.184/29`. The PA and ASA don't NAT; they filter real addresses in both directions. ISP1 is primary (R1 advertises its default at OSPF metric 10, R2 at 50).
- **Both firewalls must see both directions of a connection.** A connected route always beats OSPF, so:
  - A backup link on its **own transit subnet** is safe: OSPF cost decides when it's used. These stay **up** and fail over automatically: ASA E1/4 to DSW2, PA eth1/6/7 to DSW1/DSW2 (cost 500), ASA E1/5 to R1 and ASA E1/2 to R2 (cost 500, used only when the PA is down).
  - A backup **gateway inside a server subnet** is not safe: it gives the backup firewall a connected route that skips the primary (P1, P9). The ASA and PA also can't share a gateway IP, and Linux won't switch gateways on its own. So these were **removed** (5 Nov): PA vlan.69 `.78` (eth1/8) and ASA Vlan92 `.86` (E1/7).
- Each server zone belongs to one firewall: internal servers to the ASA, DMZ to the PA. If that firewall fails, its zone is down by design. User internet survives either firewall failing (`TOPOLOGY.md` section 6 has the failover table).

---

## 3. Addressing

### 3.1 Rack 4E ISP values (ISP appendix formula)
Row `r=4`, column `E=5`, `n=45`, `f = 45 - 16 - 6 = 23`.

| Item | Value |
|------|-------|
| WAN link (R1 <-> ISP1) | `172.17.9.92/30` |
| R1 WAN (G0/0/0) | `172.17.9.93` |
| ISP1 gateway | `172.17.9.94` |
| Public NAT block | `203.149.210.184/29` (NAT on R1) |
| WAN link (R2 <-> ISP2, added 5 Nov) | `172.17.10.92/30`: R2 G0/0/0 `.93`, ISP2 gateway `.94` |
| ISP2 public block | `129.126.142.184/29` (NAT on R2) |

`plan.md` lists R1 as `.94` and the ISP as `.93`. That is **reversed**. The table above is correct.

### 3.2 Internal subnets

| Network | Subnet | Key IPs |
|---------|--------|---------|
| Staff VLAN 23 | 172.16.0.0/27 | VIP .1, DSW1 .2, DSW2 .3; DHCP .4-.30 |
| OOB VLAN 99 | 172.16.0.32/28 | R1 .33, R2 .34, DSW1 .35, DSW2 .36, SW1-4 .37-.40, PA .41, ASA .42, Proxmox .43, TACACS1 .44, TACACS2 .45, mgmt laptop .46 |
| ATM VLAN 46 | 172.16.0.48/29 | VIP .49, DSW1 .50, DSW2 .51; DHCP .52-.54 |
| HR VLAN 61 | 172.16.0.56/29 | VIP .57, DSW1 .58, DSW2 .59; .60 reserved; DHCP .61-.62 |
| IT VLAN 67 | 172.16.0.64/29 | VIP .65, DSW1 .66, DSW2 .67; DHCP .68-.70 |
| Internal servers VLAN 69 | 172.16.0.72/29 | ASA .73, File .74, Accounts .75, Txn .76, DHCP .77 (.78 unused) |
| DMZ VLAN 92 | 172.16.0.80/29 | PA .81, Web .82, DNS .83, Jump1 .84, Jump2 .85 (.86 unused) |
| Routers <-> PA (PA vlan.10) | 172.16.0.88/29 | R1 .90, R2 .91, PA .92 |
| ASA <-> DSW1 | 172.16.0.96/30 | ASA E1/3 `inside` .97, DSW1 Gi1/0/11 .98 |
| ASA <-> DSW2 | 172.16.0.100/30 | ASA E1/4 `inside2` .101, DSW2 Gi1/0/11 .102 |
| PA <-> ASA serial | 172.16.0.104/30 | PA eth1/3 .105, ASA E1/1 .106 |
| R1 <-> ASA bypass | 172.16.0.108/30 | R1 G0/1/1 .109, ASA E1/5 `wan-backup2` .110 (was R1 <-> R2 until 5 Nov) |
| R2 <-> ASA bypass | 172.16.0.112/30 | R2 G0/1/1 .113, ASA E1/2 `wan-backup` .114 (R2 end moved from G0/0/0) |
| PA <-> DSW1 bypass | 172.16.0.116/30 | PA eth1/6 .117, DSW1 Gi1/0/12 .118 |
| PA <-> DSW2 bypass | 172.16.0.120/30 | PA eth1/7 .121, DSW2 Gi1/0/12 .122 |
| DSW1 <-> DSW2 (VLAN 100 on Po5) | 172.16.0.124/30 | DSW1 .125, DSW2 .126 |

Before 5 Nov, VLAN 100 was a shared `/29` (ASA .97, DSW1 .98, DSW2 .99, PA .100). Two routed ports can't share a subnet, which is why ASA E1/4 had to stay shut. One `/30` per link removes that limit.

HSRP: DSW1 Active for VLANs 23/46, DSW2 Active for VLANs 61/67. Each DSW tracks its own ASA link (`track 10` = IP SLA ping to `.97` / `.101`, decrement 30). Each DSW advertises its Active VLANs at OSPF cost 10 and the others at 20, so the ASA returns traffic to the Active DSW.

### 3.3 Public mappings (NAT on R1/R2 since 5 Nov)

| Service | ISP1 (R1) | ISP2 (R2, backup) | Internal |
|---------|-----------|-------------------|----------|
| Outbound PAT (`NAT-SOURCES`: HR, IT, Txn, DNS, jump hosts) | 203.149.210.185 | 129.126.142.185 | |
| Jump SSH | 203.149.210.185 tcp/2222 | 129.126.142.185 tcp/2222 | 172.16.0.84:22 |
| Web | **203.149.210.189** tcp/80 + 443 | 129.126.142.189 | 172.16.0.82 |
| DNS | **203.149.210.190** udp/tcp 53 | 129.126.142.190 | 172.16.0.83 |

Check on a router with `show ip nat translations` and `show ip nat statistics`. The lab DNS only knows the ISP1 addresses, so with R1 down outsiders reach the servers by the ISP2 IPs only. Inside users can't use the public IPs at all (IOS doesn't hairpin NAT), which is why VM101 needs the internal view (8.10).

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
1. **Po5 cabling**: DSW1 Gi1/0/9-10 <-> DSW2 Gi1/0/9-10. It carries the user VLANs (HSRP hellos) and the DSW1-DSW2 transit (VLAN 100). Since 5 Nov each DSW has its own ASA link, so Po5 is no longer in the HR/IT path, but it's still needed.
2. **Cabling after the 5 Nov redesign:** ASA E1/4 <-> DSW2 Gi1/0/11 must be connected. Proxmox eno2 (LAN 2) must go **straight into ASA E1/6**, and eno5 (LAN 3) **straight into PA eth1/4** (no DSW in between). DSW Gi1/0/13-15 on both switches are unused: unplug them, including PA eth1/8 and ASA E1/7. PA eth1/6/7 stay **up** now; they're routed `/30`s, so they can't loop.
3. **R1 default must be `.94`**, never `.93` (R1's own IP).
4. **Proxmox LAN 4 (`eno6`, bond primary) must be cabled to DSW1 Gi1/0/24.** Plugging a laptop into LAN 4 makes the bond send all OOB traffic to the laptop. Use DSW1 Gi1/0/21 (VLAN 99, `.46/28`) for a management laptop instead.

### Lab image quirk: `Mgmt-vrf`
The school image puts some router ports into the factory `Mgmt-vrf`. On R1/R2 G0/1/1 this kept the old R1-R2 link out of OSPF (G0/1/1 is now each router's ASA link). Both macros run `no vrf forwarding Mgmt-vrf` before setting the IP. That command deletes the interface IP, so the IP must be set straight after. Correct IPs: R1 `.109`, R2 `.113`. Check other router ports (e.g. R1 G0/1/0 for the OOB problem) for the same quirk.

### CLI gotchas
- **PA:** `ping host X` uses the management port; use `ping source 172.16.0.92 host X`. Config-mode `show ...` commands need `configure`; in operational mode use `show config running | match <text>`. `test security-policy-match` needs base App-ID names (`dns-base`, not `dns`). Since 5 Nov the PA does no NAT, so inbound sessions show the real server IP (`.82`, `.83`, `.84`).
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
| 16 | DNS server couldn't reach forwarders permanently (P5) | PA `DNS-SVR-FORWARD` + `DNS-SVR` in `NAT-OUTBOUND` (since 5 Nov: router `NAT-SOURCES`) | `pa440.ttl`, `pa440_cleanup.ttl` |
| 17 | Staff/ATM got a DNS server they aren't allowed to use | DHCP gives `.83` to HR/IT only | VM107 `dhcpd.conf` |
| 18 | Peer DSW's HSRP hellos dropped by `ACL_STAFF_IN` / `ACL_ATM_IN`; both DSWs Active for 23/46 (P10) | `permit udp <subnet> host 224.0.0.102 eq 1985` first | `dsw1.ttl`, `dsw2.ttl` |
| 19 | DHCP replies for the peer DSW's SVI crossed the user VLAN and were denied by the inbound user ACL (`denied udp 172.16.0.77(67) -> 172.16.0.51(67)`) (P18) | `permit udp host 172.16.0.77 eq bootps any eq bootps` near the top of all four user ACLs | `dsw1.ttl`, `dsw2.ttl` |
| 20 | Inbound HTTPS to the web server had no NAT (P4) | `NAT-WEB-HTTPS` | `pa440.ttl`, `pa440_cleanup.ttl` |
| 21 | PA public IPs (.186/.187) didn't match plan.md / ASA (.189/.190) | `PUB-WEB` .189, `PUB-DNS` .190 | `pa440.ttl` |
| 22 | Users couldn't resolve other racks' names (only on the lab DNS) | VM101 forwards to ICT DNS `172.18.4.32`; PA `ICT-DNS` added to `DNS-SVR-FORWARD` | `pa440.ttl`, `pa440_cleanup.ttl`, VM101 `named.conf.options` |
| 23 | Backup links needed manual `no shutdown` (not allowed in the demo); ASA failure was a full outage (P8) | 5 Nov redesign: firewall-to-DSW links are routed `/30`s (ASA E1/3 + E1/4, PA eth1/6 + eth1/7), all up; server-subnet backups (PA `.78`, ASA `.86`) removed; HSRP and default route track each DSW's own ASA link. **Not applied live yet** (section 10) | all firewall and DSW macros |
| 25 | Single ISP; failover depended on a PA-side static default | Second ISP on R2 (`172.17.10.93/30`, block `129.126.142.184/29`); R1-R2 link replaced by R1 <-> ASA E1/5; NAT moved to R1/R2; PA NAT rules, `DEFAULT-OUT` and ASA standby PAT removed. **Not applied live yet** (section 10) | `r1.ttl`, `r2.ttl`, `firepower1010.ttl`, `pa440.ttl`, `pa440_cleanup.ttl` |
| 26 | After the rebuild, HR/IT had no internet: the PA had no default route. PAN-OS OSPF rejects default routes by default ("Reject Default Route"); the old `DEFAULT-OUT` static had hidden this. The ASA/DSWs still learned R1's default through the PA, so traffic died at the PA (6 Nov) | `set network virtual-router default protocol ospf reject-default-route no` | `pa440.ttl` |
| 24 | HSRP `track 1 interface Vlan100` never fired (Po5 kept the SVI up), and decrement 20 tied 110 with 90, so DSW2's higher IP kept 61/67 | `track 10` (IP SLA to own ASA link), decrement 30 | `dsw1.ttl`, `dsw2.ttl` |

### Why `R1# ping 8.8.8.8` fails (don't re-diagnose)
R1 sources from `172.17.9.93` on the WAN `/30`, which the ISP doesn't route to the internet. Hosts PAT'd to `203.149.210.185` work. Test internet from an HR/IT host, never from R1.

### Static routes: required vs optional
Every static route in the macros has a `REQUIRED` or `OPTIONAL` comment.

| Route | Status |
|-------|--------|
| DSW1 `0.0.0.0/0 via .97 track 10`, DSW2 `via .101 track 10` | OPTIONAL, **not in the macros** (OSPF provides the default via the ASA link at cost 10; the PA bypass at cost 500 is only used when the ASA path is gone) |
| PA `DEFAULT-OUT 0.0.0.0/0 via .90` | **REMOVED 5 Nov** (untracked; would blackhole with R1 down) |
| DSW1/2 `vrf OOB 0.0.0.0/0 via .33` | REQUIRED |
| R1 `0.0.0.0/0 via .94 track 1` | REQUIRED (source of the OSPF default) |
| R2 `0.0.0.0/0 via 172.17.10.94 track 1` | REQUIRED (source of the backup OSPF default) |
| R1 `203.149.210.184/29 Null0`, R2 `129.126.142.184/29 Null0` | OPTIONAL (drops traffic for unused public IPs) |
| Old R1/R2 public-block routes via `.92` / ASA, track 2 | **REMOVED 5 Nov** (the public block ends at the router NAT) |
| ASA `route management 0.0.0.0/0 .33` | REQUIRED |
| PA management `default-gateway .33` | REQUIRED |
| SW1-4 `ip default-gateway .33` | REQUIRED |

---

## 6. Live state vs macros

| Item | Macro | Live | Action |
|------|-------|------|--------|
| PA `HR-IT-INTERNET` | `any/any` (6 Nov: the App-ID list blocked YouTube/Google; plan.md gives HR/IT full internet) | set to `any/any` live after the rebuild | None |
| PA `TEMP-SVR-UPDATES` / `TEMP-SVR-NAT`, ASA `SERVER_IN` www/https/8.8.8.8 lines | not in macro (on purpose) | **present** | Remove after the last install (8.4) |
| DSW HSRP permit (fix #18) | present | applied on both (seen in `ACL_ATM_IN`, 4 Nov) | None (check `ACL_STAFF_IN` too) |
| DSW DHCP peer-reply permit (fix #19) | present | applied on both (seen in `ACL_ATM_IN`, 4 Nov); ATM lease works | None |
| DSW1 Gi1/0/21 management port (VLAN 99) | in macro (6 Nov, section 9b) | configured live (4 Nov) | None |
| Firewall-to-DSW links (fix #23) | routed `/30`s, ASA E1/4 + PA eth1/6/7 up, no PA `.78` / ASA `.86` | old shared VLAN 100, ASA E1/4 + PA eth1/6/7/8 down, ASA Vlan92 shut | Apply with section 10 |
| DSW default route | none (OSPF default only) | plain static via .97 | Reload removes it (the plain static does **not** fail over) |
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
| vmbr1 | eno2 (LAN 2), direct to ASA E1/6 | Internal servers VLAN 69 | none |
| vmbr2 | eno5 (LAN 3), direct to PA eth1/4 | DMZ VLAN 92 | none |
| vmbr3 | bond0 = eno6 (LAN 4, primary, DSW1 Gi1/0/24) + eno1 (LAN 1, backup, DSW2 Gi1/0/24), active-backup | OOB VLAN 99 | 172.16.0.43/28, gw .33 |

Proxmox GUI: `https://172.16.0.43:8006` from the OOB network, e.g. a laptop on DSW1 Gi1/0/21 with `172.16.0.46/28`, gateway `.33`. Don't use LAN 4 for a laptop (see section 4).

### 8.2 Warnings about `supermicro setup.md`
- Its netplan blocks contain stray backslashes (`\-`, `\#`, `\[`, `\]`) from a document export -> `unknown key \network`. Use the templates in 8.3.
- The VM102 block has an invalid extra line `via 172.16.0.86`.
- The backup gateways `.78` and `.86` no longer exist (removed 5 Nov), and Linux doesn't fail over between metric routes anyway. Use one gateway.
- "Linked Clone" needs a template. Use Full Clone.
- Its cabling (eno2 -> ASA eth1/6, eno5 -> PA eth1/4 directly) matches the design again since 5 Nov.

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

**After the 5 Nov NAT change** the PA no longer NATs, so `TEMP-SVR-NAT` is replaced by two lines in each router's `NAT-SOURCES` ACL (the ASA and PA security rules above stay the same):
```
(R1 and R2, configure terminal)
ip access-list extended NAT-SOURCES
 permit ip 172.16.0.72 0.0.0.7 any
 permit ip 172.16.0.80 0.0.0.7 any
```

**Remove when installs are done** (install Wazuh first if you're using it):
```
(ASA)  no access-list SERVER_IN extended permit tcp object SERVERS any eq www
       no access-list SERVER_IN extended permit tcp object SERVERS any eq https
       no access-list SERVER_IN extended permit udp object SERVERS host 8.8.8.8 eq domain
(PA)   delete rulebase security rules TEMP-SVR-UPDATES
       delete rulebase nat rules TEMP-SVR-NAT      (old design only)
       commit
(R1/R2, new design)
       ip access-list extended NAT-SOURCES
        no permit ip 172.16.0.72 0.0.0.7 any
        no permit ip 172.16.0.80 0.0.0.7 any
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
| 100 Web | Nginx, hostname GruppeSechs | HR/IT internal; outside via `203.149.210.189` | PA `INTERNAL-TO-WEB`, `INBOUND-WEB`; R1/R2 static NAT |
| 101 DNS | BIND9 | Authoritative for `gruppesechs.sitict.net` (internal + external view); recursive for HR/IT via the ICT DNS | PA `INTERNAL-TO-DNS`, `INBOUND-DNS`, `DNS-SVR-FORWARD`; R1/R2 static NAT |
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
- If HR's queries time out: confirm the ASA routes the DMZ via the PA (`show route 172.16.0.80` -> via `.105`), and `tcpdump -ni ens18 port 53` on VM101.
- **Internal view: REQUIRED once NAT is on the routers (5 Nov).** Until now inside users got `.189` and the PA's NAT looped them back in. Cisco IOS routers don't hairpin NAT, so `.189` from inside stops working. The internal view gives inside users `172.16.0.82` instead. Only **HR and IT** can reach the website from inside (DSW `ACL_HR_IN`/`ACL_IT_IN`, ASA `INSIDE_IN`, PA `INTERNAL-TO-WEB`), and they're the only ones handed `.83` as their DNS server. Staff and ATM get no DNS server and are blocked from `.82`.

  **Changes on VM101** (back up first: `sudo cp -a /etc/bind /etc/bind.bak`):
  1. Find the current zone file: `grep -A3 'zone "gruppesechs.sitict.net"' /etc/bind/named.conf.local`. Below it's written `<ZONEFILE>`.
  2. Make the internal copy with the private addresses:
     ```bash
     sudo cp <ZONEFILE> /etc/bind/db.gruppesechs.internal
     sudo sed -i 's/203\.149\.210\.189/172.16.0.82/g; s/203\.149\.210\.190/172.16.0.83/g' /etc/bind/db.gruppesechs.internal
     ```
     Bump the SOA serial in the new file. Check with `grep -E 'IN\s+A' /etc/bind/db.gruppesechs.internal`: every A record should now be `172.16.0.8x`.
  3. With views, **every** zone must be inside a view. In `/etc/bind/named.conf`, comment out the default-zones include:
     ```
     // include "/etc/bind/named.conf.default-zones";
     ```
  4. Replace the existing `zone "gruppesechs.sitict.net" { ... };` block in `/etc/bind/named.conf.local` with:
     ```
     acl internal-users { localhost; 172.16.0.56/29; 172.16.0.64/29; 172.16.0.80/29; };

     view "internal" {
         match-clients { internal-users; };
         recursion yes;
         zone "gruppesechs.sitict.net" {
             type master;
             file "/etc/bind/db.gruppesechs.internal";
         };
         include "/etc/bind/named.conf.default-zones";
     };

     view "external" {
         match-clients { any; };
         recursion no;
         zone "gruppesechs.sitict.net" {
             type master;
             file "<ZONEFILE>";
         };
     };
     ```
     `internal-users` = HR, IT and the DMZ itself (so the web/jump hosts resolve the name to `.82` too). The lab resolver and the internet land in `external` and keep getting `.189` with recursion off, so `.83` doesn't become an open resolver. `allow-recursion` in `named.conf.options` can stay as it is.
  5. Check and restart:
     ```bash
     sudo named-checkconf
     sudo named-checkzone gruppesechs.sitict.net /etc/bind/db.gruppesechs.internal
     sudo systemctl restart named
     ```
  6. Test from an HR/IT PC: `nslookup gruppesechs.sitict.net` -> `172.16.0.82`, then browse `https://gruppesechs.sitict.net`. From outside (lab Wi-Fi): `nslookup gruppesechs.sitict.net` -> `203.149.210.189`.

  After every future zone edit, change **both** zone files (and both serials).

### 8.11 Still to set up on VM102/106

| Item | Notes |
|------|-------|
| **chrony** (NTP for the OOB devices) | **DONE 4 Nov (VM102 checked; confirm VM106 config matches).** Check it's serving with `sudo ss -ulpn \| grep chronyd` (must show `0.0.0.0:123`), and `chronyc -N sources`. After editing the config, restart chrony and confirm port 123 is open; on a Cisco device, reset a stuck association with `no ntp server vrf OOB 172.16.0.44` then `ntp server vrf OOB 172.16.0.44 prefer`. Devices picking `.45` over `.44` is fine when VM102 syncs from VM106 (one stratum lower); add `noselect` to each VM's `server <other TACACS>` line to make both prefer the internet pool. Setup: **on both VM102 and VM106** (devices use `.44` prefer + `.45`). In `/etc/chrony/chrony.conf`: comment out the Ubuntu `pool ...` lines, add `pool 0.sg.pool.ntp.org iburst maxsources 4`, `server 172.16.0.45 iburst` (on VM106: `.44`), `allow 172.16.0.32/28`, `local stratum 10 orphan`; `sudo systemctl restart chrony`. Upstream NTP leaves via ens19 (PA `DMZ-OUTBOUND-NTP`, router `NAT-SOURCES`); devices query over ens18 (OOB). Tests: `TEST-PLAN.md` 7.13-7.16. Fixes P14. |
| **tac_plus** | Config in `plan.md` ("TACACS config"). **Read P13 first.** Keep a console session open on each device, then test with `test aaa group tac-group bigDonkBonkAdmin <pw> legacy`. Re-enable ASA `aaa authorization command TACACS LOCAL` only after it works. |
| **Wazuh** | Manager needs roughly 4 GB RAM. Install while the 8.4 temporary rules are up. Agents on internal servers need P17 rules (ASA `SERVER_IN` + PA `inside -> dmz`, tcp/1514-1515). |
| **NetFlow analyzer** | Devices already export to `.44`/`.45` udp/2055 over the OOB VRF. |

---

## 9. Next steps (in order)

0. **Apply the 5 Nov redesign** (section 10: firewall links, second ISP, NAT on the routers) and the VM101 internal view (8.10), then run `TEST-PLAN.md` Stage 0, 6 and 9.
1. ~~Fix the jump host -> R1 OOB problem (P19)~~ Resolved. **Finish Stage 7**: NTP works on R1 (check the other devices with `show ntp associations`), then SSH to every device from both jump hosts.
2. ~~Confirm fixes #18 and #19 live on both DSWs~~ **Confirmed 4 Nov**; ATM got `.52`. Still to do: ATM -> Accounts (`TEST-PLAN.md` 4.9) and re-check HSRP (0.3).
3. **Add DSW1 Gi1/0/21** (management port) to `dsw1.ttl` once confirmed.
4. **Set up chrony, then tac_plus (carefully), then Wazuh**, while the temporary rules are still up.
5. **Remove the temporary rules** (8.4) and run `TEST-PLAN.md` 8.5 and 8.9.
6. **Run the remaining tests** (Stages 2-6 deny tests, Stage 8 Txn window).
7. **Fix P6** (R1 `WAN-IN` `.94` -> `.93`) and decide on P3, P11, P15.
8. **Final hardening (Stage 10):** remove the temporary ICMP permits on both DSWs, re-enable ASA command authorization (after TACACS works), re-test.
9. **Demo day:** set the final TXN-WINDOW (ASA `time-range`, PA `schedule`).

---

## 10. Applying the 5 Nov firewall-link redesign to the live devices

The macros describe the new design. A device rebuilt from its macro after a reload gets it automatically. To change the **running** devices without a reload, use the steps below. Do it in one session; expect short interruptions while OSPF reconverges. Untested on the devices: check each step's output before moving on.

**Order:** cabling -> R1 + R2 (10.1b) -> PA -> DSW1 -> DSW2 -> ASA. Do the PA straight after the routers: between the two steps outbound internet still works (the PA's `.185` source isn't in the routers' `NAT-SOURCES`, so it passes through untranslated), but inbound web/DNS/jump SSH is down until the PA's rules use the real server IPs. The PA goes before the DSWs so its eth1/6/7 bypass is up while the ASA links are being changed. On each DSW, Gi1/0/12 (PA link) is changed first for the same reason.

### 10.1 Cabling
- Confirm ASA E1/4 <-> DSW2 Gi1/0/11 is connected.
- Routers (second ISP): remove the R1 G0/1/1 <-> R2 G0/1/1 cable. Connect R1 G0/1/1 -> ASA E1/5. Move R2's ASA cable from R2 G0/0/0 to R2 G0/1/1 (ASA end stays E1/2). Connect R2 G0/0/0 -> ISP2.
- At the end, unplug PA eth1/8 <-> DSW1 Gi1/0/15 and ASA E1/7 <-> DSW2 Gi1/0/15.
- Server recabling (each move cuts that zone for a few seconds; no firewall or VM config changes, both ends stay untagged):
  - Unplug ASA E1/6 from DSW1 Gi1/0/14 and Proxmox eno2 from DSW1 Gi1/0/13. Connect eno2 straight to ASA E1/6.
  - Unplug PA eth1/4 from DSW2 Gi1/0/14 and Proxmox eno5 from DSW2 Gi1/0/13. Connect eno5 straight to PA eth1/4.
  - Then on both DSWs (10.3 / 10.4, or afterwards):
    ```
    interface range GigabitEthernet1/0/13 - 15
     description UNUSED_PORTS
     switchport mode access
     switchport access vlan 727
     shutdown
    interface Port-channel5
     switchport trunk allowed vlan 23,46,61,67,99,100
    no spanning-tree vlan 69 root
    no spanning-tree vlan 92 root
    no vlan 69
    no vlan 92
    ```

### 10.1b Routers: second ISP and NAT (`configure terminal`)

**R1:**
```
no ip route 203.149.210.184 255.255.255.248 172.16.0.92 track 2
no ip route 203.149.210.184 255.255.255.248 172.16.0.110 250
no track 2
no ip sla 2
ip access-list extended NAT-SOURCES
 permit ip 172.16.0.56 0.0.0.7 any
 permit ip 172.16.0.64 0.0.0.7 any
 permit ip host 172.16.0.76 any
 permit ip host 172.16.0.83 any
 permit ip host 172.16.0.84 any
 permit ip host 172.16.0.85 any
exit
interface GigabitEthernet0/0/0
 ip nat outside
interface GigabitEthernet0/0/1
 ip nat inside
interface GigabitEthernet0/1/1
 description PA_Bypass_Link_to_ASA_E1_5
 ip nat inside
 ip ospf cost 500
exit
ip route 203.149.210.184 255.255.255.248 Null0
ip nat pool PUB-PAT 203.149.210.185 203.149.210.185 prefix-length 29
ip nat inside source list NAT-SOURCES pool PUB-PAT overload
ip nat inside source static tcp 172.16.0.82 80 203.149.210.189 80 extendable
ip nat inside source static tcp 172.16.0.82 443 203.149.210.189 443 extendable
ip nat inside source static udp 172.16.0.83 53 203.149.210.190 53 extendable
ip nat inside source static tcp 172.16.0.83 53 203.149.210.190 53 extendable
ip nat inside source static tcp 172.16.0.84 22 203.149.210.185 2222 extendable
```
If the last line is rejected because `.185` is also the PAT pool, change the pool to `.186` (both addresses in the `ip nat pool` line) and re-enter it. If the 8.4 temporary installs are still needed, add the two `NAT-SOURCES` lines from 8.4.

**R2** (the order matters: G0/0/0 must give up `.113` before G0/1/1 can take it):
```
ip access-list extended WAN-IN
 deny tcp any host 172.17.10.93 eq 22 log
 deny tcp any host 172.17.10.93 eq 23 log
 permit icmp host 172.17.10.94 host 172.17.10.93 echo-reply
 permit ip any 129.126.142.184 0.0.0.7
 permit tcp any host 172.17.10.93 established
 deny ip any any log
ip access-list extended NAT-SOURCES
 permit ip 172.16.0.56 0.0.0.7 any
 permit ip 172.16.0.64 0.0.0.7 any
 permit ip host 172.16.0.76 any
 permit ip host 172.16.0.83 any
 permit ip host 172.16.0.84 any
 permit ip host 172.16.0.85 any
exit
no ip route 203.149.210.184 255.255.255.248 172.16.0.92 track 2
no ip route 203.149.210.184 255.255.255.248 172.16.0.114 250
no track 2
no ip sla 2
interface GigabitEthernet0/0/0
 description WAN_Link_to_ISP2
 no ip ospf cost 500
 ip address 172.17.10.93 255.255.255.252
 ip access-group WAN-IN in
 ip nat outside
interface GigabitEthernet0/1/1
 description PA_Bypass_Link_to_ASA_E1_2
 ip address 172.16.0.113 255.255.255.252
 ip ospf cost 500
 ip nat inside
interface GigabitEthernet0/0/1
 ip nat inside
exit
ip sla 1
 icmp-echo 172.17.10.94 source-interface GigabitEthernet0/0/0
 frequency 5
exit
ip sla schedule 1 life forever start-time now
track 1 ip sla 1 reachability
exit
ip route 0.0.0.0 0.0.0.0 172.17.10.94 track 1
ip route 129.126.142.184 255.255.255.248 Null0
router ospf 1
 passive-interface GigabitEthernet0/0/0
 no network 172.16.0.108 0.0.0.3 area 0
 default-information originate metric 50 metric-type 1
exit
ip nat pool PUB-PAT 129.126.142.185 129.126.142.185 prefix-length 29
ip nat inside source list NAT-SOURCES pool PUB-PAT overload
ip nat inside source static tcp 172.16.0.82 80 129.126.142.189 80 extendable
ip nat inside source static tcp 172.16.0.82 443 129.126.142.189 443 extendable
ip nat inside source static udp 172.16.0.83 53 129.126.142.190 53 extendable
ip nat inside source static tcp 172.16.0.83 53 129.126.142.190 53 extendable
ip nat inside source static tcp 172.16.0.84 22 129.126.142.185 2222 extendable
```
Checks: `show track 1` Up on both; `show ip ospf neighbor` on R1 (PA `.92`, R2 `.91` on G0/0/1; ASA `.110` on G0/1/1 once 10.5 is done); `ping 172.17.10.94` from R2.

### 10.2 PA
Run `pa440_cleanup.ttl`, then `pa440.ttl`. The cleanup also deletes the PA NAT rules, the `PUB-*` objects and the `DEFAULT-OUT` static route. The cleanup's new section 7 removes vlan.100, vlan.69, `VL-INSIDE-BYPASS`, `VL-SERVER`, zones `inside-l2` / `server` / `server-l2`, and the layer2 config on eth1/6/7/8.

Side effects of re-running the macros:
- The cleanup deletes `TEMP-SVR-UPDATES` and `TEMP-SVR-NAT` (8.4). If installs (Wazuh) are still pending, re-add only `TEMP-SVR-UPDATES`, plus the router `NAT-SOURCES` lines from 8.4. `TEMP-SVR-NAT` isn't needed any more.
- `HR-IT-INTERNET` is `any/any` in the macro (changed 6 Nov, see section 6).

Check: `show interface ethernet1/6` and `1/7` up with `.117/30` / `.121/30`.

### 10.3 DSW1 (`configure terminal`)
```
interface GigabitEthernet1/0/12
 no switchport
 description Routed_Bypass_to_PA_eth1_6
 ip address 172.16.0.118 255.255.255.252
 ip ospf cost 500
 ip flow monitor bank-flow-monitor input
 no ip redirects
 no ip proxy-arp
 no shutdown
router ospf 1
 no passive-interface GigabitEthernet1/0/12
 network 172.16.0.116 0.0.0.3 area 0
! Vlan100 must give up .98 before Gi1/0/11 can take it
interface Vlan100
 description DSW1_DSW2_Transit_over_Po5
 ip address 172.16.0.125 255.255.255.252
interface GigabitEthernet1/0/11
 no switchport
 description Routed_Transit_to_ASA_E1_3
 ip address 172.16.0.98 255.255.255.252
 ip ospf cost 10
 ip flow monitor bank-flow-monitor input
 no ip redirects
 no ip proxy-arp
 no shutdown
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 shutdown
vlan 100
 name DSW_TRANSIT
router ospf 1
 no network 172.16.0.96 0.0.0.7 area 0
 network 172.16.0.96 0.0.0.3 area 0
 network 172.16.0.124 0.0.0.3 area 0
 no passive-interface GigabitEthernet1/0/11
no ip sla 10
ip sla 10
 icmp-echo 172.16.0.97 source-interface GigabitEthernet1/0/11
 frequency 5
ip sla schedule 10 life forever start-time now
track 10 ip sla 10 reachability
no ip route 0.0.0.0 0.0.0.0 172.16.0.97
ip route 0.0.0.0 0.0.0.0 172.16.0.97 track 10
interface Vlan23
 ip ospf cost 10
 no standby 23 track 1
 standby 23 track 10 decrement 30
interface Vlan46
 ip ospf cost 10
 no standby 46 track 1
 standby 46 track 10 decrement 30
interface Vlan61
 ip ospf cost 20
 no standby 61 track 1
 standby 61 track 10 decrement 30
interface Vlan67
 ip ospf cost 20
 no standby 67 track 1
 standby 67 track 10 decrement 30
no track 1
```

### 10.4 DSW2 (`configure terminal`)
```
interface GigabitEthernet1/0/12
 no switchport
 description Routed_Bypass_to_PA_eth1_7
 ip address 172.16.0.122 255.255.255.252
 ip ospf cost 500
 ip flow monitor bank-flow-monitor input
 no ip redirects
 no ip proxy-arp
 no shutdown
router ospf 1
 no passive-interface GigabitEthernet1/0/12
 network 172.16.0.120 0.0.0.3 area 0
interface Vlan100
 description DSW1_DSW2_Transit_over_Po5
 ip address 172.16.0.126 255.255.255.252
interface GigabitEthernet1/0/11
 no switchport
 description Routed_Transit_to_ASA_E1_4
 ip address 172.16.0.102 255.255.255.252
 ip ospf cost 10
 ip flow monitor bank-flow-monitor input
 no ip redirects
 no ip proxy-arp
 no shutdown
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 shutdown
vlan 100
 name DSW_TRANSIT
router ospf 1
 no network 172.16.0.96 0.0.0.7 area 0
 network 172.16.0.100 0.0.0.3 area 0
 network 172.16.0.124 0.0.0.3 area 0
 no passive-interface GigabitEthernet1/0/11
no ip sla 10
ip sla 10
 icmp-echo 172.16.0.101 source-interface GigabitEthernet1/0/11
 frequency 5
ip sla schedule 10 life forever start-time now
track 10 ip sla 10 reachability
no ip route 0.0.0.0 0.0.0.0 172.16.0.97
ip route 0.0.0.0 0.0.0.0 172.16.0.101 track 10
interface Vlan23
 ip ospf cost 20
 no standby 23 track 1
 standby 23 track 10 decrement 30
interface Vlan46
 ip ospf cost 20
 no standby 46 track 1
 standby 46 track 10 decrement 30
interface Vlan61
 ip ospf cost 10
 no standby 61 track 1
 standby 61 track 10 decrement 30
interface Vlan67
 ip ospf cost 10
 no standby 67 track 1
 standby 67 track 10 decrement 30
no track 1
```
Track 10 stays Down on DSW2 until the ASA's E1/4 is configured (10.5). HSRP moves 61/67 to DSW1 meanwhile; that's expected.

### 10.5 ASA (`configure terminal`)
```
! Remove the DMZ backup (references first, then the interface)
no access-group DMZ_IN in interface dmz
no nat (dmz,wan-backup) after-auto source dynamic PAT-USERS PUB-PAT
object network WEB-SVR-NAT
 no nat (dmz,wan-backup) static 203.149.210.189
object network DNS-SVR-NAT
 no nat (dmz,wan-backup) static 203.149.210.190
object network JUMP-SSH-NAT
 no nat (dmz,wan-backup) static 203.149.210.185 service tcp ssh 2222
exit
no object network WEB-SVR-NAT
no object network DNS-SVR-NAT
no object network JUMP-SSH-NAT
clear configure access-list DMZ_IN
no access-list OUTSIDE_IN extended permit tcp any object WEB-SVR eq www
no access-list OUTSIDE_IN extended permit tcp any object WEB-SVR eq https
no access-list OUTSIDE_IN extended permit udp any object DNS-SVR eq domain
no access-list OUTSIDE_IN extended permit tcp any object DNS-SVR eq domain
no access-list OUTSIDE_IN extended permit tcp any object JUMP-SVR eq ssh
interface Vlan92
 no nameif
exit
no interface Vlan92
interface Ethernet1/7
 description UNUSED_PORT
 shutdown
! Split VLAN 100 into two /30s
router ospf 1
 no network 172.16.0.96 255.255.255.248 area 0
interface Ethernet1/3
 ip address 172.16.0.97 255.255.255.252
interface Ethernet1/4
 description Inside_Transit_to_DSW2_G1_0_11
 no switchport
 nameif inside2
 security-level 100
 ip address 172.16.0.101 255.255.255.252
 speed 1000
 duplex full
 ospf cost 10
 no shutdown
router ospf 1
 network 172.16.0.96 255.255.255.252 area 0
 network 172.16.0.100 255.255.255.252 area 0
access-group INSIDE_IN in interface inside2
! NAT moved to the routers: remove the standby PAT
no nat (inside,wan-backup) after-auto source dynamic PAT-USERS PUB-PAT
no nat (server,wan-backup) after-auto source dynamic PAT-USERS PUB-PAT
no object-group network PAT-USERS
no object network PUB-PAT
! Second router link: E1/5 to R1 G0/1/1 (E1/2 stays R2, now on R2 G0/1/1)
interface Ethernet1/2
 description PA_Bypass_Link_to_R2_G0_1_1
interface Ethernet1/5
 description PA_Bypass_Link_to_R1_G0_1_1
 no switchport
 nameif wan-backup2
 security-level 0
 ip address 172.16.0.110 255.255.255.252
 speed 1000
 duplex full
 ospf cost 500
 no shutdown
router ospf 1
 network 172.16.0.108 255.255.255.252 area 0
access-group OUTSIDE_IN in interface wan-backup2
object-group network DSWs
 no network-object host 172.16.0.99
 network-object host 172.16.0.102
object-group network DSW-SVIS
 no network-object host 172.16.0.99
 network-object host 172.16.0.102
```
Some `no` lines may report "does not exist" if that piece was never applied live. That's harmless.

### 10.6 Verify
`TEST-PLAN.md` Stage 0 (0.3-0.6b, 0.9-0.11, 0.13-0.18), then 2.1 (HR internet), 4.x (HR file share), 5.x (HR -> DMZ DNS/web), Stage 6 (inbound), then Stage 9. Do the VM101 internal view (8.10) in the same session, or HR/IT lose the website by name.
