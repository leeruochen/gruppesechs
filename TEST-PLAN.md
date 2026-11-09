# Punggol Bank: Network Connectivity Test Plan

**Last updated:** 9 November 2026 (Stages 9-14 run in the 9 Nov lab session; details and device output in `changes.md`)

Purpose: confirm every connection `plan.md` requires actually works, and every connection it forbids is actually blocked.

How to use it:
- Work through the stages in order. A later stage depends on the earlier ones.
- Every row has an **Expected** result. A deny test passes when the connection **fails**.
- Fill in the **Result** column (`PASS` / `FAIL` + date + note).
- When a row fails, run the device-side check for it. That shows which device dropped the traffic.

**Status 9 Nov:** all stages have been run. Stages 9-14 were done in the 9 Nov session after the temporary rules were removed (D1), so they were tested on the final rules. Rows not run are marked as such. The plan below is kept for reference.

**Plan from 7 Nov.** Stages 0-8 are done (results below, 6 Nov). The hardening from `HARDENING.md` is applied (`HANDOVER.md` section 11), so re-check 6.5 (Null0 route) and 7.12 (`WAN-IN`) once, and run 8.1-8.3 as part of Stage 13. Order of the remaining work:

1. Install Wazuh and wire the log sources. Stage 12B has the PA log-forwarding config, which is missing from the macros (P20).
2. Stage 10 (remove the temporary rules and ICMP permits). Do this **before** Stage 9, so redundancy is tested on the final rules.
3. Stage 9 (redundancy). Unplug tests only, no power-off tests.
4. Stages 11-13: AAA, logging and monitoring, time window and exposure checks.
5. Stage 14 (attack readiness), then the attacks in `DEMO-DAY.md` section 3.

Not planned: a macro rebuild rehearsal. The hardening was applied live and the macros were edited to match, but the edited macros have not been run.

**Status at a glance (6 Nov):** the 5 Nov redesign (routed firewall links, second ISP, NAT on R1/R2) is live and Stages 0 to 7 pass. Stages 1 to 5 were reported as working in full, so their rows are marked "(reported)". Stage 6 passes (6.9 not run), and the internal DNS view is live (HR gets `172.16.0.82`). Stage 7 passes: SSH to every device, NTP on the devices checked, and the MGMT-PC deny. **Open:** the R1 `WAN-IN` fix (P6) on the live router, the possible loop for unused public IPs on R1 (6.5), Stage 8 (8.1 to 8.3 not run; the temporary install rules need re-adding for Wazuh), Stages 9 and 10, and the Txn DNS decision (P3, P15). The final TXN-WINDOW is set on demo day: see `DEMO-DAY.md`.

---

## Part 1: Preparation

### 1.1 Test hosts

All user VLANs now get addresses from DHCP (VM107, `172.16.0.77`). DNS (`172.16.0.83`) is handed out to HR and IT only.

| Name | Plug into | Lease range | Gateway (VIP) | DNS from DHCP |
|------|-----------|-------------|---------------|---------------|
| HR-PC | SW3 Gi1/0/5-6 | 172.16.0.61 - .62 (.60 reserved) | 172.16.0.57 | 172.16.0.83 |
| IT-PC | SW4 Gi1/0/5-7 | 172.16.0.68 - .70 | 172.16.0.65 | 172.16.0.83 |
| STAFF-PC | SW1 Gi1/0/5-16 | 172.16.0.4 - .30 | 172.16.0.1 | none |
| ATM-PC | SW2 Gi1/0/5-7 | 172.16.0.52 - .54 | 172.16.0.49 | none |
| OUTSIDE-PC | Personal laptop on the lab Wi-Fi, or a host on another rack | their own | their own | lab resolver `172.18.18.36` |
| MGMT-PC | DSW1 Gi1/0/21 (VLAN 99), static `172.16.0.46/28`, gw `.33` | | | |

Check a lease with `ipconfig`, and on the access switch with `show ip dhcp snooping binding`.

### 1.2 Access-switch notes (otherwise tests fail for the wrong reason)

1. **DAI and IP Source Guard:** DHCP clients are bound automatically. Only a host with a **static** IP needs a manual `arp access-list` + `ip source binding` (see the commented OPTIONAL block in `sw3.ttl`). Never leave a static binding for a host that uses DHCP: it blocks the lease binding (P12).
2. **Port security (changed 9 Nov):** every access port allows 5 MACs, learned **dynamically** (no sticky), aged out after 5 min of inactivity and cleared on link-down. Violation mode is `restrict`: frames from a 6th MAC are dropped and logged, and the port stays up. Moving a laptop between ports needs no clearing. Check with `show port-security interface Gi1/0/<n>` (violation count) and `show logging | include PSECURE`.
3. **Windows firewall blocks inbound ping** on target laptops. For inter-VLAN ping tests:
   ```
   netsh advfirewall firewall add rule name="ICMPv4-in" protocol=icmpv4:8,any dir=in action=allow
   ```

### 1.3 Services on the servers (what each test talks to)

| VM | Service | Port(s) | Notes |
|----|---------|---------|-------|
| 103 File `.74` | Samba share `HR` | tcp/445 | Users `hrmanager`, `hradmin` (group `hrshare`); `hosts allow` = HR subnet |
| 104 Accounts `.75` | Nginx portal, basic auth | tcp/443 (80 redirects) | Users `staff1`, `atm1`; nginx `allow` = Staff + ATM |
| 105 Txn `.76` | (same pattern as Accounts if built that way) | tcp/443 | Confirm what's running before testing 4.8/4.9 |
| 107 DHCP `.77` | isc-dhcp-server | udp/67 | |
| 100 Web `.82` | Nginx, hostname GruppeSechs | tcp/80, tcp/443 | Self-signed certificate |
| 101 DNS `.83` | BIND (authoritative for `gruppesechs.sitict.net`, recursive for HR/IT) | udp+tcp/53 | Forwarders: `172.18.4.32` (ICT), then 8.8.8.8 |
| 102/106 Jump `.84/.85` | OpenSSH | tcp/22 | OOB side `.44/.45` |

If a service isn't running yet, a placeholder listener makes TCP tests meaningful: `sudo nc -lk <port>` or `sudo python3 -m http.server <port>`. Stop any placeholder before starting the real service (an old listener blocked nginx on 4 Nov).

### 1.4 Client-side commands

| Goal | Windows | Linux |
|------|---------|-------|
| ICMP | `ping <ip>` | `ping -c 4 <ip>` |
| TCP port | `Test-NetConnection <ip> -Port <port>` (`TcpTestSucceeded`) | `nc -zv -w 3 <ip> <port>` |
| Path | `tracert -d <ip>` | `traceroute -n <ip>` |
| Web | `curl.exe -m 5 http://<ip>` / `curl.exe -k -m 5 https://<ip>` | `curl -m 5 ...` |
| DNS | `nslookup <name> <server>` | `dig @<server> <name> +short` |

`Test-NetConnection` is slow when the laptop's DNS server isn't answering, because it does a reverse lookup first. That's not a connectivity failure.

### 1.5 Device-side checks

- **ASA:** `packet-tracer input <ingress-if> <tcp|udp|icmp> <src> <sport|8> <dport|0> <dst> <dport> detailed`. Read **Action** and the **egress interface**.
  - `drop (no-v4-adjacency)` = the ACL passed but the destination isn't answering ARP (VM down or unplugged).
  - UDP/53 traces always end in `inspect-dns-invalid-pak`: packet-tracer sends no DNS payload. Look at the egress interface instead.
  - Captures: `capture <name> interface <if> match <proto> <src> <dst>`, `show capture <name>`, `no capture <name>`.
- **PA:** `test security-policy-match from <zone> to <zone> source <ip> destination <ip> protocol <6|17|1> destination-port <port> application <app>`. Use the base App-ID name (e.g. `dns-base`, not `dns`). (Since 5 Nov the PA does no NAT; check NAT on R1/R2 with `show ip nat translations`.)
  - Sessions: `show session all filter source <ip>` / `destination <ip>`, then `show session id <id>` (check `s2c` bytes > 0).
  - **Inbound** sessions: before 5 Nov listed under the public destination IP; with NAT on the routers, under the real server IP (`.82`, `.83`, `.84`).
  - Ping from the data plane: `ping source <dataplane-ip> host <ip>`. Plain `ping host` uses the management port.
- **DSW:** `show access-lists ACL_<VLAN>_IN`. Counters rise as traffic matches. Deny lines also log (`%SEC-6-IPACCESSLOGP`).

---

## Part 2: Predicted failures and their status

Found by reading the configs against `plan.md`. Each links to the test that exposes it.

| # | Problem | Status | Exposed by |
|---|---------|--------|-----------|
| P1 | ASA delivered HR/IT -> DMZ traffic out its own `dmz` interface (Vlan92), skipping the PA; replies via the PA were dropped | **FIXED 4 Nov** (ASA Vlan92 shut down). 5 Nov: Vlan92 removed entirely | 5.4, 5.6 |
| P2 | ASA only permitted DHCP relay from `.98/.99` | **FIXED 4 Nov** (`DSW-SVIS` object-group in `INSIDE_IN` / `SERVER_IN`) | 4.10 |
| P3 | Txn server can't use DNS `.83`: `SERVER_IN` line 1 denies SERVERS -> INTERNAL first | Open | 8.3 |
| P4 | No NAT for inbound HTTPS to the web server | **FIXED 4 Nov** (`NAT-WEB-HTTPS`) | 6.2 |
| P5 | Servers had no internet for `apt install`; DNS server couldn't reach forwarders | **Closed 9 Nov.** DNS forwarders permanent (`DNS-SVR-FORWARD`, incl. ICT DNS). The temporary install rules were re-added for Wazuh/Suricata and **removed 9 Nov** (D1); 8.5 and 8.9 re-run and pass | 8.4, 8.9 |
| P6 | R1 `WAN-IN` SSH/Telnet denies reference `.94` (ISP), not `.93` (R1) | **FIXED** in `r1.ttl` and live; confirmed 9 Nov (7.12) | 6.7, 7.12 |
| P7 | HR <-> IT ping works while the temporary ICMP permits exist | **Closed 9 Nov:** permits removed; 10.1 HR <-> IT ping now fails | 3.5, 10.1 |
| P8 | An ASA failure is a full outage (no automatic bypass) | **Redesigned 5 Nov; verified 9 Nov** (9.4, 9.4a pass: users fail over to the PA bypass automatically). Note the PA-DSW1 bypass was down 7-9 Nov due to a wrong OSPF key (HANDOVER fix #31) | 9.4 |
| P9 | PA treated `172.16.0.72/29` as connected on dead vlan.69 | **FIXED 4 Nov** (vlan.69 removed from VR and OSPF) | 8.2 |
| P10 | `ACL_STAFF_IN` / `ACL_ATM_IN` dropped the peer DSW's HSRP hellos; both DSWs Active for VLANs 23/46 | **FIXED 4 Nov** (`permit udp <subnet> host 224.0.0.102 eq 1985`). **Confirmed 6 Nov** (0.3 passes) | 0.3 |
| P11 | DHCP renewals (unicast from client IPs) denied by ASA `INSIDE_IN` | Open (clients fall back to broadcast; slower renewals only) | 1B |
| P12 | Static HR binding blocked the DHCP lease for the same MAC | **FIXED 4 Nov** (static block removed; OPTIONAL in `sw3.ttl`) | 1.1 |
| P13 | Lockout risk once tac_plus answers (no fallback to local on a "fail") | **Resolved.** 9 Nov: the ASA console locked up (session ran as `enable_15`, TACACS denied everything incl. `exit`); fixed with console login authentication (HANDOVER fix #29). 11B fallback tested and passes | 7.4, 11.20-11.24 |
| P14 | ASA/PA clocks not synced until chrony runs on `.44/.45` | **Fixed.** All devices synced and authenticated (SW1-SW4 authenticated 7 Nov) | 8.1, 8.2 |
| P15 | Internal servers have no DNS once the temporary rules are removed | Open (decide if any internal server needs DNS) | 8.3 |
| P16 | Jump SSH (`.185:2222`) might not match `JUMP-SSH-IN` after port translation | **Not an issue** (6.4 passed) | 6.4 |
| P17 | Wazuh/syslog agents on internal servers can't reach `.84/.85` | **FIXED 9 Nov**: ASA `SERVER_IN` line 1 + PA `WAZUH-AGENTS` (tcp/1514-1515 to `.84`), in both macros | 12.23 |
| P18 | DHCP replies addressed to the peer DSW's SVI crossed the user VLAN and were denied by the inbound user ACL (`denied udp 172.16.0.77(67) -> 172.16.0.51(67)`) | **FIXED 4 Nov** in macros (`permit udp host 172.16.0.77 eq bootps any eq bootps`). Confirm with 1.5 | 1.5 |
| P19 | VM102 (`.44`) couldn't reach R1 `.33`; R1/R2 NTP to `.44` showed `reach 0` | **RESOLVED 4 Nov.** Two separate causes: (1) the ping failure was temporary, almost certainly while Proxmox LAN 4 (the bond's primary) was unplugged / on a laptop; it passed on retry with ARP replies seen on `bond0`. (2) NTP to `.44`: the config was right, but the running chronyd hadn't loaded it, so it wasn't listening on udp/123 (`ss` showed only `127.0.0.1:323`) and was still using the Ubuntu pools. Fixed by restarting chrony and resetting R1's association. R1 now reaches `.44` (stratum 4). | 7.1, 7.14 |
| P20 | The PA has the syslog server profile `SYSLOG-JUMP` but no log-forwarding profile and no system/config match lists, so its traffic, threat, DoS, zone-protection and system logs never reach `.44`/`.45` (and so never reach Wazuh) | **FIXED**: `LFP-WAZUH` was committed and complete (12.11, 9 Nov); now in `pa440.ttl` section 10b | 12.11-12.15 |
| P21 | DSW1 Gi1/0/12 OSPF key `Cisco123!` instead of `GsOspf2026`: the PA-DSW1 bypass adjacency was down since 7 Nov | **FIXED 9 Nov** (found from the PA's forwarded `routed-OSPF-md5chksum-bad` log; DSW1 re-keyed) | 0.6a, 0.9, 9.7 |
| P22 | Port-only web NAT on R1/R2 sank scans to Null0, so the PA never logged or blocked them | **FIXED 9 Nov** (1:1 web NAT on R1/R2) | 12.14, 6.5 |

---

## Stage 0: Infrastructure health (CLI only)

| ID | Device | Command | Expected | Result |
|----|--------|---------|----------|--------|
| 0.1 | DSW1, DSW2 | `show etherchannel summary` | Po1-Po5 `(SU)`, members `(P)`. **Po5 must be up.** | PASS 6 Nov (Po5 Gi1/0/10 had a loose cable, reseated) |
| 0.2 | SW1-SW4 | `show etherchannel summary` | Both uplinks `(SU)` | PASS 6 Nov (SW1 seen, SW2-4 reported) |
| 0.3 | DSW1, DSW2 | `show standby brief` | 23/46: DSW1 Active, DSW2 Standby. 61/67: DSW2 Active, DSW1 Standby. No `unknown` standby. | PASS 6 Nov (DSW1 Active 23/46, DSW2 Active 61/67, real Standby on both; P10 fix confirmed) |
| 0.4 | DSW1, DSW2 | `show ip route 0.0.0.0` | DSW1 `O*E1` via `172.16.0.97`, DSW2 via `172.16.0.101` (OSPF default from R1; no static route since 5 Nov) | PASS 6 Nov (OSPF default via .97 on DSW1 and .101 on DSW2, metric 40, no static) |
| 0.5 | DSW1, DSW2 | `show track 10` | Up on both (required for HSRP and route failover in the 5 Nov design) | PASS 6 Nov (Up on both) |
| 0.6 | ASA | `show ospf neighbor` | R1 `.109`, R2 `.113`, PA `.105`, DSW1 `.98`, DSW2 `.102`, all FULL | PASS 6 Nov (R1 .109, R2 .113, PA .105, DSW1 .98, DSW2 .102 all FULL) |
| 0.6a | DSW1, DSW2 | `show ip ospf neighbor` | ASA (`.97` / `.101`), PA (`.117` / `.121`), other DSW over Vlan100 (`.126` / `.125`), all FULL | PASS 6 Nov (DSW1: .97, .117, .126; DSW2: .101, .121, .125; all FULL) |
| 0.6b | ASA | `show route 172.16.0.56` and `show route 172.16.0.0` | HR via `.102` (DSW2), Staff via `.98` (DSW1) | PASS 6 Nov (HR/IT via .102, Staff/ATM via .98) |
| 0.7 | ASA | `show route` | `0.0.0.0/0 via 172.16.0.105`; `172.16.0.80/29 via 172.16.0.105` (not connected `dmz`) | PASS 6 Nov |
| 0.8 | ASA | `show nat` | Empty (no `obj_any`, no standby PAT after 5 Nov) | PASS 6 Nov (only built-in nlp_ entries) |
| 0.9 | PA | `show routing protocol ospf neighbor` | R1 `.90`, R2 `.91`, ASA `.106`, DSW1 `.118`, DSW2 `.122` full | PASS 6 Nov (R1 .90, R2 .91, ASA .106, DSW1 .118, DSW2 .122 all full) |
| 0.10 | PA | `show routing route` | `0.0.0.0/0 via .90` learned by OSPF (no static `DEFAULT-OUT` after 5 Nov); `172.16.0.72/29 via .106` (O2); HR/IT via `.106`, **not** via `.118`/`.122` | PASS 6 Nov (default via .90 O1; HR/IT via .106; .72/29 via .106) |
| 0.11 | PA | `show interface ethernet1/6`, `1/7` | Link state **up** (5 Nov design), IPs `.117/30`, `.121/30` | PASS 6 Nov (eth1/6 .117/30 and eth1/7 .121/30 up) |
| 0.12 | PA | `show running security-policy` | `DENY-ALL` last; `TEMP-SVR-UPDATES` present only while installs are ongoing | PASS 6 Nov (DENY-ALL last, no TEMP-SVR-UPDATES; re-add it for installs) |
| 0.13 | R1 | `show ip route 0.0.0.0` | Only `via 172.17.9.94` | PASS 6 Nov |
| 0.14 | R1 | `show track` | Track 1 Up (track 2 removed 5 Nov) | PASS 6 Nov |
| 0.15 | R1 | `show ip ospf neighbor` | PA `.92` + R2 `.91` on G0/0/1, ASA `.110` on G0/1/1, all FULL | PASS 6 Nov (PA .92 + R2 .91 on G0/0/1, ASA .110 on G0/1/1) |
| 0.16 | R1 | `ping 172.17.9.94` | 5/5 | PASS 6 Nov |
| 0.17 | R2 | `show track 1`; `ping 172.17.10.94`; `show ip ospf neighbor` | Track Up; 5/5; PA `.92` + R1 `.90` on G0/0/1, ASA `.114` on G0/1/1 | PASS 6 Nov (PA .92 + R1 .90 on G0/0/1, ASA .114 on G0/1/1) |
| 0.18 | R1, R2 | `show ip nat translations` (with HR browsing) | R1: HR `.6x` -> `203.149.210.185`, plus the static `.189`/`.190`/`.185:2222` entries. R2: only static entries while R1 is up | PASS 9 Nov (R1: HR `172.16.0.61` -> `203.149.210.185`, plus the static entries; `.189` is now a 1:1 entry with no ports) |
| 0.19 | PA, ASA, DSW | `show ip route 0.0.0.0` / `show routing route destination 0.0.0.0/0` | Default via R1 (metric ~10 + path), not R2 (50 + path) | PASS 6 Nov (default learned from R1 on DSW1, DSW2, ASA and PA) |

---

## Stage 1: Each VLAN gets an address and reaches its gateway

| ID | From | Test | Expected | Note | Result |
|----|------|------|----------|------|--------|
| 1.1 | HR-PC | `ipconfig`, `ping 172.16.0.57` | Lease in `.61-.62`, ping replies | Ping works only because of the temporary ICMP permit | PASS 4 Nov (`.61`); PASS 6 Nov (reported) |
| 1.2 | IT-PC | `ipconfig`, `ping 172.16.0.65` | Lease in `.68-.70`, ping replies | | PASS 4 Nov (`.68` seen); PASS 6 Nov (reported) |
| 1.3 | STAFF-PC | `ipconfig`; `ping 172.16.0.1`; `arp -a` | Lease in `.4-.30`; **ping fails**; gateway MAC in ARP table; no DNS server | `ACL_STAFF_IN` ends in `deny ip any any` | Lease PASS 4 Nov (`.4`); PASS 6 Nov (reported) |
| 1.4 | STAFF-PC | (DSW logs) | No `ACL_STAFF_IN denied ... -> 172.16.0.83(53)` | Confirms DNS isn't handed to Staff | PASS 4 Nov (after DHCP change); PASS 6 Nov (reported) |
| 1.5 | ATM-PC | `ipconfig /renew`; `arp -a` | Lease in `.52-.54`; gateway `.49` MAC in ARP table | Needs the P18 fix live on both DSWs | PASS 4 Nov (`.52`; fixes #18/#19 confirmed on DSW1 and DSW2); PASS 6 Nov (reported) |
| 1.6 | Each switch | `show ip dhcp snooping binding`; `show ip arp inspection statistics vlan <N>` | Laptop listed; DAI drop count flat | | PASS 6 Nov (reported) |

---

## Stage 2: Internet access (HR and IT yes, Staff and ATM no)

| ID | From | Test | Expected | If it fails, check | Result |
|----|------|------|----------|--------------------|--------|
| 2.1 | HR-PC | `ping 8.8.8.8` | Reply | PA `show session all filter source 172.16.0.61` | PASS 4 Nov; PASS 6 Nov (reported) |
| 2.2 | HR-PC | `nslookup google.com` (uses `.83`) | Answer | 5.12a | PASS 4 Nov; PASS 6 Nov (reported) |
| 2.3 | HR-PC | Browse google.com, twitch.tv | Loads | PA `HR-IT-INTERNET` | PASS 4 Nov; PASS 6 Nov (reported) |
| 2.4 | HR-PC | `tracert -d 8.8.8.8` | `.57` -> `.97` -> `.105` -> `.90` -> `172.17.9.94` -> ... | Proves the serial path | PASS 6 Nov (reported) |
| 2.5 | R1 | `show ip nat translations \| include 172.16.0.6` while HR browses | HR translated to `203.149.210.185` (PA NAT before 5 Nov) | | PASS 6 Nov (reported) |
| 2.6 | IT-PC | Repeat 2.1-2.3 | Same as HR | | PASS 6 Nov (reported) |
| 2.7 | STAFF-PC | `ping 8.8.8.8`; `Test-NetConnection 8.8.8.8 -Port 443` | **Both fail** | DSW `ACL_STAFF_IN` deny counter | PASS 6 Nov (reported) |
| 2.8 | ATM-PC | Same as 2.7 | **Both fail** | `ACL_ATM_IN` | PASS 6 Nov (reported) |
| 2.9 | ASA | `packet-tracer input inside tcp 172.16.0.4 12345 8.8.8.8 443` | **drop** at `deny ip object-group STAFF-ATM any` | Second layer | PASS 6 Nov (reported) |
| 2.10 | R1 | `ping 8.8.8.8` | **Fails (expected)**: R1 sources from the WAN `/30` | | PASS 4 Nov (fails as expected); PASS 6 Nov (reported) |

---

## Stage 3: VLAN isolation (no VLAN reaches another)

Targets need inbound ICMP allowed (1.2 step 3) and a listener for TCP rows (e.g. `python -m http.server 8080`).

| ID | From -> To | Test | Expected | Result |
|----|-----------|------|----------|--------|
| 3.1 | HR -> Staff | ping, TCP 8080 | **Both fail** (reply dropped by `ACL_STAFF_IN`) | PASS 6 Nov (reported) |
| 3.2 | HR -> ATM | ping, TCP | **Both fail** | PASS 6 Nov (reported) |
| 3.3 | Staff -> HR | ping, TCP | **Both fail** | PASS 6 Nov (reported) |
| 3.4 | ATM -> Staff | ping, TCP | **Both fail** | PASS 6 Nov (reported) |
| 3.5 | HR <-> IT | ping | **Succeeds during testing (P7)**; must fail after Stage 10 | PASS 6 Nov (reported; ping works until Stage 10) |
| 3.6 | HR -> IT | TCP 8080 | **Fails** | PASS 6 Nov (reported) |
| 3.7 | IT -> HR | TCP 8080 | **Fails** | PASS 6 Nov (reported) |
| 3.8 | HR-PC | `ping 172.16.0.35`; `Test-NetConnection 172.16.0.35 -Port 22` | **Both fail** (OOB is in VRF OOB) | PASS 6 Nov (reported) |

---

## Stage 4: Internal servers (VLAN 69, behind the ASA)

### 4A: Baseline

| ID | Where | Test | Expected | Result |
|----|-------|------|----------|--------|
| 4.1 | ASA | `ping server 172.16.0.74` (and `.75`, `.76`, `.77`) | Reply | PASS 6 Nov (reported) |
| 4.2 | Each VM | `ip route` | One default, via `172.16.0.73` | PASS 6 Nov (reported) |
| 4.3 | Each VM | `ping -c 4 172.16.0.73` | Reply | PASS 6 Nov (reported) |

### 4B: Access matrix (File = HR only; Accounts and Txn = Staff and ATM only)

| ID | From -> To | Test | Expected | Enforced at | Result |
|----|-----------|------|----------|-------------|--------|
| 4.4 | HR -> File `.74` | `ping 172.16.0.74` | Reply | DSW + ASA ICMP permits | PASS 6 Nov (reported) |
| 4.5 | HR -> File `.74` | `net use H: \\172.16.0.74\HR /user:hrmanager`, save a file, check `sudo ls -la /srv/samba/hr` on VM103 | Works, file owned by hrmanager:hrshare | DSW, ASA `permit tcp HR FILE-SVR eq 445` | PASS 4 Nov; PASS 6 Nov (reported) |
| 4.6 | IT -> File `.74` | ping, `Test-NetConnection -Port 445` | **Both fail** (ICMP at ASA, TCP at DSW) | | PASS 6 Nov (reported) |
| 4.7 | Staff -> File `.74` | `Test-NetConnection -Port 445` | **Fails** | DSW | PASS 6 Nov (reported) |
| 4.8 | Staff -> Accounts `.75` | Browser `https://172.16.0.75` (login staff1); `Test-NetConnection -Port 443` | Page loads; True | DSW, ASA `permit ip STAFF-ATM ACCT-SVR` | PASS 4 Nov; PASS 6 Nov (reported) |
| 4.8a | Staff -> Txn `.76` | Same, against `.76` | Loads | ASA `permit ip STAFF-ATM TXN-SVR` | PASS 6 Nov (reported) |
| 4.9 | ATM -> Accounts `.75`, Txn `.76` | Same (login atm1) | Loads | | PASS 6 Nov (reported) |
| 4.10 | DHCP relay | Leases on each VLAN (Stage 1) | Leases issued | ASA `DSW-SVIS` permits | PASS 4 Nov (HR, IT, Staff); PASS 6 Nov (reported) |
| 4.11 | HR -> Accounts `.75`, Txn `.76` | ping, `Test-NetConnection -Port 443` | **Both fail** | ICMP at ASA, TCP at DSW | PASS 6 Nov (reported) |
| 4.11a | HR -> DHCP `.77` | ping | **Fails** (leases still work via relay) | ASA | PASS 6 Nov (reported) |
| 4.11b | HR -> ASA `.73` | ping | **Fails** (ASA only answers pings on the ingress interface `.97`) | | PASS 6 Nov (reported) |
| 4.12 | IT -> Accounts / Txn | ping, TCP 443 | **Both fail** | | PASS 6 Nov (reported) |
| 4.13 | Staff -> `.75` on another port (e.g. 22) | `Test-NetConnection -Port 22` | Reaches the server (plan.md allows Staff/ATM `ip` to these servers) | | PASS 6 Nov (reported) |
| 4.14 | Server -> server | VM103: `nc -zv 172.16.0.75 443` | **Pass** (same subnet, no firewall in between) | | PASS 6 Nov (reported) |
| 4.15 | Server -> user VLAN | VM103: `ping 172.16.0.61` | **Fails** (`SERVER_IN` denies SERVERS -> INTERNAL) | | PASS 6 Nov (reported) |

---

## Stage 5: DMZ servers (VLAN 92, behind the PA)

### 5A: Baseline

| ID | Where | Test | Expected | Result |
|----|-------|------|----------|--------|
| 5.1 | PA | `ping source 172.16.0.81 host 172.16.0.82` (and `.83`, `.84`, `.85`) | Reply | PASS 6 Nov (reported) |
| 5.2 | Each DMZ VM | `ip route` | One default, via `172.16.0.81` (jump hosts: on `ens19` only) | PASS 6 Nov (reported) |
| 5.3 | ASA | `show route 172.16.0.80` | Route via `.105` (no Vlan92 after 5 Nov) | PASS 4 Nov; PASS 6 Nov (reported) |

### 5B: Access matrix (HR/IT reach Web and DNS; IT reaches the jump hosts)

| ID | From -> To | Test | Expected | Result |
|----|-----------|------|----------|--------|
| 5.4 | ASA | `packet-tracer input inside tcp 172.16.0.61 12345 172.16.0.82 443 detailed` | Allow, egress **`outside`** | PASS 6 Nov (reported) |
| 5.5 | PA | `test security-policy-match from inside to dmz source 172.16.0.61 destination 172.16.0.82 protocol 6 destination-port 443 application ssl` | `INTERNAL-TO-WEB` allow | PASS 6 Nov (reported) |
| 5.6 | HR -> Web `.82` | Browser `http://172.16.0.82` and `https://172.16.0.82` | Page loads | PASS 4 Nov (HR and IT HTTPS sessions seen on the PA); PASS 6 Nov (reported) |
| 5.7 | HR -> DNS `.83` | `nslookup google.com 172.16.0.83` | Answer | PASS 4 Nov; PASS 6 Nov (reported) |
| 5.8 | IT -> Web, DNS | Repeat 5.6, 5.7 | Pass | PASS 4 Nov (IT reached the site by name); PASS 6 Nov (reported) |
| 5.9 | IT -> Jump `.84`, `.85` | `ssh <user>@172.16.0.84` | Login prompt | PASS 6 Nov (reported) |
| 5.10 | HR -> Jump `.84` | `Test-NetConnection 172.16.0.84 -Port 22` | **Fails** (no SSH permit for HR) | PASS 6 Nov (reported) |
| 5.11 | Staff/ATM -> Web `.82` | `Test-NetConnection 172.16.0.82 -Port 80` | **Fails** | PASS 6 Nov (reported) |
| 5.12 | HR -> `.81`, `.82`, `.83`, `.84`, `.85` | ping | **All fail** (no ICMP to the DMZ in plan.md) | PASS 6 Nov (reported) |
| 5.12a | HR -> DNS, internet name | `nslookup google.com 172.16.0.83` | Answer | PASS 4 Nov; PASS 6 Nov (reported) |
| 5.12b | HR -> DNS, another rack's name | `nslookup wkg00n.sitict.net 172.16.0.83` | Answer (via the ICT forwarder `172.18.4.32`) | PASS 4 Nov; PASS 6 Nov (reported) |
| 5.12c | HR -> DNS, own name | `nslookup gruppesechs.sitict.net 172.16.0.83` | `172.16.0.82` (internal view; outside clients get `203.149.210.189`) | PASS 6 Nov (HR gets 172.16.0.82 from the internal view) |
| 5.13 | DMZ -> internal | Web VM: `ping 172.16.0.74`; `nc -zv 172.16.0.74 445` | **Both fail** (containment) | PASS 6 Nov (reported) |
| 5.14 | DMZ -> user VLAN | Web VM: `ping 172.16.0.61` | **Fails** | PASS 6 Nov (reported) |

---

## Stage 6: Inbound from outside

Public mappings: Web `203.149.210.189` (1:1 since 9 Nov; the PA allows only 80 + 443), DNS `203.149.210.190`, Jump `203.149.210.185:2222`. On a Windows laptop use `curl.exe` (PowerShell's `curl` is `Invoke-WebRequest` and rejects `-k`/`-m`). Test from a personal laptop on the lab Wi-Fi or another rack.

| ID | Test | Expected | Device check | Result |
|----|------|----------|--------------|--------|
| 6.1 | `curl -m 5 http://203.149.210.189` | Web page | PA `show session all filter destination 203.149.210.189` | PASS 6 Nov (curl) |
| 6.2 | `curl -k -m 5 https://203.149.210.189` / browser | Page (certificate warning) | R1 static NAT `.189:443`; PA `INBOUND-WEB` | PASS 6 Nov (curl -k, via R1 static NAT); PASS 9 Nov after the 1:1 NAT change (browser) |
| 6.3 | `nslookup gruppesechs.sitict.net 203.149.210.190` | `203.149.210.189` | PA `show session all filter destination 203.149.210.190` | PASS 6 Nov (returns 203.149.210.189) |
| 6.3a | `nslookup gruppesechs.sitict.net` (lab resolver) + browser `https://gruppesechs.sitict.net` | `.189`, page loads | Lab resolver `172.18.18.36` caches for the record's TTL | PASS 6 Nov (lab resolver 172.18.18.36 returns .189) |
| 6.4 | `ssh -p 2222 <user>@203.149.210.185` | Jump host login | R1 static NAT `.185:2222`; PA `JUMP-SSH-IN` | PASS 6 Nov (landed on VM102, ens19 172.16.0.84, via R1 static NAT .185:2222) |
| 6.4a | HR/IT PC: `nslookup gruppesechs.sitict.net`, then browse it | `172.16.0.82` (internal view); page loads | VM101 views (HANDOVER 8.10) | PASS 6 Nov (HR nslookup returns 172.16.0.82; site loads) |
| 6.5 | `ping 203.149.210.189` | **Fails** | | PASS 6 Nov (no echo reply; TTL expired seen then). PASS 9 Nov: `tracert -d` to `.189` and `.187` both end at R1 `172.17.9.93` then `*`, no loop (`.189` is 1:1 and dropped by the PA; `.187` hits Null0) |
| 6.6 | `ssh -p 22 <user>@203.149.210.185`; any port other than 2222 | **Fails** | | PASS 6 Nov (no login prompt; no error text captured) |
| 6.7 | `ssh 172.17.9.93` (R1 WAN) | **Fails** | R1 `WAN-IN` final deny (P6) | PASS 6 Nov (no login prompt; no error text captured) |
| 6.8 | Anything to `172.16.0.x` | **Fails** (not routed) | | PASS 6 Nov (no login prompt; no error text captured) |
| 6.9 | Another rack's user browses `https://gruppesechs.sitict.net` | Page loads | | PASS 9 Nov |

---

## Stage 7: Out-of-band management (VLAN 99, via the jump hosts)

Same pattern as Lab 3.1: engineer -> jump host -> device management IP. Run from VM102 (`.44`), reached via the Proxmox console, IT VLAN (`ssh <user>@172.16.0.84`), or outside (`ssh -p 2222 <user>@203.149.210.185`).

| ID | Test from VM102 | Expected | Result |
|----|-----------------|----------|--------|
| 7.0 | `ip -br a`; `ip route` | ens18 `.44/28`, ens19 `.84/29`; one default via `.81` on ens19 | PASS 6 Nov |
| 7.1 | `ping 172.16.0.33` (R1), `.34` (R2) | Reply | PASS 6 Nov (.33 and .34 reply) |
| 7.2 | ping `.35`, `.36`, `.37`-`.40` | Reply | PASS 6 Nov (.35 - .40 reply) |
| 7.3 | ping `.41` (PA), `.42` (ASA), `.43` (Proxmox) | Reply | PASS 6 Nov (.41 PA, .42 ASA, .43 Proxmox reply; .44, .45, .46 also reply) |
| 7.4 | `ssh bigDonkBonkAdmin@<Cisco mgmt IP>` | Login. Pause first while TACACS times out, unless tac_plus is running (P13) | PASS 6 Nov (SSH to all Cisco devices works; login is LOCAL, tac_plus not running) |
| 7.5 | `ssh bigDonkBonkAdmin@172.16.0.42` (ASA) | Login | PASS 6 Nov. Needs ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa (ASA offers ssh-rsa only). First login forces a password change. |
| 7.6 | `ssh bigDonkBonkAdmin@172.16.0.41` (PA) | Login | PASS 6 Nov |
| 7.7 | `curl -k -s -o /dev/null -w "%{http_code}\n" https://172.16.0.43:8006` | `200` | PASS 6 Nov (200) |
| 7.8 | Repeat 7.1-7.4 from VM106 (`.45`) | Same | PASS 6 Nov (VM106 SSH and chrony seen; ping sweep from VM106 not shown) |
| 7.9 | From MGMT-PC (`.46`): `ssh` to `.35` | **Fails** (`MGMT-ONLY`); browser `https://172.16.0.43:8006` **works** | PASS 6 Nov (SSH to .35 refused; Proxmox GUI .43 works) |
| 7.10 | DSW1: `ping vrf OOB 172.16.0.44` | Reply | PASS 6 Nov |
| 7.11 | ASA: `ping management 172.16.0.44` | Reply | PASS 6 Nov |
| 7.12 | R1: `show access-lists WAN-IN` | P6: lines 10/20 reference `.94` | 6 Nov: P6 confirmed (lines referenced .94). **PASS 9 Nov:** lines 10/20 reference `172.17.9.93` on the live R1 (P6 closed) |
| 7.13 | VM102 and VM106: `chronyc tracking`; `chronyc -N sources`; `sudo ss -ulpn \| grep chronyd` | `Leap status: Normal`; `0.sg.pool.ntp.org` entries and the other TACACS VM listed; chronyd on **`0.0.0.0:123`** (only `127.0.0.1:323` = not serving) | PASS 6 Nov (VM102: Leap Normal, stratum 3, ss -uln shows 0.0.0.0:123; VM106 chrony Normal; sudo not available for this user) |
| 7.14 | Cisco devices: `show ntp associations`; `show ntp status` | Both `.44` and `.45` with `reach` > 0; `Clock is synchronized`. Either may be `*`; the other should be a candidate | PASS 6 Nov (R1 4 Nov; SW1 seen 6 Nov; other devices reported OK) |
| 7.15 | ASA: `show ntp associations`; `show ntp status`. PA: `show ntp` | Synchronized to `.44` / `.45` | PASS 6 Nov (ASA synced to .44, PA synced to .45) |
| 7.16 | VM102: `sudo chronyc clients` | The OOB devices (`.33`-`.42`) appear | Skipped (needs sudo) |

**Diagnosing 7.1 (P19):**
1. Ping sweep `.33`-`.43` from VM102. All down -> the Proxmox bond (`cat /proc/net/bonding/bond0` on the host: active slave `eno6`, both slaves up; LAN 4 cabled to DSW1 Gi1/0/24, not to a laptop). Only `.33` down -> R1.
2. R1: `show ip interface brief | include 0/1/0`; `show run interface GigabitEthernet0/1/0` (must be `vrf forwarding OOB`, `172.16.0.33 255.255.255.240`); `show ip arp vrf OOB`. Check for the lab image's `Mgmt-vrf` quirk.
3. DSW1: `show run interface GigabitEthernet1/0/22` (VLAN 99); `show mac address-table vlan 99`.

---

## Stage 8: Server outbound and special cases

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 8.1 | Txn `.76`: `curl -m 5 https://www.google.com` outside TXN-WINDOW, **after** the temporary rules are removed | **Fails** | PASS 9 Nov (covered by 13.2) |
| 8.2 | ASA: `packet-tracer input server tcp 172.16.0.76 12345 8.8.8.8 443` with TXN-WINDOW covering now (edit ASA `time-range` and PA `schedule`, then restore) | Allow on both firewalls | PASS 9 Nov (covered by 13.1, live curl instead of packet-tracer) |
| 8.3 | ASA: `packet-tracer input server udp 172.16.0.76 12345 172.16.0.83 53` | Should allow in window; **expected drop at `SERVER_IN` (P3)** | Not run (P3 decision still open) |
| 8.4 | Any server: `sudo apt update` | Works **only** while the temporary rules exist | PASS 4 Nov (installs done). 6 Nov: fails without the rules, as expected |
| 8.5 | After removing the temporary rules: repeat 8.4 | **Fails** | PASS 6 Nov (apt update fails; temporary rules not present after the reload). PASS 9 Nov after D1 (VM103 and VM100 `apt update` fail) |
| 8.6 | Jump `.84`: `chronyc sources` / `ntpdate -q 0.sg.pool.ntp.org` | Works (PA `DMZ-OUTBOUND-NTP`) | PASS 6 Nov (VM102 syncs to 0.sg.pool.ntp.org) |
| 8.7 | Jump `.84`: `dig @8.8.8.8 google.com +short` | Works | PASS 6 Nov (dig @8.8.8.8 from VM102) |
| 8.8 | VM101: `dig @172.18.4.32 google.com +short` | Works (PA `DNS-SVR-FORWARD` -> `ICT-DNS`) | PASS 4 Nov, PASS 6 Nov (dig @172.18.4.32 from VM101) |
| 8.9 | VM101, **after** removing the temporary rules: `dig @127.0.0.1 google.com +short` and PA `test security-policy-match from dmz to outside source 172.16.0.83 destination 172.18.4.32 protocol 17 destination-port 53 application dns-base` | Answer; rule `DNS-SVR-FORWARD` | PASS 6 Nov (dig @127.0.0.1 resolves; rule DNS-SVR-FORWARD present; policy-match command itself not run). PASS 9 Nov after D1 (`dig @127.0.0.1 google.com +short` answers) |

---

## Stage 9: Redundancy (optional, after Stages 0-8)

Run after Stage 10, so the final ACLs are in place. One at a time, restore after each. All failures are caused by **unplugging a cable** (no power-off tests, no config changes), except 9.1, which is a reversible `shutdown`. Keep `ping 8.8.8.8 -t` running on an HR host throughout, and after every restore check that all OSPF neighbours are FULL again (MD5 adjacencies must re-form) and run `show standby brief` on both DSWs.

| ID | Action | Expected | Result |
|----|--------|----------|--------|
| 9.1 | DSW2: `interface Vlan61` -> `shutdown` | DSW1 takes over VLAN 61; HR `ping 8.8.8.8 -t` drops a few packets then recovers | PASS 9 Nov |
| 9.2 | Unplug one member of SW3's port-channel to DSW2 | No loss | PASS 9 Nov |
| 9.3 | Unplug R1 G0/0/1 (R1 <-> PA) | PA reroutes via R2; internet recovers after OSPF converges | PASS 9 Nov |
| 9.4 | Unplug ASA E1/3 **and** E1/4 (ASA cut off from both DSWs) | DSW `show track 10` Down on both; `show ip route 0.0.0.0` via PA (`.117` / `.121`); HR/IT internet and DMZ recover. Internal servers unreachable (by design). | PASS 9 Nov |
| 9.4a | Unplug DSW2 Gi1/0/11 (DSW2 <-> ASA E1/4) only | DSW2 track 10 Down; `show standby brief`: DSW1 Active for all four VLANs; HR internet recovers, still through the ASA (E1/3) | PASS 9 Nov |
| 9.4b | Unplug PA eth1/3 (PA <-> ASA E1/1) | ASA `show ospf neighbor` loses `6.6.6.6`; ASA `show route 0.0.0.0` via R1 `.110` (E1/5, `wan-backup2`); HR/IT internet recovers (R1 still NATs). DMZ unreachable from the user VLANs (by design) | PASS 9 Nov, **slow**: noticeably longer outage than the other tests; HR ping showed `Reply from 172.16.0.59: Destination host unreachable` while OSPF reconverged. Two paths move at once (ASA default to R1 E1/5, and R1's return path probably via the PA + DSW bypass), so traffic is briefly asymmetric across the firewalls. Accepted for the lab; outage time not recorded |
| 9.6 | Unplug R1 G0/0/0 (ISP1 link; `show track 1` on R1 goes Down, R1 stops originating the default) | PA/ASA default moves to R2 `.91`; HR `tracert -d 8.8.8.8` shows `172.17.10.94`; R2 `show ip nat translations` shows `129.126.142.185`. From outside, `curl -k https://129.126.142.189` works; the name doesn't (lab DNS points at ISP1) | PASS 9 Nov (second attempt). R1 `show track 1` Down; `show ip route 0.0.0.0`: `Known via "ospf 1" ... metric 60 ... from 2.2.2.2`. The first attempt was most likely checked before the track (IP SLA every 5 s + timeout) and OSPF converged: wait before checking |
| 9.6a | Unplug R1 G0/0/1 **and** G0/1/1 (R1 cut off from the PA and the ASA, as if R1 had died) | Same result as 9.6: default moves to R2, internet recovers through ISP2 | PASS 9 Nov |
| 9.6b | During 9.6 or 9.6a, from the outside laptop: `curl -k -m 5 https://129.126.142.189` and `ssh -p 2222 <user>@129.126.142.185` | Both work through R2 (uRPF and `WAN-IN` on R2 don't block them) | PASS 9 Nov (SSH to `129.126.142.185:2222`) |
| 9.5 | Restore; re-run 0.1, 0.3, 0.5, 0.6, 0.19, 2.1 | Baseline (default back on R1) | PASS 9 Nov. Failback to ISP1 is slow (track must see ISP1 again, OSPF re-advertises, existing flows stay on R2's NAT until they time out) |
| 9.7 | After **every** restore: `show ip ospf neighbor` on R1, R2, DSW1, DSW2; `show ospf neighbor` on the ASA; `show routing protocol ospf neighbor` on the PA | Same neighbour counts as before (R1/R2/DSWs 3, ASA 5, PA 5), all FULL with MD5 | PASS 9 Nov |
| 9.8 | After restoring 9.4a / 9.4: `show standby brief` on both DSWs | DSW1 Active 23/46, DSW2 Active 61/67 again (preempt) | PASS 9 Nov |

**Finding (9 Nov): R1 track 1 flaps.** `show track 1` on R1 showed 87 changes since creation; only a few came from these tests, so the IP SLA probe to ISP1 (`172.17.9.94`) times out on and off. Each flap moves user traffic to R2 and back. Possible fix (**not adopted**): `show ip sla statistics 1`, then `track 1 ip sla 1 reachability` / `delay down 15 up 30` on R1 and R2.

---

## Stage 10: Final hardening re-test (run before Stage 9 and Stages 11-14)

1. Remove the temporary rules (HANDOVER.md 8.4) if not already done, and run 8.5 and 8.9.
2. Remove the temporary ICMP permits on DSW1 and DSW2 (marked `TODO (FINAL HARDENING)`):
   ```
   ip access-list extended ACL_HR_IN
    no permit icmp 172.16.0.56 0.0.0.7 any
   ip access-list extended ACL_IT_IN
    no permit icmp 172.16.0.64 0.0.0.7 any
   ```
3. ~~Tighten PA `HR-IT-INTERNET`~~ Dropped 6 Nov: an App-ID list blocks normal sites (YouTube = `youtube-base`/`quic`, not `ssl`), and plan.md gives HR/IT full internet. The rule stays `any/any`.
4. Re-run 2.1-2.3, 2.6, 3.5, 4.5, 5.6-5.9, 5.12b, 6.1-6.4.

| ID | Test | Expected after hardening | Result |
|----|------|--------------------------|--------|
| 10.1 | HR <-> IT ping | **Now fails** | PASS 9 Nov (fails) |
| 10.2 | HR `ping 172.16.0.57` / `.97` / `.90` | **Now fail** (inside 172.16.0.0/25) | PASS 9 Nov (all fail) |
| 10.3 | HR `ping 8.8.8.8` | Still works | PASS 9 Nov (after R1 `WAN-IN` line 50 was restored, see note) |
| 10.4 | HR -> File ping + share | Still works (explicit permits) | PASS 9 Nov (ping `.74`, `H:` mapped, file written) |
| 10.5 | HR/IT browsing and DNS | Still works (`HR-IT-INTERNET` is `any/any`). If a site breaks, check the PA traffic log | PASS 9 Nov (`nslookup` via `.83`, internal name -> `.82`, browsing works). HTTP to `.82` also re-checked after the ASA `INSIDE_IN` order fix |

**Incident during Stage 10 (9 Nov):** while removing R1's redundant ICMP line, line 50 of `WAN-IN` (`permit ip any 203.149.210.184 0.0.0.7`) was removed by mistake (`no 50`). All HR/IT internet and all inbound web/DNS/jump SSH stopped at once. Restored with `ip access-list extended WAN-IN` / `50 permit ip any 203.149.210.184 0.0.0.7`. Never remove that line (HANDOVER fix #32).

---

## Stage 11: AAA (TACACS+ and local fallback)

Run from VM102 (`.44`) and repeat a few rows from VM106 (`.45`). Passwords are in `plan.md`. Users: `bigDonkBonkAdmin` (privilege 15) and `donkBonkEng1` / `donkBonkEng2` (engineers: show, ping, traceroute, interface shutdown / no shutdown). Never run the "refused" commands as the admin.

### 11A: Matrix (TACACS up)

For every device in the table, run these four checks:

1. **Admin:** `ssh bigDonkBonkAdmin@<mgmt ip>`. IOS: `show privilege` says level 15. ASA: `show curpriv` says level 15. PA: `show system info` works and `configure` is accepted.
2. **Engineer, allowed:** `ssh donkBonkEng1@<mgmt ip>`. IOS: `show ip interface brief`, `show ip route`, `ping <mgmt ip>`. ASA: `show route`, `show ospf neighbor`. PA: `show system info`.
3. **Engineer, refused:** IOS: `reload`, `write erase`, `router ospf 1`, `ip access-list extended TEST`, `username x privilege 15 secret y`. ASA: `reload` (and `show running-config`, see note). PA: any `set ...` command and `commit`. Each is rejected.
   - **IOS:** `configure terminal` is **allowed** for engineers; the refusals (`router ospf 1`, `reload`) come from the parser view `eng-view` as `% Invalid input detected`, **before** TACACS is asked, so they do not appear as denials in the tac_plus log.
   - **ASA:** engineers land at `#` with privilege 7 (auto-enable, HANDOVER fix #30); `configure terminal` is permitted by the `engineer` profile; `reload` is refused with `Command authorization failed` and an `engineer deny reload` line in `authz.log`.
   - **ASA `show running-config`:** the `engineer` profile permits any `show ...`, so it is currently **allowed**. Optional tac_plus fix in HANDOVER section 12 G (not applied).
4. **Wrong password:** one attempt as `donkBonkEng2` with a wrong password is rejected.

Note: `ping 172.16.0.33` from R1 fails because `.33` is in VRF OOB. Use `ping vrf OOB 172.16.0.33` (not an AAA fault).

| ID | Device | Mgmt IP | Result |
|----|--------|---------|--------|
| 11.1 | R1 | 172.16.0.33 | PASS 9 Nov. Admin priv 15; engineer in `eng-view`, `show ip interface brief` / `show ip route` work; `router ospf 1` and `reload` refused (`% Invalid input`); wrong `donkBonkEng2` password rejected |
| 11.2 | R2 | 172.16.0.34 | Not re-tested individually (same IOS AAA config and view as R1) |
| 11.3 | DSW1 | 172.16.0.35 | Not re-tested individually (same as R1) |
| 11.4 | DSW2 | 172.16.0.36 | Not re-tested individually (same as R1) |
| 11.5 | SW1 | 172.16.0.37 | Not re-tested individually (same as R1) |
| 11.6 | SW2 | 172.16.0.38 | Not re-tested individually (same as R1) |
| 11.7 | SW3 | 172.16.0.39 | Not re-tested individually (same as R1) |
| 11.8 | SW4 | 172.16.0.40 | Not re-tested individually (same as R1) |
| 11.9 | PA | 172.16.0.41 | Not run in 11A (PA local login checked in 11.23) |
| 11.10 | ASA | 172.16.0.42 | PASS 9 Nov after fixes #29/#30. Admin `#` priv 15, everything permitted; engineer `#` priv 7, `show route` and `configure terminal` permitted, `reload` denied (`authz.log`: `donkBonkEng1 engineer deny reload`). Engineer's local ASA password was forced to change at first login |

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 11.11 | VM102 and VM106: `sudo grep donkBonkEng1 /var/log/tac_plus-ng/acct.log \| tail -20` after 11.1-11.10 | Each login and each executed command is listed with the user, the device IP and a timestamp | PASS 9 Nov (VM102: engineer's login and commands with user, device IP and time) |
| 11.12 | VM102: `sudo tail -30 /var/log/tac_plus-ng/authz.log` | ASA: refused commands appear as `deny` lines. IOS: refused by `eng-view` (no tac_plus deny line) | PASS 9 Nov (ASA `engineer deny reload`); IOS behaves as described |
| 11.13 | R1: `test aaa group tac-group donkBonkEng1 <pw> legacy` | `User successfully authenticated` | Not run (logins in 11.1 prove it) |
| 11.14 | Repeat checks 1 and 2 on two devices from VM106. Also, on the ASA: `test aaa-server authentication TACACS host 172.16.0.45 username bigDonkBonkAdmin password <pw>` | Same as from VM102; `INFO: Authentication Successful` | ASA -> VM106 direct test PASS 9 Nov (VM106 `authc.log` `172.16.0.42 bigDonkBonkAdmin ... shell login succeeded`); R1 through VM106 shown in 11.20 |

### 11B: TACACS down (local fallback)

**Keep a console session open on the device under test before stopping anything.** The ASA command authorization falls back to LOCAL only when TACACS is **unreachable** (never on a deny), so test it last. The service is **`tac_plus-ng`** (use the `ubuntu` account for `sudo`; `bigDonkBonkAdmin` has no sudo on the VMs).

**ASA dead time:** after tac_plus is stopped on both servers the ASA marks both FAILED and stays on LOCAL for ~10 minutes, even once tac_plus is back. Either wait, or run `aaa-server TACACS active host 172.16.0.44` and `... 172.16.0.45` (check with `show aaa-server TACACS`). During that window the ASA wants the **local** password, which differs from `plan.md`.

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 11.20 | VM102: `sudo systemctl stop tac_plus-ng`. SSH to R1 as `bigDonkBonkAdmin` | Login works through VM106 (`.45`) | PASS 9 Nov. VM106 `authc.log`: `172.16.0.33 bigDonkBonkAdmin ... shell login succeeded`. No delay: VM102 is up with only the service stopped, so `.44` refuses at once. (R1 `show tacacs` still shows `.44` "Alive" because no `deadtime` is configured) |
| 11.21 | Also stop tac_plus-ng on VM106. SSH to R1 as `bigDonkBonkAdmin` | The local account logs in | PASS 9 Nov (no TACACS banner) |
| 11.22 | Same on DSW1 and SW1, and on the console of each | Local login works | Not run (same IOS AAA config as R1) |
| 11.23 | Same on the PA | Local login works (`AUTH-TACACS` then `AUTH-LOCAL` in `SEQ-ADMIN`) | PASS 9 Nov |
| 11.24 | Same on the ASA, then run `show curpriv`, `show version` and `configure terminal`. Also check the console login: log out, log back in (`Username:` prompt), `enable`, `configure terminal`, `exit` | Admin works (privilege 15); console session runs as the logged-in user, **not** `enable_15` | PASS 9 Nov. `Username : bigDonkBonkAdmin`, priv 15, `P_PRIV`; `configure terminal` works (falls back to LOCAL). `show version`: up 3 days 2 hours. Console-login check: done as part of the fix #29 work |
| 11.25 | Log in as `donkBonkEng1` locally on R1 | Lands in `eng-view`: `show` works, `router ospf 1` refused; `enable` refused | PASS 9 Nov (`show ip route` works; `router ospf 1` `% Invalid input`; `enable` -> `% Access denied`, as intended) |
| 11.26 | PA: 5 wrong passwords for `donkBonkEng2` | Account locked for 5 minutes; `bigDonkBonkAdmin` still logs in. Check with `show log system subtype equal auth` | Not run (optional) |
| 11.27 | ASA (TACACS still stopped so LOCAL is used): 6 wrong passwords for `donkBonkEng2` | `show aaa local user lockout` lists the user. Unlock: `clear aaa local user lockout username donkBonkEng2` | Not run (optional) |
| 11.28 | Restore: `sudo systemctl start tac_plus-ng` on VM102 and VM106; ASA `aaa-server TACACS active host .44/.45`. One admin login on R1 and on the ASA | TACACS login works again | PASS 9 Nov (both servers already `ACTIVE` on the ASA; ASA login used TACACS `.44`) |

---

## Stage 12: Logging and monitoring

Run 12A before Wazuh is involved, so you know the feeds work. Run 12B before 12C.

### 12A: Syslog and NetFlow arrive at `.44` and `.45`

On VM102, then VM106, start a capture and generate the event:
```
sudo tcpdump -ni ens18 -A udp port 514
sudo tcpdump -ni ens18 -c 10 udp port 2055
```

| ID | Device | Generate the event | Expected | Result |
|----|--------|--------------------|----------|--------|
| 12.1 | R1, R2, DSW1, DSW2 | `configure terminal`, then `end` | A `%SYS-5-CONFIG_I` line from `.33`, `.34`, `.35`, `.36` arrives on **both** `.44` and `.45` | R1 on `.44`: PASS 9 Nov (`172.16.0.33 > 172.16.0.44.514: SYSLOG local7.notice ... %SYS-5-CONFIG_I: Configured from console by bigDonkBonkAdmin on vty0 (172.16.0.44)`). R2, DSW1, DSW2 and the `.45` copy: not captured |
| 12.2 | SW1-SW4 | Same | Lines from `.37`-`.40` on both | Not captured (SW3 events reach Wazuh, see 12.25) |
| 12.3 | ASA | `configure terminal`, then `end` | An ASA message (for example `%ASA-5-111008` / `111010`) from `.42` on both | Partial 9 Nov: ASA syslog arrives on `.44` (`%ASA-6-302020` / `302021`, local4.info). The `111008`/`111010` config event itself not captured |
| 12.4 | DSW1 | From STAFF-PC: `Test-NetConnection 172.16.0.74 -Port 445` | `%SEC-6-IPACCESSLOGP ... ACL_STAFF_IN denied` in `show logging` on DSW1 and on `.44` | Not run |
| 12.5 | R1 | From the outside laptop: `ssh 172.17.9.93` | `WAN-IN denied tcp ... eq 22` logged on R1 and arrives at `.44` | Not run |
| 12.6 | DSW1 | From MGMT-PC (`.46`): `ssh bigDonkBonkAdmin@172.16.0.35` | `list MGMT-ONLY denied 172.16.0.46` logged in `show logging` and arrives at `.44` | Not run |
| 12.7 | ASA | VM103: `ping 172.16.0.61` (4.15) | `%ASA-4-106023 Deny` in `show logging` and at `.44` | Not run |
| 12.8 | R1, DSW1 | `show logging` | `Console logging: disabled`; buffer logging at informational; entries from 12.1 present | Not run |
| 12.9 | R1, R2, DSW1, DSW2 | `show flow exporter statistics`; `show flow monitor bank-flow-monitor cache` | Exporter packets sent increase and no export errors; cache lists flows. (Only these four have a flow monitor.) | Not run |
| 12.10 | VM102 and VM106 | The `tcpdump ... udp port 2055` capture | NetFlow v9 packets from `.33`-`.36` on both | Not run |

**Observation (9 Nov): ASA log noise.** The `302020`/`302021` lines are the DSWs' IP SLA probes (`track 10`, every 5 s), about 4 lines per 5 s all day into Wazuh (VM102 disk ~77%). Optional, not applied: `no logging message 302020` / `no logging message 302021` on the ASA. Timestamps: R1 and the ASA log in local time (SGT), matching VM102.

### 12B: PA log forwarding (P20)

**Status 9 Nov:** this config was already committed and complete on the live PA (including both zone lines rejected on 7 Nov), and it is now in `pa440.ttl` section 10b. The block below is kept for reference.

The PA has the `SYSLOG-JUMP` profile, but nothing is sent to it until a log-forwarding profile is attached. In `configure` mode (check each keyword with `?` as before; web UI paths are given):

```
! Objects > Log Forwarding: traffic and threat logs
set shared log-settings profiles LFP-WAZUH match-list ML-TRAFFIC log-type traffic filter "All Logs" send-syslog SYSLOG-JUMP
set shared log-settings profiles LFP-WAZUH match-list ML-THREAT log-type threat filter "All Logs" send-syslog SYSLOG-JUMP

! Device > Log Settings: system (includes admin login failures and lockouts) and config logs
set shared log-settings system match-list ML-SYS filter "All Logs" send-syslog SYSLOG-JUMP
set shared log-settings config match-list ML-CFG filter "All Logs" send-syslog SYSLOG-JUMP

! Attach the profile to every rule that should be logged
set rulebase security rules INBOUND-WEB log-setting LFP-WAZUH
set rulebase security rules INBOUND-DNS log-setting LFP-WAZUH
set rulebase security rules JUMP-SSH-IN log-setting LFP-WAZUH
set rulebase security rules INTERNAL-TO-DNS log-setting LFP-WAZUH
set rulebase security rules INTERNAL-TO-WEB log-setting LFP-WAZUH
set rulebase security rules IT-TO-JUMP-SSH log-setting LFP-WAZUH
set rulebase security rules HR-IT-INTERNET log-setting LFP-WAZUH
set rulebase security rules TXN-INTERNET-TIMED log-setting LFP-WAZUH
set rulebase security rules DMZ-OUTBOUND-NTP log-setting LFP-WAZUH
set rulebase security rules DNS-SVR-FORWARD log-setting LFP-WAZUH
set rulebase security rules DENY-ALL log-setting LFP-WAZUH

! DoS rule (Slowloris) and the zones (zone-protection scan/flood logs)
set rulebase dos rules DOS-DMZ-SERVERS log-setting LFP-WAZUH
set zone outside network log-setting LFP-WAZUH
set zone dmz network log-setting LFP-WAZUH
validate full
commit
```
Web UI equivalents: Security / DoS Protection rule > Actions > Log Forwarding; Network > Zones > Log Setting; Device > Log Settings. The `ALLOW-OSPF` rule is left out on purpose (it has `log-end no`).

Syslog leaves the PA over its management port (`172.16.0.41`), which is in the OOB subnet with `.44`/`.45`, so no firewall rule is needed.

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 12.11 | `show config running \| match LFP-WAZUH` | Profile, both match lists and every `log-setting` line above | PASS 9 Nov (profile + 14 `log-setting` lines: 11 rules, `DOS-DMZ-SERVERS`, zones `outside` and `dmz`) |
| 12.12 | HR browses a site; on VM102 `sudo tcpdump -ni ens18 -A udp port 514 and host 172.16.0.41` | A TRAFFIC line naming rule `HR-IT-INTERNET` | PASS 9 Nov (TRAFFIC for `HR-IT-INTERNET` from `172.16.0.61`, app `ssl`; also `INBOUND-WEB`, `INBOUND-DNS`, `DMZ-OUTBOUND-NTP`) |
| 12.13 | Outside laptop: `printf 'SSH-2.0-test\r\n' \| nc -w 3 203.149.210.189 80` (SSH banner on the web port; `INBOUND-WEB` allows only web-browsing and ssl) | `show log traffic` has a deny / reset entry for rule `DENY-ALL` (application `ssh` or `unknown-tcp`), and the TRAFFIC line arrives at `.44` | Not run |
| 12.14 | Outside laptop: `nmap -sS -Pn --max-rate 80 -p 1-1000 203.149.210.189` (would block your source for 300 s) | `show log threat` has scan entries; THREAT lines arrive at `.44` | **Not achievable — documented limitation (9 Nov).** Before the 1:1 NAT change no scan reached the PA (Null0 at R1). After it, four attempts never triggered TCP Port Scan block-ip: full-speed scans trip SYN-flood SYN cookies first (scan masked; many ports falsely show `open`), and below that rate `DENY-ALL` drops make nmap slow down so the distinct-port count never crosses the threshold in the 2 s interval. TCP Flood threat entries were logged instead (evidence for 14.1). See `changes.md` Parts 3.5 and 15.2 |
| 12.15 | PA: one failed admin login, then `commit` | A SYSTEM line (auth failure) and a CONFIG line arrive at `.44` | Auth part PASS 9 Nov (`SYSTEM,auth ... auth-fail ... failed authentication for user 'bigDonkBonkAdmin'`). CONFIG line not captured |

**Finding from the forwarded PA logs (9 Nov):** repeating `routed-OSPF-md5chksum-bad ... interface ethernet1/6, from IP 172.16.0.118` exposed the wrong OSPF key on DSW1 Gi1/0/12 (P21). After the fix the PA has 5 neighbours again.

### 12C: Wazuh

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 12.20 | Manager: `<remote>` block for syslog (below) is active; `sudo ss -ulpn \| grep 514` | Manager listens on udp/514 | PASS 9 Nov (`UNCONN 0.0.0.0:514 users:(("wazuh-remoted",...))`) |
| 12.21 | Wazuh dashboard, events filtered by source | Log lines from `.33`-`.42` appear, and the PA lines from 12B decode (check one with `sudo /var/ossec/bin/wazuh-logtest`) | PASS 9 Nov. R1 `%SYS-5-CONFIG_I` -> decoder `cisco-ios`, rule 4721 (level 3); PA `auth-fail` -> decoder `paloalto`, rule 64502 (level 3). ASA events show under `location:172.16.0.42` as "ASA error message" (label from the `%ASA-3` severity, not a fault). Only rule matches at level >= 3 are stored; `logall` deliberately off (disk) |
| 12.22 | Manager: `sudo /var/ossec/bin/agent_control -l` | Agents on VM100, 101, 103, 104, 105, 107 are `Active` (no agent on VM106 by decision; VM102 is the manager) | PASS 9 Nov (`file`, `web`, `dns`, `accounts`, `txn`, `dhcp` Active, plus `000 tacacs (server)`) |
| 12.23 | VM103 (internal server): `nc -zv 172.16.0.84 1514` and `1515` | Succeeds (needs the P17 rules on the ASA and the PA) | PASS 9 Nov (both ports). `172.16.0.85` fails, as expected: VM106 runs no manager |
| 12.24 | Three wrong SSH passwords to the jump host from outside | An SSH authentication-failure alert in Wazuh | PASS 9 Nov ("sshd: authentication failed", rule 5760, agent `tacacs`) |
| 12.25 | Err-disable a user port (`shutdown` / `no shutdown` on an access port) | The `%LINK` / `%PM` events appear in Wazuh | PASS 9 Nov (SW3 `.39`: "Router configuration changed", rule 4721) |

Manager side, for the syslog feed from the devices (`/var/ossec/etc/ossec.conf`, then `sudo systemctl restart wazuh-manager`):
```
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>172.16.0.32/28</allowed-ips>
</remote>
```
Devices send to both `.44` and `.45`. If the manager runs on only one of them, the other copy is simply not collected.

---

## Stage 13: Time window, DNS exposure and self-audit

### 13A: TXN window, live (covers 8.1 and 8.2)

Use `DEMO-DAY.md` section 1 to change the window on **both** firewalls, and restore the final window afterwards. Test by IP, from the VM105 console.

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 13.1 | Window covers now. VM105: `curl -k -m 5 -s -o /dev/null -w "%{http_code}\n" https://8.8.8.8` | A status code (200 or 302). ASA `show access-list SERVER_IN \| include TXN` hit counts rise; PA `show session all filter source 172.16.0.76` lists the session | PASS 9 Nov (ASA `periodic daily 16:00 to 19:00`, PA `recurring daily 16:00-19:00`, tested ~17:10) |
| 13.2 | Window ends before now (set it two hours ahead). Same `curl` | Times out (`000`) | PASS 9 Nov (window 19:00-21:00 on both; `000`; ASA time-range inactive) |
| 13.3 | Any window. VM103 and VM104: same `curl` | Fails (only VM105 is allowed out) | PASS 9 Nov (`000`) |
| 13.4 | Restore the final window. ASA `show time-range TXN-WINDOW`; PA `show config running \| match TXN-WINDOW` | Both show the same days and hours | PASS 9 Nov (restored to macro value: weekdays 01:00-03:00 on both) |

Run 13A only after the temporary rules are gone (they let every server out at any time, giving false results). **Never leave the window empty:** on 9 Nov it was briefly removed entirely; an ASA time-range with no `periodic` line may count as always active (VM105 internet 24/7), and the PA mishandles an empty schedule. It was restored straight away.

### 13B: DNS exposure (from the outside laptop, against `203.149.210.190`)

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 13.10 | `dig @203.149.210.190 google.com` | `status: REFUSED` (no recursion in the external view) | PASS 9 Nov |
| 13.11 | `dig @203.149.210.190 gruppesechs.sitict.net +short` | `203.149.210.189` | PASS 9 Nov |
| 13.12 | `dig @203.149.210.190 gruppesechs.sitict.net AXFR` | `Transfer failed` or `REFUSED` | PASS 9 Nov |
| 13.13 | `dig @203.149.210.190 version.bind chaos txt +short` | `"not disclosed"` | PASS 9 Nov |
| 13.14 | `for i in $(seq 1 100); do dig @203.149.210.190 gruppesechs.sitict.net +short +tries=1 +time=1; done \| sort \| uniq -c` | Fewer than 100 answers (rate limiting at 10 per second). On VM101, `sudo journalctl -u named \| grep -i limit` may show the limiting messages | Not run |
| 13.15 | HR PC: `nslookup gruppesechs.sitict.net` | `172.16.0.82` (internal view still works) | Not re-run (passed 6 Nov as 5.12c / 6.4a) |

### 13C: Self-audit with nmap

The scanner in 13.20 triggers PA scan protection and blocks your source for 300 seconds. Run it from a laptop you can wait out, or after the other outside tests.

| ID | From | Command | Expected | Result |
|----|------|---------|----------|--------|
| 13.20 | Outside laptop | `nmap -Pn -sS --max-rate 80 -p 22,23,53,80,443,2222 203.149.210.185-190` | Open only: `.185` 2222, `.189` 80 and 443, `.190` 53. Everything else `filtered` | PASS 9 Nov (exactly those open) |
| 13.21 | Outside laptop | `nmap -Pn -sU -p 53 203.149.210.190` | `53/udp` open or `open\|filtered` | PASS 9 Nov |
| 13.22 | Outside laptop | `ping 203.149.210.189`, `ping 203.149.210.186` | No reply | PASS 9 Nov |
| 13.23 | HR or IT PC | `nmap -Pn -p 22,23,80,443,8006 172.16.0.33-46` | All `filtered` (OOB is not reachable from users) | Not run (no nmap on the HR laptop). Equivalent `Test-NetConnection 172.16.0.35 -Port 22` / `172.16.0.43 -Port 8006` = `False` (3.8 passed 6 Nov) |
| 13.24 | VM102 | `nmap -Pn -p 22,23,80,443,830 172.16.0.33-42` | Cisco devices: only `22` open (no `23`, `80`, `443`). ASA: `22` only. PA: `22` and `443` | PASS 9 Nov (`.33`-`.40` and ASA `.42`: only 22 open, rest closed; PA `.41`: 22 and 443 open, rest filtered) |
| 13.25 | VM102 | `nmap -Pn -p 22,8006 172.16.0.43` | `22` and `8006` open (Proxmox) | PASS 9 Nov |

---

## Stage 14: Attack readiness

Done before the attacks in `DEMO-DAY.md` section 3, so each scene starts clean and the claim "normal users are not affected" is measured, not assumed.

### 14A: Normal use under attack

Baseline first, with no attack running. From the HR laptop, 20 requests to the web server, and 20 from the outside laptop to `.189`:
```
for i in $(seq 1 20); do curl -k -s -o /dev/null -w "%{http_code} %{time_total}\n" https://172.16.0.82; done
for i in $(seq 1 20); do curl -k -s -o /dev/null -w "%{http_code} %{time_total}\n" https://203.149.210.189; done
```
On a Windows laptop use PowerShell instead:
```
1..20 | % { curl.exe -k -s -o NUL -w "%{http_code} %{time_total}`n" https://172.16.0.82 }
```
Note the typical `time_total`. Repeat both loops while each attack runs.

| ID | Attack running | Expected (second outside laptop and HR) | Result |
|----|---------------|------------------------------------------|--------|
| 14.1 | SYN flood to `203.149.210.189:443` (Scene 4) | All 20 requests return `200`; times stay close to the baseline. If they are much slower, lower the flood rate for the demo | PASS 9 Nov. PA `show counter global filter delta yes \| match syn`: `flow_dos_syncookie_blk_dur` 20/s -> 610/s, `SynCookie sent 3953`. Wazuh: `TCP Flood` critical threat from `172.16.0.41`. HR and outside `curl` to `.189` still `200` at baseline time |
| 14.2 | Slowloris to `https://gruppesechs.sitict.net` (Scene 5) | All 20 return `200`; `show dos-protection rule DOS-DMZ-SERVERS` on the PA shows hits; the DoS log line arrives at `.44` | PASS 9 Nov. `show dos-protection rule DOS-DMZ-SERVERS statistics`: `sessions current 11, dropped 46374, ip tracked 3`. Wazuh: `DOS-DMZ-SERVERS` Session Limit Event. Second browser/curl still loads. (VM100 `ss` / nginx error.log not captured) |
| 14.3 | macof or dhcpig on one access port (Scene 1) | A second PC on **another port of the same switch** keeps its lease and its ping to the gateway | Not run 9 Nov (L2 defences verified in Stage 1 and HARDENING 4.1-4.4) |
| 14.4 | Brute force against `.185:2222` (Scene 6) | A legitimate login from IT (`ignoreip`) still works; after the ban, a legitimate login from a different outside address works | PASS 9 Nov via 12.24 (rule 5760; fail2ban `sshd` bans the source; IT in `ignoreip` unaffected) |

**Zone protection during the 9 Nov tests:** live `ZP-OUTSIDE` was tuned to SYN alarm 500 / activate 1000 / max 2000 and TCP Port Scan threshold 20 (from the macro's 100/200/2000 and 100). **Restored to the macro values on the evening of 9 Nov (PA commit job 101).** An activate rate of 200/s is low for a public website; fine for the lab, production values would come from a traffic baseline.

### 14B: Side effects and reset

Do these between runs of a scene, and before the demo.

| Attack | Side effect | Reset |
|--------|-------------|-------|
| MAC flooding (macof) | Extra MACs dropped and logged (`restrict`); port stays up; violation counter rises | Nothing: learned MACs age out after 5 min or on link-down. Optional `clear port-security dynamic interface Gi1/0/N`. Check `show port-security interface Gi1/0/N` |
| BPDU injection | Port err-disabled (`bpduguard`) | `shutdown` / `no shutdown`. Check `show spanning-tree inconsistentports` |
| DHCP starvation (dhcpig) | Port err-disabled (`dhcp-rate-limit`); possibly a few scope leases used | `shutdown` / `no shutdown`; `clear ip dhcp snooping binding *`. On VM107, if the scope is exhausted: `sudo systemctl stop isc-dhcp-server`, `sudo truncate -s 0 /var/lib/dhcp/dhcpd.leases`, `sudo systemctl start isc-dhcp-server` (clients renew) |
| ARP spoofing (bettercap) | Port may err-disable (`arp-inspection` rate) | `shutdown` / `no shutdown`. On the victim: `arp -d *` |
| DTP, double tagging | None | None |
| Spoofed source (hping) | WAN-IN counters and log lines | Optional: `clear ip access-list counters WAN-IN` on R1 for a clean demo |
| nmap scan | PA *may* block the scanner's source for 300 s (not triggered in the 9 Nov tests); fast scans trip SYN cookies and show false `open` ports | Wait 300 s, or remove the entry in Monitor > Block IP List, or scan from another address. Wait ~30 s after a flood log so the zone leaves SYN-cookie mode |
| SYN flood | Counters on the PA; sessions time out by themselves | Nothing. Do **not** `clear session all`: it drops every user session |
| Slowloris | Open connections time out after 10 s; nginx 429s | Nothing |
| SSH brute force | fail2ban bans the attacker for 15 min | `sudo fail2ban-client set sshd unbanip <ip>` |
| Admin lockout | PA account locked 5 min; ASA local account locked | ASA: `clear aaa local user lockout username <user>`. PA: wait 5 min, or unlock from another superuser |

---

## Stage 15: Caching DNS (VM108, 172.16.0.86)

All of these can be run remotely: `ssh -p 2222 bigDonkBonkAdmin@203.149.210.185` (lands on VM102), then SSH on from there. HR on 9 Nov: `nslookup google.com 172.16.0.86` and `nslookup gruppesechs.sitict.net 172.16.0.86` passed (ASA `INSIDE_IN` line 7 hitcnt 14).

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

---

## Results summary

| Stage | Status (9 Nov) |
|-------|----------------|
| 0 Infrastructure | All pass (0.18 confirmed 9 Nov) |
| 1 Addressing | Pass (reported) |
| 2 Internet | Pass (reported), incl. Staff/ATM deny tests |
| 3 VLAN isolation | Pass (reported); 3.5 HR <-> IT ping now fails (10.1) |
| 4 Internal servers | Pass (reported), incl. ATM and deny tests |
| 5 DMZ | Pass (reported); HR/IT resolve the site name to `172.16.0.82` |
| 6 Inbound | All pass (6.5 no loop and 6.9 confirmed 9 Nov) |
| 7 OOB | Pass. 7.16 skipped (no sudo). 7.12 / P6 closed 9 Nov |
| 8 Server outbound | Pass. 8.1/8.2 via 13A; 8.5/8.9 re-run after D1. 8.3 not run (P3 open) |
| 9 Redundancy | Pass 9 Nov (9.4b slow; failback slow; R1 track 1 flaps) |
| 10 Hardening | 10.1-10.3 pass 9 Nov; 10.4/10.5 to confirm |
| 11 AAA | Pass 9 Nov (ASA fixes #29/#30); 11.2-11.9 not re-tested individually; 11.22, 11.26, 11.27 not run |
| 12 Logging and monitoring | Wazuh, PA forwarding, decoding, agents, alerts pass. 12.1/12.3 partial; 12.2, 12.4-12.10, 12.13 not captured; 12.14 auto-block not achievable (documented) |
| 13 Time window, DNS exposure, self-audit | Pass 9 Nov (13.14, 13.15, 13.23 not run) |
| 14 Attack readiness | 14.1, 14.2, 14.4 pass; 14.3 not run |
| 15 Caching DNS (VM108) | HR lookups via .86 pass 9 Nov; 15.1-15.21 not run |
