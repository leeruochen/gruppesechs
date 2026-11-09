# Punggol Bank Network: Handover Document

**Rack:** 4E (`gruppesechs.sitict.net`)
**Topology:** Full PTSO (Parallel Topologically, Serial Operationally)
**Last updated:** 9 November 2026 (section 12 worked through end to end: Wazuh dashboard, log sources, agents, cleanup and Stages 9-14 all done. Suricata IDS added, see `IDS-SURICATA.md`. The 9 Nov session is written up in `changes.md`; all of it is now folded into this file and the macros.)

This file is the full context for continuing on another device. Chat history from the AI assistant does not transfer between machines, so everything needed is written here. Test results live in `TEST-PLAN.md`; the 9 Nov blow-by-blow is in `changes.md`.

## Where things stand

- **Network:** end-to-end path works. HR and IT reach the internet through DSW -> ASA -> PA -> R1 -> ISP.
- **Servers:** all built. DHCP (VM107), DNS (VM101), File/Samba (VM103), Accounts portal (VM104), Txn (VM105), Web (VM100), TACACS/jump hosts (VM102/106).
- **Seen working (4 Nov):** DHCP leases (HR, IT, Staff), internet and browsing for HR/IT, internal DNS + other racks' names (e.g. `wkg00n.sitict.net`), HR file share, Staff -> Accounts portal, web server from inside and outside over HTTPS, outside SSH to the jump host (`.185:2222`).
- **Done since (9 Nov, `changes.md`):**
  1. **Wazuh is live.** Dashboard open; device syslog + PA log forwarding (`LFP-WAZUH`) reach it and decode; 6 agents Active (VM100/101/103/104/105/107); Suricata IDS on VM102 feeds it too. **No agent on VM106** (decision, `changes.md` Part 4.5).
  2. **All temporary rules removed** (section 12 D1, 9 Nov): the install rules (8.4), the temporary jump-to-servers SSH rule, and the two router `NAT-SOURCES` server lines. Internal servers now reach the internet only inside the TXN window (VM105) and via the Proxmox console.
  3. **Final hardening done:** DSW temporary ICMP permits removed (Stage 10), R1 `WAN-IN` redundant ICMP line removed, R1/R2 `WAN-IN` SSH/Telnet denies now reference `.93` (P6).
  4. **Stages 9-14 of `TEST-PLAN.md` done** (redundancy, AAA, logging, TXN window, DNS exposure, self-audit, attack readiness). See the `changes.md` parts and the filled-in `TEST-PLAN.md`.
  5. **ASA AAA rebuilt:** console login authentication added (`enable_15` bug, fix #29), engineer auto-enable (fix #30), command authorization re-confirmed on. tac_plus service is **`tac_plus-ng`** (config `/etc/tac_plus-ng.cfg`, logs `/var/log/tac_plus-ng/`).
  6. **R1/R2 web NAT changed to 1:1** so port scans reach the PA and are logged/blocked (fix #33, `changes.md` Part 3).
- **Open / still to do:**
  1. **Macros vs live:** all 9 Nov live config is now in the macros (this edit). A rebuild-from-macro rehearsal has still not been run.
  2. **Credentials to rotate** (exposed during 9 Nov troubleshooting, keep out of repo): Wazuh admin/kibanaserver/wazuh-wui; ASA local passwords for `bigDonkBonkAdmin` and `donkBonkEng1`; TACACS/engine passwords. See `changes.md` Part 16.2.
  3. **Housekeeping:** fix SSH hardening filename on VM102 (`99-hardening-conf` -> `99-hardening.conf`); set VM106 timezone to Asia/Singapore. (PA zone-protection values restored and the leftover `TEMP-JUMP-UPDATES` rule deleted, 9 Nov evening.)
  4. **Not adopted (by decision):** R1/R2 track `delay` dampening (14.1); Wazuh agent on VM106; `show running-config` deny for engineers (left open).
  5. **Demo prep (section 12 H):** save a `show running-config` from every device; walk `DEMO-DAY.md`; set the final TXN window on the day.

---

## 1. Repository layout

```
configs/
├── HANDOVER.md            <- this file
├── changes.md             <- 9 Nov lab session log (section 12 work, fixes #29-#33), Parts 1-16
├── TEST-PLAN.md           <- connectivity test matrix with results (Stages 0-14)
├── TOPOLOGY.md            <- topology and failover table
├── HARDENING.md           <- hardening commands by device (section numbers used in section 11)
├── hardening-changes.md   <- what was actually applied live, with device output (7 Nov)
├── IDS-SURICATA.md        <- Suricata IDS on VM102 (DMZ mirror -> Wazuh), 9 Nov
├── ATTACKS.md             <- attack catalogue (16 attacks, defences, verification)
├── LAB-COVERAGE.md        <- labs 1.1-3.5 mapped to the configs
├── DEMO-DAY.md            <- demo-day commands (TXN window, attack runbook)
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
| DMZ VLAN 92 | 172.16.0.80/29 | PA .81, Web .82, DNS .83, Jump1 .84, Jump2 .85, Caching DNS .86 (VM108, 9 Nov) |
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
| Outbound PAT (`NAT-SOURCES`: HR, IT, Txn, DNS, caching DNS, jump hosts) | 203.149.210.185 | 129.126.142.185 | |
| Jump SSH | 203.149.210.185 tcp/2222 | 129.126.142.185 tcp/2222 | 172.16.0.84:22 |
| Web | **203.149.210.189** 1:1 (all ports, PA filters) | 129.126.142.189 1:1 | 172.16.0.82 |
| DNS | **203.149.210.190** udp/tcp 53 | 129.126.142.190 | 172.16.0.83 |

Web is **1:1 static NAT** on R1/R2 since 9 Nov (`changes.md` Part 3): every port now reaches the PA, which drops the unused ones with `DENY-ALL` and lets its zone protection see and block scanners. Port-only NAT (80/443) used to sink the rest to Null0 at the router, so the PA never saw a scan. DNS and jump SSH keep port-specific statics.

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

### Shared keys and lockouts (hardening, 7 Nov)

| Purpose | Key | Where it must match |
|---|---|---|
| NTP (key id 1, MD5) | `bigBoyNTPkey` | `/etc/chrony/chrony.keys` on **both** VM102 and VM106 (`1 MD5 bigBoyNTPkey`), all Cisco devices, ASA, PA |
| OSPF MD5 (key id 1) | `GsOspf2026` | Both ends of every OSPF link (R1, R2, DSW1, DSW2, ASA, PA profile `OSPF-MD5`) |
| HSRP MD5 | `GsHsrp2026` | DSW1 and DSW2, VLANs 23/46/61/67 |

(`HARDENING.md` lists `GsNtpKey2026` for NTP. That value was **not** used; the real key is `bigBoyNTPkey`.)

- **ASA:** local accounts lock after 5 failed logins (`aaa local authentication attempts max-fail 5`; privilege 15 is exempt). Unlock: `clear aaa local user lockout username <user>`. Command authorization (`aaa authorization command TACACS LOCAL`) is **on**: keep a console session open when rebuilding the ASA in case tac_plus is down.
  - **Console login (9 Nov, `changes.md` Part 1):** the console now authenticates too (`aaa authentication serial console TACACS LOCAL` + `aaa authentication enable console TACACS LOCAL`). Before this, a console `enable` ran the session as the pseudo-user `enable_15`, which no tac_plus profile matches, so command authorization denied **every** command, including `exit`. The console login now counts toward `max-fail 5`.
  - **Emergency unlock:** a TACACS **deny** never falls back to LOCAL; only an **unreachable** server does. To get out when command authorization locks you: `sudo systemctl stop tac_plus-ng` on **both** VM102 and VM106. After starting it again the ASA stays on LOCAL for ~10 min (dead time) unless you run `aaa-server TACACS active host 172.16.0.44` (and `.45`); during that window it wants the **local** password.
  - **Local passwords differ from `plan.md`:** `bigDonkBonkAdmin` and `donkBonkEng1` were forced to change at first login; kept in private notes. `donkBonkEng2` will change at its first ASA login. LOCAL is used only when both TACACS servers are unreachable.
- **TACACS service:** it is **`tac_plus-ng`**, not `tac_plus`. Config `/etc/tac_plus-ng.cfg`; logs `/var/log/tac_plus-ng/authc.log` (logins), `authz.log` (per-command permit/deny), `acct.log` (accounting). Read `authz.log` first when a command is refused — the username on the deny line is the cause (`enable_15` = no login on that line). `bigDonkBonkAdmin` has no sudo on the VMs; use `ubuntu` for service control.
- **PA:** admin lockout after 5 failed logins for 5 minutes; idle timeout 10 minutes.
- **SW1-SW4:** err-disabled ports (BPDU guard, DHCP rate limit, DAI) recover by themselves after 300 s. Port security (changed 9 Nov, live and in the macros): max 5 **dynamic** MACs per access port, `violation restrict` (drop and log, no err-disable), `aging time 5` + `aging type inactivity`. Sticky was removed: it never aged out and broke moving laptops between ports. Nothing to clear by hand.

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
| 23 | Backup links needed manual `no shutdown` (not allowed in the demo); ASA failure was a full outage (P8) | 5 Nov redesign: firewall-to-DSW links are routed `/30`s (ASA E1/3 + E1/4, PA eth1/6 + eth1/7), all up; server-subnet backups (PA `.78`, ASA `.86`) removed; HSRP and default route track each DSW's own ASA link. Applied live (OSPF neighbours match the new design, checked 7 Nov) | all firewall and DSW macros |
| 25 | Single ISP; failover depended on a PA-side static default | Second ISP on R2 (`172.17.10.93/30`, block `129.126.142.184/29`); R1-R2 link replaced by R1 <-> ASA E1/5; NAT moved to R1/R2; PA NAT rules, `DEFAULT-OUT` and ASA standby PAT removed. Applied live (OSPF neighbours match the new design, checked 7 Nov) | `r1.ttl`, `r2.ttl`, `firepower1010.ttl`, `pa440.ttl`, `pa440_cleanup.ttl` |
| 26 | After the rebuild, HR/IT had no internet: the PA had no default route. PAN-OS OSPF rejects default routes by default ("Reject Default Route"); the old `DEFAULT-OUT` static had hidden this. The ASA/DSWs still learned R1's default through the PA, so traffic died at the PA (6 Nov) | `set network virtual-router default protocol ospf reject-default-route no` | `pa440.ttl` |
| 27 | Every device showed `.45` as `insane` / `reach 0` after NTP auth was turned on (7 Nov). VM106's `chrony.keys` had no key, so its replies were unsigned and rejected. Devices `prefer` `.44`, so nothing looked wrong until the `.45` line was checked | `1 MD5 bigBoyNTPkey` added to VM106's `chrony.keys`, chrony restarted | VM106 |
| 28 | ASA lost `authenticated` on `.44` after `no ntp authentication-key 1 md5 GsNtpKey2026`: that command deletes key 1 **whatever value is typed**, which removed the new key too | Re-enter `ntp authentication-key 1 md5 bigBoyNTPkey`. To change a key, just enter the new line | `firepower1010.ttl` (comment) |
| 24 | HSRP `track 1 interface Vlan100` never fired (Po5 kept the SVI up), and decrement 20 tied 110 with 90, so DSW2's higher IP kept 61/67 | `track 10` (IP SLA to own ASA link), decrement 30 | `dsw1.ttl`, `dsw2.ttl` |
| 29 | ASA console session ran as `enable_15` (no username), so TACACS denied every command under command authorization, including `exit` | `aaa authentication serial console TACACS LOCAL` + `aaa authentication enable console TACACS LOCAL` | `firepower1010.ttl` |
| 30 | ASA engineers stuck at `>` (TACACS has no `enable =` for engineers, and a TACACS fail never falls back to LOCAL); and command authorization had been left off after the Part 1 unlock | `aaa authorization exec authentication-server auto-enable` (applies the login priv level, no `enable` needed); command authorization re-enabled | `firepower1010.ttl` |
| 31 | DSW1 Gi1/0/12 OSPF key was `Cisco123!`, not `GsOspf2026`; the cost-500 PA-DSW1 bypass adjacency had been down since the 7 Nov hardening (the "all 9 FULL" check missed it). Found from the PA's forwarded `routed-OSPF-md5chksum-bad` system log | Re-keyed the interface to `GsOspf2026` (live). `dsw1.ttl` was already correct | live (DSW1) |
| 32 | R1 `WAN-IN` line 50 (`permit ip any 203.149.210.184 0.0.0.7`) removed by mistake during D3, which dropped all HR/IT internet and all inbound web/DNS/jump SSH | Restored line 50. **It must never be removed** (it is the permit that lets return traffic and inbound services reach the public block) | live (R1) |
| 33 | Port-only static NAT (80/443) dropped inbound scans at R1/R2 Null0, so PA zone protection never saw them, never logged a scan and never blocked the scanner | Web NAT changed to **1:1** on R1/R2; the PA now filters every port | `r1.ttl`, `r2.ttl` |

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
| PA `TEMP-SVR-UPDATES`, ASA `SERVER_IN` www/https/8.8.8.8 lines, router `NAT-SOURCES` server lines | not in macro (on purpose) | **removed 9 Nov** (D1, `changes.md` Part 12.2) | None. `suricata-update` will now fail without internet |
| PA `TEMP-JUMP-UPDATES` (VM102 web access for the Suricata install) | not in macro (on purpose); `pa440_cleanup.ttl` deletes it | **removed 9 Nov evening** (commit job 99; missed by D1) | None |
| PA `ZP-OUTSIDE` values | SYN 100/200/2000, scan 8001 threshold 100 | **restored 9 Nov evening** (commit job 101) from the tuned test values (500/1000/2000, threshold 20). Confirmed: `show zone-protection zone outside` = SYN 100/200/2000; scan 8001/8002/8003 threshold 100 | None |
| ASA `SERVER_IN` line 1 Wazuh permit (tcp/1514-1515 SERVERS -> JUMP-SVR) | **now in `firepower1010.ttl`** (9 Nov) | present (live 9 Nov) | None |
| PA `WAZUH-AGENTS` rule + `WAZUH-AGENT` service | **now in `pa440.ttl`** before `DENY-ALL` (9 Nov) | present (live 9 Nov, commit job 81) | None |
| PA `LFP-WAZUH` log forwarding (profile, match lists, every `log-setting`, zones) | **now in `pa440.ttl`** (section 10b, 9 Nov) | committed and complete (12.11 pass) | None |
| R1/R2 web NAT | 1:1 static NAT in `r1.ttl` / `r2.ttl` (9 Nov) | 1:1 (live 9 Nov) | None |
| ASA console AAA (`aaa authentication serial/enable console`, `aaa authorization exec ... auto-enable`) | **now in `firepower1010.ttl`** (9 Nov) | present (live 9 Nov) | None |
| PA admin authentication (`AUTH-TACACS`, `AUTH-LOCAL`, sequence `SEQ-ADMIN`, `deviceconfig system authentication-profile SEQ-ADMIN`), `timezone Asia/Singapore`, `hostname-type-in-syslog FQDN` | **was missing; added to `pa440.ttl` 9 Nov** (found by the `runshow.md` audit) | present | None |

**Macro audit against `runshow.md` (captured 9 Nov ~21:40, all 10 devices).** Every macro line was checked against the running config per interface/ACL/block, with type-7 keys decoded. Everything matches (1:1 NAT, `NAT-SOURCES`, ACLs without the ICMP permits, OSPF keys on every link incl. DSW1 Gi1/0/12, TACACS and NTP keys, port-security `restrict`/aging, ASA AAA and `SERVER_IN`, the whole PA rulebase/DoS/log forwarding/zone protection) **except** the live drifts below. The macros are correct for all of them. **All fixed live on 9 Nov** (after the audit); the table is kept as a record:

| Device | Live drift | Fix (live) |
|---|---|---|
| DSW2 | `interface Vlan46` HSRP key is `GsOspf2026` (DSW1 uses `GsHsrp2026`), so DSW1 and DSW2 reject each other's VLAN 46 hellos and can both go Active for the ATM gateway | `interface Vlan46` / `no standby 46 authentication` / `standby 46 authentication md5 key-string GsHsrp2026`; check `show standby vlan 46 brief` on both (DSW1 Active, DSW2 Standby) |
| R2 | `WAN-IN` line 9 is `deny ip 203.149.210.184 0.0.0.7 any log` (R1's block) instead of R2's own `129.126.142.184 0.0.0.7` | `ip access-list extended WAN-IN` / `no deny ip 203.149.210.184 0.0.0.7 any log` / `9 deny ip 129.126.142.184 0.0.0.7 any log` |
| SW3 | Unused port Gi1/0/10 is not shut (left up after the 12.25 test) | `interface Gi1/0/10` / `shutdown` |
| ASA | `INSIDE_IN` `permit tcp object-group HR-IT object WEB-SVR eq www` is the **last** line, after `deny ip any any log`, so HR/IT `http://172.16.0.82` is denied (HTTPS still works). The macro order is correct | `no access-list INSIDE_IN extended permit tcp object-group HR-IT object WEB-SVR eq www`, then `access-list INSIDE_IN line 9 extended permit tcp object-group HR-IT object WEB-SVR eq www` (check with `show access-list INSIDE_IN` that it sits just above the `eq https` line) |
| ASA | `ntp trusted-key 1` missing (probably lost in the fix #28 incident) | `ntp trusted-key 1`; check `show ntp associations detail` shows `.44` authenticated |
| R1 | `logging console informational` (macro: `no logging console`) | `no logging console` (cosmetic) |
| DSW HSRP permit (fix #18) | present | applied on both (seen in `ACL_ATM_IN`, 4 Nov) | None (check `ACL_STAFF_IN` too) |
| DSW DHCP peer-reply permit (fix #19) | present | applied on both (seen in `ACL_ATM_IN`, 4 Nov); ATM lease works | None |
| DSW1 Gi1/0/21 management port (VLAN 99) | in macro (6 Nov, section 9b) | configured live (4 Nov) | None |
| Firewall-to-DSW links (fix #23) | routed `/30`s, ASA E1/4 + PA eth1/6/7 up, no PA `.78` / ASA `.86` | same (all 9 OSPF links FULL, 7 Nov) | None |
| DSW default route | none (OSPF default only) | OSPF default via own ASA link (`.97` / `.101`), 7 Nov | None |
| R1 `WAN-IN` `permit icmp any 203.149.210.184 0.0.0.7` | removed 7 Nov (redundant) | **removed live 9 Nov** (D3) | None. Keep `permit ip any 203.149.210.184 0.0.0.7` (line 50) — never remove it (fix #32) |
| R1/R2 `WAN-IN` SSH/Telnet denies (P6) | reference `.93` (router's own IP) | reference `.93` (fixed live by 7 Nov; confirmed 9 Nov) | None |
| SW1-SW4 NTP authentication | present (7 Nov) | applied (7 Nov) | None |
| PA DoS rule `DOS-DMZ-SERVERS` (`HARDENING.md` 6.3) | present (uncommented, corrected path) | committed live | None |
| ASA command authorization | on | on (both TACACS servers working) | None |
| R1 extra `0.0.0.0/0 .94 250` | absent | present (harmless) | None |
| ASA live `INSIDE_IN` | `DSW-SVIS` bootps line only | old `DSWs` bootps line also present (redundant) | Optional: `no access-list INSIDE_IN extended permit udp object-group DSWs object DHCP-SVR eq bootps` |
| Temporary ICMP permits on DSW `ACL_HR_IN` / `ACL_IT_IN` | **removed from `dsw1.ttl` / `dsw2.ttl`** (9 Nov) | **removed live 9 Nov** (D2, Stage 10) | None |

---

## 7. Testing

Full matrix, results and diagnosis steps are in `TEST-PLAN.md`. Summary (9 Nov):

| Stage | Status |
|-------|--------|
| 0 Infrastructure | All pass (0.18 NAT table confirmed 9 Nov) |
| 1-5 | Pass (6 Nov) |
| 6 Inbound | All pass; 6.5 (no loop: `.189` is 1:1 and dropped by the PA, `.187` hits Null0) and 6.9 confirmed 9 Nov |
| 7 OOB | Pass; 7.12 (P6) confirmed fixed 9 Nov |
| 8 Server outbound | Pass; 8.1/8.2 covered by 13A, 8.5/8.9 re-run after D1 (9 Nov). 8.3 (P3) not run |
| 9 Redundancy | Pass 9 Nov (9.4b slow; failback to ISP1 slow; R1 track 1 flaps) |
| 10 Hardening | 10.1-10.3 pass 9 Nov; 10.4/10.5 to confirm |
| 11 AAA | Pass 9 Nov (11.22, 11.26, 11.27 not run, optional) |
| 12 Logging | 12.11, 12.12, 12.15 (auth), 12.20-12.25 pass; 12.1 (R1) and 12.3 (ASA) partial; 12.2, 12.4-12.10, 12.13 not captured; 12.14 auto-block not achievable |
| 13 TXN / DNS / audit | Pass 9 Nov (13.14, 13.15, 13.23 not run) |
| 14 Attack readiness | 14.1, 14.2, 14.4 pass; 14.3 not run |

### Open problems (numbers match `TEST-PLAN.md` Part 2)

| # | Problem | Next step |
|---|---------|-----------|
| P19 | ~~VM102 couldn't reach R1 `.33`; NTP to `.44` reach 0~~ | **Resolved 4 Nov.** Ping passes on retry (earlier failure was during the LAN 4 / bond disturbance). NTP to `.44`: chronyd hadn't loaded the edited config (not listening on udp/123); fixed by restarting chrony and resetting R1's association. R1 now reaches `.44` at stratum 4. |
| P3 | Txn server can't reach DNS `.83` (`SERVER_IN` line 1 denies first) | Decide if Txn needs names; if so add a permit above line 1 |
| P6 | ~~R1 `WAN-IN` SSH/Telnet denies reference `.94`~~ | **Resolved.** `.93` in `r1.ttl` and live (confirmed 9 Nov, 7.12) |
| P11 | DHCP renewals from client IPs denied by ASA `INSIDE_IN` | Optional: permit udp from user subnets to `DHCP-SVR eq bootps` |
| P13 | ~~TACACS lockout risk once tac_plus answers~~ | **Resolved 7 Nov.** Both TACACS servers work; ASA command authorization re-enabled. Still keep a console open when rebuilding a device |
| P14 | ~~Clocks not synced until chrony runs on `.44/.45`~~ | **Resolved 7 Nov.** Authenticated NTP on R1, R2, DSWs, ASA, PA (`.44` `*`, `.45` candidate). SW1-SW4: authenticated too (7 Nov) |
| P15 | Internal servers have no DNS after the temporary rules go | Decide if needed |
| P17 | ~~Wazuh agents on internal servers can't reach `.84/.85`~~ | **Resolved 9 Nov.** ASA `SERVER_IN` line 1 + PA `WAZUH-AGENTS` (tcp/1514-1515 to `.84`), in both macros. 12.23 pass |
| P20 | ~~PA logs never reached `.44`/`.45` (no log-forwarding profile)~~ | **Resolved.** `LFP-WAZUH` committed and complete (12.11), now in `pa440.ttl` |

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
| 108 | Caching DNS (BIND, clone of VM101) | vmbr2 | 172.16.0.86/29 | .81 (PA) | 127.0.0.1 |

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

### 8.4 Temporary internet rules for package installs (REMOVED 9 Nov)

> **Status: all removed 9 Nov** (section 12 D1, `changes.md` Part 12.2), after the Wazuh agents and Suricata were installed. The ASA `SERVER_IN` www/https/domain lines, PA `TEMP-SVR-UPDATES`, and the two router `NAT-SOURCES` server lines (`172.16.0.72/0.0.0.7`, `172.16.0.80/0.0.0.7`) are gone. Internal/DMZ servers can no longer reach the internet (except VM105 in the TXN window and the jump hosts for NTP/DNS). `suricata-update` and `apt` on a server will fail from now on. The commands below are kept as the reference for **re-adding** them if another install is ever needed.

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
| 108 Caching DNS | BIND9 (recursive only) | HR/IT/DMZ resolver; caches answers; own name forwarded to VM101 | DSW `ACL_HR_IN`/`ACL_IT_IN` 100/105, ASA `INSIDE_IN` CACHE-DNS, PA `INTERNAL-TO-DNS`, `DNS-SVR-FORWARD`; R1/R2 `NAT-SOURCES` |
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

### 8.10b Caching DNS server (VM108, 172.16.0.86, BIND) - built 9 Nov

Full clone of VM101 (`qm clone 101 108 --name cachedns --full`), booted with `link_down=1` until netplan was changed to `.86/29`, gw `.81`, nameserver `127.0.0.1`. New hostname `cachedns`, machine-id and SSH host keys. The VM101 zone files and views were removed (backup in `/etc/bind.from-vm101`).

- `/etc/bind/named.conf`: default-zones include is active again (no views).
- `/etc/bind/named.conf.options`:
  ```
  acl trusted { localhost; 172.16.0.80/29; 172.16.0.56/29; 172.16.0.64/29; };
  options {
      directory "/var/cache/bind";
      listen-on { 127.0.0.1; 172.16.0.86; };
      listen-on-v6 { none; };
      recursion yes;
      allow-query       { trusted; };
      allow-recursion   { trusted; };
      allow-query-cache { trusted; };
      allow-transfer    { none; };
      forwarders { 172.18.4.32; 8.8.8.8; };
      forward first;
      dnssec-validation auto;
      validate-except { "sitict.net"; };
      max-cache-size 256m;
      max-ncache-ttl 300;
      version "not disclosed";
      minimal-responses yes;
  };
  ```
- `/etc/bind/named.conf.local`: `zone "gruppesechs.sitict.net" { type forward; forward only; forwarders { 172.16.0.83; }; };`. VM101 puts `.86` in its internal view (its `internal-users` ACL already covers 172.16.0.80/29), so the cache gets `172.16.0.82`. Zone edits are still made on VM101 only.
- Not NATed inbound, so it isn't reachable from the internet. `.86` to `.83` stays on vmbr2 (no firewall in the path).
- Rules added (live 9 Nov and in the macros): DSW1/DSW2 `ACL_HR_IN` and `ACL_IT_IN` seq 100/105 (udp/tcp 53 to `.86`), ASA object `CACHE-DNS` + `INSIDE_IN` lines 6-7, PA address `CACHE-DNS` in `INTERNAL-TO-DNS` (destination) and `DNS-SVR-FORWARD` (source), R1/R2 `NAT-SOURCES permit ip host 172.16.0.86 any`.
- **Saved-config note (9 Nov):** `write memory` was run by mistake on the ASA (saved the `CACHE-DNS` object only) and on DSW1/DSW2 (saved the four ACL lines plus the rest of the running config). This overwrote the previous lab setup's startup config on those three devices. R1, R2 and PA were not saved.
- **Pending:** DHCP (VM107) still hands out `172.16.0.83` only. To cut over: `option domain-name-servers 172.16.0.86, 172.16.0.83;` in the HR and IT subnets.

#### VM108 tests (all remote, via SSH)

Same IDs as `TEST-PLAN.md` Stage 15. Start with `ssh -p 2222 bigDonkBonkAdmin@203.149.210.185` (lands on VM102), then SSH on from there. Already passed on 9 Nov from HR: `nslookup google.com 172.16.0.86` and `nslookup gruppesechs.sitict.net 172.16.0.86` (ASA `INSIDE_IN` line 7 hitcnt 14).

**From home (outside):**

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 15.1 | `dig @203.149.210.190 gruppesechs.sitict.net +short` | `203.149.210.189` (VM101 external view unchanged) | Not run |
| 15.2 | `dig @203.149.210.190 google.com` | `REFUSED` / no answer (VM101 still not an open resolver) | Not run |
| 15.3 | `dig @203.149.210.185 google.com +time=2 +tries=1` | Timeout (the cache has no public mapping) | Not run |

**From VM102 (source .84, inside the cache's `trusted` ACL):**

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 15.4 | `dig @172.16.0.86 example.org \| grep -E "status\|Query time"`, run twice | `NOERROR` both times; the 2nd `Query time` ≈ 0 msec (served from cache) | Not run |
| 15.5 | `dig @172.16.0.86 wkg00n.sitict.net +short` | Another rack's IP (via ICT DNS) | Not run |
| 15.6 | `dig @172.16.0.86 gruppesechs.sitict.net +short` | `172.16.0.82` (forwarded to VM101's internal view) | Not run |
| 15.7 | `dig @172.16.0.86 nosuchname-4e.com \| grep status` | `NXDOMAIN` | Not run |
| 15.8 | `dig @172.16.0.86 isc.org +dnssec \| grep flags` and `dig @172.16.0.86 dnssec-failed.org \| grep status` | `ad` flag present; `SERVFAIL` for dnssec-failed.org (DNSSEC validation on). If isc.org itself SERVFAILs, the ICT forwarder strips DNSSEC: set `dnssec-validation no;` | Not run |
| 15.9 | `dig @172.16.0.86 version.bind chaos txt +short` | `"not disclosed"` | Not run |
| 15.10 | `dig @172.16.0.86 gruppesechs.sitict.net AXFR` | `Transfer failed` | Not run |

**On VM108 (`ssh ubuntu@172.16.0.86` from VM102):**

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 15.11 | `systemctl is-active named; systemctl is-enabled named` | `active`, `enabled` | Not run |
| 15.12 | `sudo rndc dumpdb -cache && sudo grep -ci example.org /var/cache/bind/named_dump.db` | > 0 (the name is in the cache) | Not run |
| 15.13 | `sudo journalctl -u named --since today \| grep -iE "error\|denied\|servfail"` | Nothing unexpected | Not run |
| 15.14 | Proxmox (`ssh root@172.16.0.43`): `qm config 108 \| grep -E "onboot\|net0"` | `onboot: 1`; net0 on `vmbr2` with no `link_down=1` | Not run |

**Firewalls (simulated HR/Staff traffic, from VM102 SSH into each device):**

| ID | Device | Test | Expected | Result |
|----|--------|------|----------|--------|
| 15.15 | ASA | `packet-tracer input inside2 udp 172.16.0.61 40000 172.16.0.86 53` | `Result: ALLOW` (INSIDE_IN CACHE-DNS) | Not run |
| 15.16 | ASA | `packet-tracer input inside udp 172.16.0.5 40000 172.16.0.86 53` (Staff) | `Result: DROP` | Not run |
| 15.17 | PA | `test security-policy-match from inside to dmz source 172.16.0.61 destination 172.16.0.86 protocol 17 destination-port 53 application dns` | `INTERNAL-TO-DNS` | Not run |
| 15.18 | PA | `test security-policy-match from dmz to outside source 172.16.0.86 destination 172.18.4.32 protocol 17 destination-port 53 application dns` | `DNS-SVR-FORWARD` | Not run |
| 15.19 | PA | `show log traffic src in 172.16.0.86` | Allowed dns sessions to 172.18.4.32 / 8.8.8.8 | Not run |
| 15.20 | R1 | `show ip nat translations \| include 172.16.0.86` (right after 15.4 with a new name) | Entries translated to `203.149.210.185` | Not run |
| 15.21 | DSW1/DSW2 | `show ip access-lists ACL_HR_IN \| include 172.16.0.86` | Match counters on seq 100 after an HR lookup | Not run |

### 8.11 Still to set up on VM102/106

| Item | Notes |
|------|-------|
| **chrony** (NTP for the OOB devices) | **DONE 4 Nov; NTP auth added 7 Nov.** Both VMs need `1 MD5 bigBoyNTPkey` in `/etc/chrony/chrony.keys` (owner `root:_chrony`, mode 640) and `keyfile /etc/chrony/chrony.keys` in `chrony.conf`; a missing key makes every device reject that server (fix #27). Check it's serving with `sudo ss -ulpn \| grep chronyd` (must show `0.0.0.0:123`), and `chronyc -N sources`. After editing the config, restart chrony and confirm port 123 is open; on a Cisco device, reset a stuck association with `no ntp server vrf OOB 172.16.0.44` then `ntp server vrf OOB 172.16.0.44 prefer`. Devices picking `.45` over `.44` is fine when VM102 syncs from VM106 (one stratum lower); add `noselect` to each VM's `server <other TACACS>` line to make both prefer the internet pool. Setup: **on both VM102 and VM106** (devices use `.44` prefer + `.45`). In `/etc/chrony/chrony.conf`: comment out the Ubuntu `pool ...` lines, add `pool 0.sg.pool.ntp.org iburst maxsources 4`, `server 172.16.0.45 iburst` (on VM106: `.44`), `allow 172.16.0.32/28`, `local stratum 10 orphan`; `sudo systemctl restart chrony`. Upstream NTP leaves via ens19 (PA `DMZ-OUTBOUND-NTP`, router `NAT-SOURCES`); devices query over ens18 (OOB). Tests: `TEST-PLAN.md` 7.13-7.16. Fixes P14. |
| **tac_plus** | **DONE (both VMs, 7 Nov).** Service is **`tac_plus-ng`** (not `tac_plus`); config `/etc/tac_plus-ng.cfg`; logs `/var/log/tac_plus-ng/authc.log`, `authz.log`, `acct.log`. Profiles: `manager` (permit all, priv 15) for `net_manager`; `engineer` (show/ping/traceroute/clear/interface/description/switchport/exit/end/logout/terminal, `configure terminal`, `(no) shutdown`; priv 7) for `net_engineers`. ASA command authorization is on live and in the macro. Test after changes with `test aaa group tac-group bigDonkBonkAdmin <pw> legacy` (IOS) or `test aaa-server authentication TACACS host 172.16.0.45 username ...` (ASA, checks VM106 directly). Validate config before restart: `sudo /usr/local/sbin/tac_plus-ng -P /etc/tac_plus-ng.cfg`. VM106 logs in UTC, VM102 in SGT: set VM106 to `Asia/Singapore` so the logs line up. |
| **SSH hardening + fail2ban** | **DONE (7 Nov, `HARDENING.md` 11).** `/etc/ssh/sshd_config.d/99-hardening.conf`; fail2ban `sshd` jail ignores `172.16.0.32/28` and `172.16.0.64/29`. Check: `sudo fail2ban-client status sshd`. |
| **Wazuh** | **DONE (9 Nov).** Manager 4.14.10 on VM102. Dashboard `https://172.16.0.44` (fixed: `opensearch.hosts` missing port; passwords reset; `wazuh.yml` API password). Syslog from devices: `<remote>` syslog udp/514, `allowed-ips 172.16.0.32/28` in `/var/ossec/etc/ossec.conf` (backup `ossec.conf.bak`). Suricata `eve.json` read as a `localfile` (`IDS-SURICATA.md`). Agents 4.14.10 on VM100, 101, 103, 104, 105, 107 (manager address `172.16.0.84`), all Active; **no agent on VM106** (decision). VM102 is the manager's own local agent `000`. P17 path: ASA `SERVER_IN` line 1 + PA `WAZUH-AGENTS` (both in the macros). `logall` deliberately **not** enabled (VM102 disk ~77%). Check: `sudo /var/ossec/bin/agent_control -l`. |
| **NetFlow analyzer** | Devices already export to `.44`/`.45` udp/2055 over the OOB VRF. |

---

## 9. Next steps

Superseded by **section 12**, the single ordered checklist for the next lab session. The old list is done or folded into it: the 5 Nov redesign is live, Stages 0-8 were run (6 Nov), chrony and tac_plus are done, and the hardening is in section 11.

---

## 10. Applying the 5 Nov firewall-link redesign to the live devices

> **Status 7 Nov: applied.** All 9 OSPF links of the new design are FULL. Keep this section only as a reference for how the live migration was done.

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
ip nat inside source static 172.16.0.82 203.149.210.189
ip nat inside source static udp 172.16.0.83 53 203.149.210.190 53 extendable
ip nat inside source static tcp 172.16.0.83 53 203.149.210.190 53 extendable
ip nat inside source static tcp 172.16.0.84 22 203.149.210.185 2222 extendable
```
If the last line is rejected because `.185` is also the PAT pool, change the pool to `.186` (both addresses in the `ip nat pool` line) and re-enter it. If the 8.4 temporary installs are still needed, add the two `NAT-SOURCES` lines from 8.4.

(9 Nov: the web mapping is **1:1** `ip nat inside source static 172.16.0.82 <public .189>` on both routers, replacing the old port-80/443 statics; fix #33. On a router that still has the port statics, remove them first: `no ip nat inside source static tcp 172.16.0.82 80 <.189> 80 extendable` and the same for 443.)

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
ip nat inside source static 172.16.0.82 129.126.142.189
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

---

## 11. Hardening (7 Nov)

Commands are in `HARDENING.md` (section numbers below refer to it). Device output from applying them is in `hardening-changes.md`. Everything marked "done" is live **and** in the macros, so a reloaded Cisco device or ASA gets it back from its `.ttl`. The keys are in section 4.

### 11.1 Status by section

| HARDENING.md | Status | Notes |
|---|---|---|
| 1.1 Login brute-force (`login block-for`) | **Dropped** | Quiet mode replaces `MGMT-ONLY` with an ACL that has no `vrf-also`, so it blocks **all** SSH over the OOB VRF, `.44` included. CoPP was tried as a replacement and also dropped (never applied live). Brute-force protection comes from fail2ban (11), ASA `max-fail` (5.3) and PA admin-lockout (6.5) |
| 1.2 Misc IOS hygiene | Skipped | Commands were rejected on the lab images |
| 1.3 NTP auth + NTP ACL | Done: for all devices | Key `bigBoyNTPkey` |
| 2.1 WAN-IN anti-spoofing | Done: R1, R2 | R1's redundant `permit icmp any 203.149.210.184 0.0.0.7` removed from `r1.ttl` and live (9 Nov). **Line 50 `permit ip any 203.149.210.184 0.0.0.7` must stay** (fix #32) |
| 2.2 uRPF, no unreachables, no CDP on WAN | Done: R1, R2 | |
| 2.3 Null0 for the public block | Done: R1, R2 | |
| 3.1 HSRP MD5 | Done: DSW1, DSW2 | |
| 3.3 Remove temporary ICMP permits | **Done 9 Nov** | Live on both DSWs and in `dsw1.ttl` / `dsw2.ttl`; Stage 10 passed |
| 4.1-4.4 Access switches (DHCP rate limit, errdisable recovery, DAI validate, storm control, no CDP) | Done: SW1-SW4 | |
| 4.5 IPSG mac-check | Not done (optional) | |
| 5.1 ASA MPF + threat detection | Done | `show service-policy interface inside` |
| 5.2 ASA uRPF on `server` | Skipped | |
| 5.3 ASA local lockout | Done | Command authorization on; console login authentication + exec auto-enable added 9 Nov (fixes #29, #30) |
| 5.4 ASA NTP auth | Done | See fix #28 before changing the key |
| 6.1, 6.2 PA zone protection (outside, dmz) | Done | Macro values: SYN 100/200/2000, scan threshold 100. During the 9 Nov scan tests the live `ZP-OUTSIDE` was tuned to SYN 500/1000/2000 and TCP Port Scan threshold 20; **restored to the macro values the same evening** (commit job 101). Recon auto-block could not be triggered (documented limitation, `changes.md` Part 15.2) |
| 6.3 PA DoS per-client limit | Done | DoS profiles are `set profiles dos-protection ...`,|
| 6.4 PA vlan.92 OSPF passive | Done | |
| 6.5 PA admin lockout + idle timeout | Done | |
| 6.6 PA NTP auth | Done | `show ntp`: `symmetric-key`, `synched` |
| 7 OSPF MD5 on all 9 links | Done | Key `GsOspf2026`, PA profile `OSPF-MD5` on eth1/3, eth1/6, eth1/7, vlan.10. DSW1 Gi1/0/12 had the wrong key until 9 Nov (fix #31) |
| 8 chrony keys | Done: VM102, VM106 | Fix #27 |
| 9 BIND (version, no AXFR, RRL) | Done: VM101 | |
| 10 nginx Slowloris limits | Done: VM100 | A raw TCP connection is closed after ~10 s (`client_header_timeout`): expected |
| 11 sshd hardening + fail2ban | Done: VM102, VM106 | |
| Syslog -> Wazuh | Done 9 Nov | Device syslog arrives at the Wazuh manager on `.44` (udp/514) and decodes (R1 `%SYS-5-CONFIG_I` -> rule 4721; PA -> `paloalto` decoder). Optional, not applied: `no logging message 302020` / `302021` on the ASA to drop the IP SLA ICMP build/teardown noise (~4 lines per 5 s) |
| Syslog (from the lab notes, not in `HARDENING.md`) | Done: all IOS devices + ASA | IOS: `no logging console`, `logging buffered 64000 informational`, `logging monitor informational`, `logging synchronous` on con 0 and vty 0 2. ASA: `logging buffered informational`, `logging monitor informational`. `logging trap` stays `informational`: level 4 would drop OSPF/HSRP changes and ACL `log` hits. Use `terminal monitor` per SSH session to watch logs live. Check: `show logging` |

### 11.2 Verification (run after any rebuild)

| Device | Command | Expected |
|---|---|---|
| R1, R2, DSW1, DSW2 | `show ip ospf neighbor` | R1: ASA `5.5.5.5` (G0/1/1), R2 + PA (G0/0/1). R2: same pattern. DSW1/2: peer DSW (Vlan100), PA (Gi1/0/12), ASA (Gi1/0/11). All FULL. **Count them: exactly 3 on each** — "all FULL" alone missed the missing PA-DSW1 adjacency until 9 Nov (fix #31) |
| same | `show ip ospf interface \| include line protocol\|authentication\|key` | `Cryptographic authentication enabled`, `Youngest key id is 1` on every OSPF link |
| ASA | `show ospf neighbor`, `show ospf interface \| include line protocol\|authentication\|key` | 5 neighbours FULL; auth on outside, wan-backup, wan-backup2, inside, inside2 |
| PA | `show routing protocol ospf neighbor`, `show config running \| match authentication` | **Exactly 5** neighbours full (R1, R2, ASA, DSW1 `.118`, DSW2 `.122`); `OSPF-MD5` on 4 interfaces. No `routed-OSPF-md5chksum-bad` lines in the system log / Wazuh |
| All Cisco + ASA | `show ntp associations detail` | `.44` `authenticated`, `sane`, `our_master`; `.45` `authenticated`, reach climbing to 377 |
| PA | `show ntp` | `synched`, `authentication-type: symmetric-key` (`autokey` = 6.6 not committed) |
| DSW1, DSW2 | `show standby brief` | DSW1 Active 23/46, DSW2 Active 61/67 |
| ASA / DSWs / PA | `show route 0.0.0.0` / `show ip route 0.0.0.0` / `show routing route type ospf` | ASA via `.105`, DSW1 via `.97`, DSW2 via `.101`, PA via `.90` (never a cost-500 bypass) |
| DSW2 (remote stand-in for an HR/IT PC) | `ping 8.8.8.8 source Vlan61`, `ping 8.8.8.8 source Vlan67`, `telnet 172.16.0.82 443 /source-interface Vlan61` | 5/5, 5/5, `Open` |

### 11.3 Lessons from applying it

- **OSPF auth, one link at a time:** do both ends within the 40 s dead timer and the adjacency never drops. A PA commit takes longer, so PA links drop for about a minute; do the cost-500 backups first so traffic has a path. Interface commands must be entered under `interface ...`, not at `(config)#`.
- **Commit pending PA changes before starting OSPF auth**, otherwise every commit pushes them too.
- **NTP:** a device that `prefer`s `.44` stays synced even when `.45` is broken, so always read the `.45` line as well. After a server comes back, `insane` / `reject` with low reach is normal for ~10 minutes.
- **ASA NTP:** `no ntp authentication-key 1 ...` deletes key 1 regardless of the value typed (fix #28).

### 11.4 Still to do (live)

**Nothing left from the hardening (9 Nov).** Both items are done live and in the macros:

1. ~~R1: remove the redundant ICMP line~~ **Done 9 Nov** (D3).
2. ~~3.3: remove the temporary ICMP permits on both DSWs~~ **Done 9 Nov** (D2, Stage 10 passed).

> **WARNING — R1 `WAN-IN` line 50.** `50 permit ip any 203.149.210.184 0.0.0.7` must **never** be removed. It is the line that lets return traffic (HR/IT internet) and inbound web/DNS/jump SSH reach the public block. Removing it by mistake on 9 Nov (`no 50`) cut all of that instantly (fix #32). When editing `WAN-IN`, remove lines by their exact text, not by sequence number. Correct live `WAN-IN` (R1, 9 Nov): lines 1-9 anti-spoofing; 10/20 deny tcp 22/23 to `172.17.9.93` log; 30 permit icmp echo-reply from `.94`; **50 permit ip any 203.149.210.184 0.0.0.7**; 60 permit tcp established to `.93`; 70 deny ip any any log.

---

## 12. Lab session checklist (continue here)

**State at the end of the 9 Nov lab session:** sections A-F are **done** (details in `changes.md`, Parts 1-16). Only H (demo prep) and the G decisions remain.

| Step | Status (9 Nov) |
|---|---|
| A Dashboard | Done. Fixed `opensearch.hosts` missing port; passwords reset; `wazuh.yml` API password |
| B Log sources | Done. Syslog into Wazuh (12.20, 12.1); PA log forwarding (12.11, 12.12, 12.15 auth); decoding (12.21) |
| C Agents | Done. 6 agents Active (12.22); agent path open (12.23); alerts (12.24, 12.25). No agent on VM106 |
| D Cleanup | Done. All temporary rules removed (D1); DSW ICMP permits removed (D2); R1 WAN-IN (D3) |
| E Tests | Done. Stage 11 AAA, Stage 13, Stage 9 redundancy, Stage 14 (bar the skipped L2 run 14.3) |
| F Macros | Done 9 Nov (this update): `pa440.ttl`, `pa440_cleanup.ttl`, `firepower1010.ttl`, `r1.ttl`, `r2.ttl`, `dsw1.ttl`, `dsw2.ttl` |
| G Decisions | P3/P15/P11 still open (low impact) |
| H Demo prep | Outstanding |

Test IDs refer to `TEST-PLAN.md`.

### A. Open the Wazuh dashboard
- [x] **A1** (done 9 Nov) In the lab, plug a laptop into **DSW1 Gi1/0/21** (VLAN 99), set `172.16.0.46/28`, gateway `172.16.0.33`, and browse `https://172.16.0.44` (accept the certificate warning). Don't use Proxmox LAN 4 (section 4).
  - From outside the lab, tunnel through the jump host: `ssh -p 2222 -L 8443:localhost:443 <user>@203.149.210.185`, then browse `https://localhost:8443`.
- [x] **A2** (done 9 Nov: the fault was `opensearch.hosts` missing its port) If the page doesn't load, on VM102: `sudo systemctl status wazuh-dashboard`, `sudo ss -tlnp | grep 443`, and check `server.host` in `/etc/wazuh-dashboard/opensearch_dashboards.yml` (it must be `0.0.0.0` or `172.16.0.44`, not `127.0.0.1`). Check the host firewall (`sudo ufw status`).
- [x] **A3** (done 9 Nov; passwords were reset and must be rotated/recorded privately) Get the admin password the installer generated: `sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt` (run where you ran the installer). Record it in your private notes, not in the repo.

### B. Wire the log sources into Wazuh
- [x] **B1** (done 9 Nov: `wazuh-remoted` on `0.0.0.0:514`; R1 and ASA lines captured on `.44`) On VM102, add this inside `<ossec_config>` in `/var/ossec/etc/ossec.conf`, then `sudo systemctl restart wazuh-manager` and check `sudo ss -ulpn | grep 514`:
  ```
  <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>udp</protocol>
    <allowed-ips>172.16.0.32/28</allowed-ips>
  </remote>
  ```
  The devices send to both `.44` and `.45`. Only the manager on `.44` is collected.
- [x] **B2** (done 9 Nov: already committed and complete, incl. both zone lines; now in `pa440.ttl`) PA: confirm and commit the log forwarding.
  ```
  run show config running | match LFP-WAZUH
  set zone outside network log-setting LFP-WAZUH
  set zone dmz network log-setting LFP-WAZUH
  validate full
  commit
  ```
  Expected in the first output: the profile, 11 security rules, the DoS rule and (after the zone lines) both zones. If a zone line is rejected, type `set zone outside network ?` or use Network > Zones > Log Setting in the web UI. Re-enter any rule line that is missing.
- [x] **B3** (PA part done 9 Nov; 12.1 R1 and 12.3 ASA partly; R2/DSWs/SWs, ACL deny logs and NetFlow 12.2, 12.4-12.10 not captured) Run `TEST-PLAN.md` 12.1-12.10 (syslog and NetFlow reach `.44` and `.45`) and 12.11-12.15 (PA logs). Careful: 12.14 blocks your source for 300 s.
- [x] **B4** (done 9 Nov, 12.21 pass) In the dashboard, confirm events from `.33`-`.42` and decoded PA lines (12.21).

### C. Install the agents
Keep the temporary internet rules (8.4) up until this step is finished: the agents download from the internet.

- [x] **C1** (done 9 Nov, commit job 81; 12.23 pass; now in both macros) Open the agent path (P17). The manager's address for servers is its DMZ IP `172.16.0.84`.
  ```
  (ASA, configure terminal) The permit must sit ABOVE the SERVERS -> INTERNAL deny (INTERNAL is 172.16.0.0/25 and includes the DMZ):
  access-list SERVER_IN line 1 extended permit tcp object SERVERS object JUMP-SVR range 1514 1515

  (PA, configure)
  set service WAZUH-AGENT protocol tcp port 1514-1515
  set rulebase security rules WAZUH-AGENTS from inside to dmz source SERVERS destination JUMP-SVR source-user any category any application any service WAZUH-AGENT action allow log-end yes log-setting LFP-WAZUH
  move rulebase security rules WAZUH-AGENTS before DENY-ALL
  validate full
  commit
  ```
  Test from VM103: `nc -zv 172.16.0.84 1514` and `1515`.
- [x] **C2** (done 9 Nov, 12.22 pass) Install agents on VM100, 101, 103, 104, 105, 107 (server address `172.16.0.84`). **Changed 9 Nov: no agent on VM106.** VM102 runs the manager's local agent. Install command used (per server, 4.14.10, agent enrols itself over 1515):
  ```
  wget -q https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.10-1_amd64.deb && sudo WAZUH_MANAGER='172.16.0.84' WAZUH_AGENT_NAME='<name>' dpkg -i ./wazuh-agent_4.14.10-1_amd64.deb && sudo systemctl daemon-reload && sudo systemctl enable --now wazuh-agent
  ```
  Agent names: `file` (103), `web` (100), `dns` (101), `accounts` (104), `txn` (105), `dhcp` (107). If an agent shows "Never connected", the service isn't running: `sudo systemctl daemon-reload && sudo systemctl enable --now wazuh-agent`.
- [x] **C3** (done 9 Nov: 12.24 rule 5760, 12.25 rule 4721) Run 12.20-12.25: `sudo /var/ossec/bin/agent_control -l` shows every agent Active, and a failed SSH login produces an alert.

### D. Clean up (after C2 is finished)
- [x] **D1** (done 9 Nov, after the Suricata install; `changes.md` Part 12.2) Removed all five temporary rules: ASA `SERVER_IN` www/https/domain lines; PA `TEMP-SVR-UPDATES`; R1/R2 `NAT-SOURCES` server lines; PA `TEMP-JUMP-TO-SERVERS` and ASA `OUTSIDE_IN` ssh permit `JUMP-SVR -> SERVERS` (the temporary SSH path used to install the agents, `changes.md` Part 5). 8.5, 8.9 and the Part 5.3 checks pass. **Follow-up 9 Nov evening:** the PA `TEMP-JUMP-UPDATES` rule from the Suricata install (`IDS-SURICATA.md` 3.3) was missed by D1 and was still live (index 12). **Deleted, commit job 99.** The PA rulebase now has no `TEMP-*` rules; `DENY-ALL` is the last custom rule.
- [x] **D2** (done 9 Nov) Removed the temporary ICMP permits on both DSWs, live and in the macros. Stage 10: 10.1-10.3 pass.
- [x] **D3** (done 9 Nov) R1 redundant ICMP line removed. P6 was already fixed live. Line 50 was removed by mistake and restored immediately (fix #32).

### E. Remaining tests (in this order)
- [x] **E1** (done 9 Nov) 0.18, 6.5, 6.9, 7.12 all pass.
- [x] **E2** (done 9 Nov) Stage 9 all pass. 9.4b (PA eth1/3 unplugged) is noticeably slow (brief asymmetry across the two firewalls while OSPF reconverges); failback to ISP1 is slow. R1 `show track 1` showed 87 changes: the IP SLA to ISP1 flaps (`changes.md` Part 14.1). Track `delay` dampening was **not adopted**.
- [x] **E3** (done 9 Nov) Stage 11A/11B pass; ASA fixes #29/#30 found here. 11.22, 11.26, 11.27 not run (optional).
- [x] **E4** (done 9 Nov) 13A (after D1), 13B, 13C pass. 13.14, 13.15, 13.23 not run (13.23 replaced by the 3.8 check).
- [x] **E5** (done 9 Nov) 14.1 SYN flood and 14.2 Slowloris pass with user-impact checks; 14.4 covered by fail2ban (12.24); 14.3 not run this session (L2 defences verified in Stage 1 / hardening). Recon auto-block not achievable (documented).

### F. Put what is now live into the macros
- [x] **F1** (done 9 Nov) `pa440.ttl`: `LFP-WAZUH` profile, match lists, every `log-setting` line (section 10b), `WAZUH-AGENT` service and `WAZUH-AGENTS` rule before `DENY-ALL`. `pa440_cleanup.ttl` now also deletes `WAZUH-AGENTS`, `WAZUH-AGENT`, `TEMP-JUMP-TO-SERVERS` and `TEMP-JUMP-UPDATES`.
- [x] **F2** (done 9 Nov) `firepower1010.ttl`: Wazuh permit in `SERVER_IN` above the deny; `aaa authentication serial console TACACS LOCAL`; `aaa authorization exec authentication-server auto-enable`.
- [x] **F2b** (done 9 Nov) `r1.ttl` / `r2.ttl`: 1:1 web NAT. `dsw1.ttl` / `dsw2.ttl`: temporary ICMP permits removed (Gi1/0/12 key in `dsw1.ttl` was already `GsOspf2026`).
- [x] **F3** (done 9 Nov) `TEST-PLAN.md` results filled in; sections 6 and 11 here updated.

### G. Decisions still open
- [ ] P3 / P15: the Txn server and internal servers have no DNS. Decide if they need it.
- [ ] P11: DHCP renewals from client IPs are denied by the ASA (low impact).
- [ ] Engineer `show running-config` on the ASA: the tac_plus `engineer` profile permits any `show ...`. Optional fix on **both** VM102 and VM106, inside `profile engineer` above the `show|ping|...` line: `if (cmd =~ /^show running-config/) deny`, then `sudo /usr/local/sbin/tac_plus-ng -P /etc/tac_plus-ng.cfg && sudo systemctl restart tac_plus-ng`. Not applied.
- [ ] Optional: ASA `no logging message 302020` / `302021` to cut the IP SLA ICMP log noise into Wazuh (add to `firepower1010.ttl` if applied).

### H. Before demo day
- [ ] Rotate the credentials exposed on 9 Nov (section "Where things stand", open item 2) and record them privately.
- [x] Restore PA `ZP-OUTSIDE` to the macro values (SYN 100/200/2000, TCP port-scan threshold 100). Done 9 Nov evening, commit job 101.
- [x] Remove the leftover PA `TEMP-JUMP-UPDATES` rule. Done 9 Nov evening, commit job 99.
- [ ] Housekeeping: rename VM102 `/etc/ssh/sshd_config.d/99-hardening-conf` -> `99-hardening.conf`; set VM106 timezone `Asia/Singapore`.
- [ ] After any Proxmox host reboot or VM102 Stop/Start, re-apply the Suricata `tc` mirror (`IDS-SURICATA.md` 3.1).
- [ ] Save a `show running-config` from every device to a file (no `write memory`; a reload wipes the Cisco devices and the ASA).
- [ ] Walk through `DEMO-DAY.md` section 3: the pre-flight table, then each scene once.
- [ ] On the day: set the final TXN window on both firewalls (`DEMO-DAY.md` section 1). **Never leave `TXN-WINDOW` empty** (an empty ASA time-range may count as always active).