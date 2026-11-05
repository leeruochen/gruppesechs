# Punggol Bank: Network Connectivity Test Plan

**Last updated:** 4 November 2026

Purpose: confirm every connection `plan.md` requires actually works, and every connection it forbids is actually blocked.

How to use it:
- Work through the stages in order. A later stage depends on the earlier ones.
- Every row has an **Expected** result. A deny test passes when the connection **fails**.
- Fill in the **Result** column (`PASS` / `FAIL` + date + note).
- When a row fails, run the device-side check for it. That shows which device dropped the traffic.

**Status at a glance (4 Nov):** all servers are built. Stage 0 passes. Internet, DHCP, DNS (internal, external and other racks' names), the file share, the Accounts portal, the web server (inside and outside, HTTP and HTTPS) and SSH into the jump host from outside have all been seen working. **Open:** the jump host can't reach R1's OOB address (7.1), the ATM DHCP lease after fix #19, and most deny tests haven't been run yet.

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
2. **Port security:** SW2 (ATM) allows 1 MAC per port, the others 2. A violation err-disables the port and it doesn't recover by itself. Before moving a laptop to a new port:
   ```
   clear port-security sticky interface GigabitEthernet1/0/<n>
   interface GigabitEthernet1/0/<n>
    shutdown
    no shutdown
   ```
   Check with `show port-security interface Gi1/0/<n>` and `show logging | include PSECURE`. A laptop with Hyper-V/VMware/VirtualBox adapters can send from two MACs and trip the limit of 1.
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
- **PA:** `test security-policy-match from <zone> to <zone> source <ip> destination <ip> protocol <6|17|1> destination-port <port> application <app>`. Use the base App-ID name (e.g. `dns-base`, not `dns`). Also `test nat-policy-match ...`.
  - Sessions: `show session all filter source <ip>` / `destination <ip>`, then `show session id <id>` (check `s2c` bytes > 0).
  - **Inbound** sessions are listed under the **public** destination IP, not the internal one.
  - Ping from the data plane: `ping source <dataplane-ip> host <ip>`. Plain `ping host` uses the management port.
- **DSW:** `show access-lists ACL_<VLAN>_IN`. Counters rise as traffic matches. Deny lines also log (`%SEC-6-IPACCESSLOGP`).

---

## Part 2: Predicted failures and their status

Found by reading the configs against `plan.md`. Each links to the test that exposes it.

| # | Problem | Status | Exposed by |
|---|---------|--------|-----------|
| P1 | ASA delivered HR/IT -> DMZ traffic out its own `dmz` interface (Vlan92), skipping the PA; replies via the PA were dropped | **FIXED 4 Nov** (ASA Vlan92 shut down; manual failover only) | 5.4, 5.6 |
| P2 | ASA only permitted DHCP relay from `.98/.99` | **FIXED 4 Nov** (`DSW-SVIS` object-group in `INSIDE_IN` / `SERVER_IN`) | 4.10 |
| P3 | Txn server can't use DNS `.83`: `SERVER_IN` line 1 denies SERVERS -> INTERNAL first | Open | 8.3 |
| P4 | No NAT for inbound HTTPS to the web server | **FIXED 4 Nov** (`NAT-WEB-HTTPS`) | 6.2 |
| P5 | Servers had no internet for `apt install`; DNS server couldn't reach forwarders | **Partly fixed.** DNS forwarders permanent (`DNS-SVR-FORWARD`, incl. ICT DNS). Package installs use the **temporary** rules (HANDOVER.md 8.4), still in place | 8.4, 8.9 |
| P6 | R1 `WAN-IN` SSH/Telnet denies reference `.94` (ISP), not `.93` (R1) | Open (low impact: the final deny still blocks it) | 6.7, 7.12 |
| P7 | HR <-> IT ping works while the temporary ICMP permits exist | Expected until Stage 10 | 3.5 |
| P8 | An ASA failure is a full outage (no automatic bypass) | By design (bypass ports down) | 9.4 |
| P9 | PA treated `172.16.0.72/29` as connected on dead vlan.69 | **FIXED 4 Nov** (vlan.69 removed from VR and OSPF) | 8.2 |
| P10 | `ACL_STAFF_IN` / `ACL_ATM_IN` dropped the peer DSW's HSRP hellos; both DSWs Active for VLANs 23/46 | **FIXED 4 Nov** (`permit udp <subnet> host 224.0.0.102 eq 1985`). Re-check 0.3 | 0.3 |
| P11 | DHCP renewals (unicast from client IPs) denied by ASA `INSIDE_IN` | Open (clients fall back to broadcast; slower renewals only) | 1B |
| P12 | Static HR binding blocked the DHCP lease for the same MAC | **FIXED 4 Nov** (static block removed; OPTIONAL in `sw3.ttl`) | 1.1 |
| P13 | Lockout risk once tac_plus answers (no fallback to local on a "fail") | Open: read before enabling TACACS | 7.4 |
| P14 | ASA/PA clocks not synced until chrony runs on `.44/.45` | Open | 8.1, 8.2 |
| P15 | Internal servers have no DNS once the temporary rules are removed | Open (decide if any internal server needs DNS) | 8.3 |
| P16 | Jump SSH (`.185:2222`) might not match `JUMP-SSH-IN` after port translation | **Not an issue** (6.4 passed) | 6.4 |
| P17 | Wazuh/syslog agents on internal servers can't reach `.84/.85` | Open: needs ASA + PA rules before Wazuh agents | n/a |
| P18 | DHCP replies addressed to the peer DSW's SVI crossed the user VLAN and were denied by the inbound user ACL (`denied udp 172.16.0.77(67) -> 172.16.0.51(67)`) | **FIXED 4 Nov** in macros (`permit udp host 172.16.0.77 eq bootps any eq bootps`). Confirm with 1.5 | 1.5 |
| P19 | VM102 (`.44`) couldn't reach R1 `.33`; R1/R2 NTP to `.44` showed `reach 0` | **RESOLVED 4 Nov.** Two separate causes: (1) the ping failure was temporary, almost certainly while Proxmox LAN 4 (the bond's primary) was unplugged / on a laptop; it passed on retry with ARP replies seen on `bond0`. (2) NTP to `.44`: the config was right, but the running chronyd hadn't loaded it, so it wasn't listening on udp/123 (`ss` showed only `127.0.0.1:323`) and was still using the Ubuntu pools. Fixed by restarting chrony and resetting R1's association. R1 now reaches `.44` (stratum 4). | 7.1, 7.14 |

---

## Stage 0: Infrastructure health (CLI only)

| ID | Device | Command | Expected | Result |
|----|--------|---------|----------|--------|
| 0.1 | DSW1, DSW2 | `show etherchannel summary` | Po1-Po5 `(SU)`, members `(P)`. **Po5 must be up.** | PASS 4 Nov |
| 0.2 | SW1-SW4 | `show etherchannel summary` | Both uplinks `(SU)` | PASS 4 Nov |
| 0.3 | DSW1, DSW2 | `show standby brief` | 23/46: DSW1 Active, DSW2 Standby. 61/67: DSW2 Active, DSW1 Standby. No `unknown` standby. | **Re-check** after P10 fix |
| 0.4 | DSW1, DSW2 | `show ip route 0.0.0.0` | `S* 0.0.0.0/0 via 172.16.0.97` | PASS 4 Nov |
| 0.5 | DSW1, DSW2 | `show track 10` | Up, only if the macro's tracked route was applied. A plain live static is acceptable. | n/a (plain static live) |
| 0.6 | ASA | `show ospf neighbor` | R2 `.113`, PA `.105`, DSW1 `.98`, DSW2 `.99` FULL (2WAY/DROTHER also normal). No PA `.100`. | PASS 4 Nov |
| 0.7 | ASA | `show route` | `0.0.0.0/0 via 172.16.0.105`; `172.16.0.80/29 via 172.16.0.105` (not connected `dmz`) | PASS 4 Nov |
| 0.8 | ASA | `show nat` | No `obj_any` dynamic NAT | PASS 4 Nov |
| 0.9 | PA | `show routing protocol ospf neighbor` | R1 `.90`, R2 `.91`, ASA `.106` full | PASS 4 Nov |
| 0.10 | PA | `show routing route` | `0.0.0.0/0 via .90` (static `DEFAULT-OUT`); `172.16.0.72/29 via .106` (O2); HR/IT via `.106` | PASS 4 Nov |
| 0.11 | PA | `show interface ethernet1/6`, `1/7` | Link state **down** | PASS 4 Nov |
| 0.12 | PA | `show running security-policy` | `DENY-ALL` last; `TEMP-SVR-UPDATES` present only while installs are ongoing | PASS 4 Nov |
| 0.13 | R1 | `show ip route 0.0.0.0` | Only `via 172.17.9.94` | PASS 4 Nov |
| 0.14 | R1 | `show track` | Track 1 and 2 Up | PASS 4 Nov |
| 0.15 | R1 | `show ip ospf neighbor` | PA `.92` + R2 `.91` on G0/0/1, R2 `.110` on G0/1/1, all FULL | PASS 4 Nov (after Mgmt-vrf fix) |
| 0.16 | R1 | `ping 172.17.9.94` | 5/5 | PASS 4 Nov |

---

## Stage 1: Each VLAN gets an address and reaches its gateway

| ID | From | Test | Expected | Note | Result |
|----|------|------|----------|------|--------|
| 1.1 | HR-PC | `ipconfig`, `ping 172.16.0.57` | Lease in `.61-.62`, ping replies | Ping works only because of the temporary ICMP permit | PASS 4 Nov (`.61`) |
| 1.2 | IT-PC | `ipconfig`, `ping 172.16.0.65` | Lease in `.68-.70`, ping replies | | PASS 4 Nov (`.68` seen) |
| 1.3 | STAFF-PC | `ipconfig`; `ping 172.16.0.1`; `arp -a` | Lease in `.4-.30`; **ping fails**; gateway MAC in ARP table; no DNS server | `ACL_STAFF_IN` ends in `deny ip any any` | Lease PASS 4 Nov (`.4`) |
| 1.4 | STAFF-PC | (DSW logs) | No `ACL_STAFF_IN denied ... -> 172.16.0.83(53)` | Confirms DNS isn't handed to Staff | PASS 4 Nov (after DHCP change) |
| 1.5 | ATM-PC | `ipconfig /renew`; `arp -a` | Lease in `.52-.54`; gateway `.49` MAC in ARP table | Needs the P18 fix live on both DSWs | PASS 4 Nov (`.52`; fixes #18/#19 confirmed on DSW1 and DSW2) |
| 1.6 | Each switch | `show ip dhcp snooping binding`; `show ip arp inspection statistics vlan <N>` | Laptop listed; DAI drop count flat | | |

---

## Stage 2: Internet access (HR and IT yes, Staff and ATM no)

| ID | From | Test | Expected | If it fails, check | Result |
|----|------|------|----------|--------------------|--------|
| 2.1 | HR-PC | `ping 8.8.8.8` | Reply | PA `show session all filter source 172.16.0.61` | PASS 4 Nov |
| 2.2 | HR-PC | `nslookup google.com` (uses `.83`) | Answer | 5.12a | PASS 4 Nov |
| 2.3 | HR-PC | Browse google.com, twitch.tv | Loads | PA `HR-IT-INTERNET` | PASS 4 Nov |
| 2.4 | HR-PC | `tracert -d 8.8.8.8` | `.57` -> `.97` -> `.105` -> `.90` -> `172.17.9.94` -> ... | Proves the serial path | |
| 2.5 | PA | `test nat-policy-match from inside to outside source 172.16.0.61 destination 8.8.8.8 protocol 6 destination-port 443` | `NAT-OUTBOUND` -> `203.149.210.185` | | |
| 2.6 | IT-PC | Repeat 2.1-2.3 | Same as HR | | |
| 2.7 | STAFF-PC | `ping 8.8.8.8`; `Test-NetConnection 8.8.8.8 -Port 443` | **Both fail** | DSW `ACL_STAFF_IN` deny counter | |
| 2.8 | ATM-PC | Same as 2.7 | **Both fail** | `ACL_ATM_IN` | |
| 2.9 | ASA | `packet-tracer input inside tcp 172.16.0.4 12345 8.8.8.8 443` | **drop** at `deny ip object-group STAFF-ATM any` | Second layer | |
| 2.10 | R1 | `ping 8.8.8.8` | **Fails (expected)**: R1 sources from the WAN `/30` | | PASS 4 Nov (fails as expected) |

---

## Stage 3: VLAN isolation (no VLAN reaches another)

Targets need inbound ICMP allowed (1.2 step 3) and a listener for TCP rows (e.g. `python -m http.server 8080`).

| ID | From -> To | Test | Expected | Result |
|----|-----------|------|----------|--------|
| 3.1 | HR -> Staff | ping, TCP 8080 | **Both fail** (reply dropped by `ACL_STAFF_IN`) | |
| 3.2 | HR -> ATM | ping, TCP | **Both fail** | |
| 3.3 | Staff -> HR | ping, TCP | **Both fail** | |
| 3.4 | ATM -> Staff | ping, TCP | **Both fail** | |
| 3.5 | HR <-> IT | ping | **Succeeds during testing (P7)**; must fail after Stage 10 | |
| 3.6 | HR -> IT | TCP 8080 | **Fails** | |
| 3.7 | IT -> HR | TCP 8080 | **Fails** | |
| 3.8 | HR-PC | `ping 172.16.0.35`; `Test-NetConnection 172.16.0.35 -Port 22` | **Both fail** (OOB is in VRF OOB) | |

---

## Stage 4: Internal servers (VLAN 69, behind the ASA)

### 4A: Baseline

| ID | Where | Test | Expected | Result |
|----|-------|------|----------|--------|
| 4.1 | ASA | `ping server 172.16.0.74` (and `.75`, `.76`, `.77`) | Reply | |
| 4.2 | Each VM | `ip route` | One default, via `172.16.0.73` | |
| 4.3 | Each VM | `ping -c 4 172.16.0.73` | Reply | |

### 4B: Access matrix (File = HR only; Accounts and Txn = Staff and ATM only)

| ID | From -> To | Test | Expected | Enforced at | Result |
|----|-----------|------|----------|-------------|--------|
| 4.4 | HR -> File `.74` | `ping 172.16.0.74` | Reply | DSW + ASA ICMP permits | |
| 4.5 | HR -> File `.74` | `net use H: \\172.16.0.74\HR /user:hrmanager`, save a file, check `sudo ls -la /srv/samba/hr` on VM103 | Works, file owned by hrmanager:hrshare | DSW, ASA `permit tcp HR FILE-SVR eq 445` | PASS 4 Nov |
| 4.6 | IT -> File `.74` | ping, `Test-NetConnection -Port 445` | **Both fail** (ICMP at ASA, TCP at DSW) | | |
| 4.7 | Staff -> File `.74` | `Test-NetConnection -Port 445` | **Fails** | DSW | |
| 4.8 | Staff -> Accounts `.75` | Browser `https://172.16.0.75` (login staff1); `Test-NetConnection -Port 443` | Page loads; True | DSW, ASA `permit ip STAFF-ATM ACCT-SVR` | PASS 4 Nov |
| 4.8a | Staff -> Txn `.76` | Same, against `.76` | Loads | ASA `permit ip STAFF-ATM TXN-SVR` | |
| 4.9 | ATM -> Accounts `.75`, Txn `.76` | Same (login atm1) | Loads | | **Pending** (needs 1.5) |
| 4.10 | DHCP relay | Leases on each VLAN (Stage 1) | Leases issued | ASA `DSW-SVIS` permits | PASS 4 Nov (HR, IT, Staff) |
| 4.11 | HR -> Accounts `.75`, Txn `.76` | ping, `Test-NetConnection -Port 443` | **Both fail** | ICMP at ASA, TCP at DSW | |
| 4.11a | HR -> DHCP `.77` | ping | **Fails** (leases still work via relay) | ASA | |
| 4.11b | HR -> ASA `.73` | ping | **Fails** (ASA only answers pings on the ingress interface `.97`) | | |
| 4.12 | IT -> Accounts / Txn | ping, TCP 443 | **Both fail** | | |
| 4.13 | Staff -> `.75` on another port (e.g. 22) | `Test-NetConnection -Port 22` | Reaches the server (plan.md allows Staff/ATM `ip` to these servers) | | |
| 4.14 | Server -> server | VM103: `nc -zv 172.16.0.75 443` | **Pass** (same subnet, no firewall in between) | | |
| 4.15 | Server -> user VLAN | VM103: `ping 172.16.0.61` | **Fails** (`SERVER_IN` denies SERVERS -> INTERNAL) | | |

---

## Stage 5: DMZ servers (VLAN 92, behind the PA)

### 5A: Baseline

| ID | Where | Test | Expected | Result |
|----|-------|------|----------|--------|
| 5.1 | PA | `ping source 172.16.0.81 host 172.16.0.82` (and `.83`, `.84`, `.85`) | Reply | |
| 5.2 | Each DMZ VM | `ip route` | One default, via `172.16.0.81` (jump hosts: on `ens19` only) | |
| 5.3 | ASA | `show interface Vlan92`; `show route 172.16.0.80` | Vlan92 **administratively down**; route via `.105` | PASS 4 Nov |

### 5B: Access matrix (HR/IT reach Web and DNS; IT reaches the jump hosts)

| ID | From -> To | Test | Expected | Result |
|----|-----------|------|----------|--------|
| 5.4 | ASA | `packet-tracer input inside tcp 172.16.0.61 12345 172.16.0.82 443 detailed` | Allow, egress **`outside`** | |
| 5.5 | PA | `test security-policy-match from inside to dmz source 172.16.0.61 destination 172.16.0.82 protocol 6 destination-port 443 application ssl` | `INTERNAL-TO-WEB` allow | |
| 5.6 | HR -> Web `.82` | Browser `http://172.16.0.82` and `https://172.16.0.82` | Page loads | PASS 4 Nov (HR and IT HTTPS sessions seen on the PA) |
| 5.7 | HR -> DNS `.83` | `nslookup google.com 172.16.0.83` | Answer | PASS 4 Nov |
| 5.8 | IT -> Web, DNS | Repeat 5.6, 5.7 | Pass | PASS 4 Nov (IT reached the site by name) |
| 5.9 | IT -> Jump `.84`, `.85` | `ssh <user>@172.16.0.84` | Login prompt | |
| 5.10 | HR -> Jump `.84` | `Test-NetConnection 172.16.0.84 -Port 22` | **Fails** (no SSH permit for HR) | |
| 5.11 | Staff/ATM -> Web `.82` | `Test-NetConnection 172.16.0.82 -Port 80` | **Fails** | |
| 5.12 | HR -> `.81`, `.82`, `.83`, `.84`, `.85` | ping | **All fail** (no ICMP to the DMZ in plan.md) | |
| 5.12a | HR -> DNS, internet name | `nslookup google.com 172.16.0.83` | Answer | PASS 4 Nov |
| 5.12b | HR -> DNS, another rack's name | `nslookup wkg00n.sitict.net 172.16.0.83` | Answer (via the ICT forwarder `172.18.4.32`) | PASS 4 Nov |
| 5.12c | HR -> DNS, own name | `nslookup gruppesechs.sitict.net 172.16.0.83` | `203.149.210.189` | |
| 5.13 | DMZ -> internal | Web VM: `ping 172.16.0.74`; `nc -zv 172.16.0.74 445` | **Both fail** (containment) | |
| 5.14 | DMZ -> user VLAN | Web VM: `ping 172.16.0.61` | **Fails** | |

---

## Stage 6: Inbound from outside

Public mappings: Web `203.149.210.189` (80 + 443), DNS `203.149.210.190`, Jump `203.149.210.185:2222`. Test from a personal laptop on the lab Wi-Fi or another rack.

| ID | Test | Expected | Device check | Result |
|----|------|----------|--------------|--------|
| 6.1 | `curl -m 5 http://203.149.210.189` | Web page | PA `show session all filter destination 203.149.210.189` | |
| 6.2 | `curl -k -m 5 https://203.149.210.189` / browser | Page (certificate warning) | PA NAT `NAT-WEB-HTTPS` | PASS 4 Nov |
| 6.3 | `nslookup gruppesechs.sitict.net 203.149.210.190` | `203.149.210.189` | PA `show session all filter destination 203.149.210.190` | PASS 4 Nov |
| 6.3a | `nslookup gruppesechs.sitict.net` (lab resolver) + browser `https://gruppesechs.sitict.net` | `.189`, page loads | Lab resolver `172.18.18.36` caches for the record's TTL | Pending (old `.186` cached on 4 Nov; re-check) |
| 6.4 | `ssh -p 2222 <user>@203.149.210.185` | Jump host login | PA `JUMP-SSH-IN`, `NAT-JUMP-SSH` | PASS 4 Nov |
| 6.5 | `ping 203.149.210.189` | **Fails** | | |
| 6.6 | `ssh -p 22 <user>@203.149.210.185`; any port other than 2222 | **Fails** | | |
| 6.7 | `ssh 172.17.9.93` (R1 WAN) | **Fails** | R1 `WAN-IN` final deny (P6) | |
| 6.8 | Anything to `172.16.0.x` | **Fails** (not routed) | | |
| 6.9 | Another rack's user browses `https://gruppesechs.sitict.net` | Page loads | | |

---

## Stage 7: Out-of-band management (VLAN 99, via the jump hosts)

Same pattern as Lab 3.1: engineer -> jump host -> device management IP. Run from VM102 (`.44`), reached via the Proxmox console, IT VLAN (`ssh <user>@172.16.0.84`), or outside (`ssh -p 2222 <user>@203.149.210.185`).

| ID | Test from VM102 | Expected | Result |
|----|-----------------|----------|--------|
| 7.0 | `ip -br a`; `ip route` | ens18 `.44/28`, ens19 `.84/29`; one default via `.81` on ens19 | PASS 4 Nov (addresses) |
| 7.1 | `ping 172.16.0.33` (R1), `.34` (R2) | Reply | PASS 4 Nov (failed earlier while the Proxmox bond was disturbed, P19) |
| 7.2 | ping `.35`, `.36`, `.37`-`.40` | Reply | |
| 7.3 | ping `.41` (PA), `.42` (ASA), `.43` (Proxmox) | Reply | |
| 7.4 | `ssh bigDonkBonkAdmin@<Cisco mgmt IP>` | Login. Pause first while TACACS times out, unless tac_plus is running (P13) | |
| 7.5 | `ssh bigDonkBonkAdmin@172.16.0.42` (ASA) | Login | |
| 7.6 | `ssh bigDonkBonkAdmin@172.16.0.41` (PA) | Login | |
| 7.7 | `curl -k -s -o /dev/null -w "%{http_code}\n" https://172.16.0.43:8006` | `200` | |
| 7.8 | Repeat 7.1-7.4 from VM106 (`.45`) | Same | |
| 7.9 | From MGMT-PC (`.46`): `ssh` to `.35` | **Fails** (`MGMT-ONLY`); browser `https://172.16.0.43:8006` **works** | |
| 7.10 | DSW1: `ping vrf OOB 172.16.0.44` | Reply | |
| 7.11 | ASA: `ping management 172.16.0.44` | Reply | |
| 7.12 | R1: `show access-lists WAN-IN` | P6: lines 10/20 reference `.94` | |
| 7.13 | VM102 and VM106: `chronyc tracking`; `chronyc -N sources`; `sudo ss -ulpn \| grep chronyd` | `Leap status: Normal`; `0.sg.pool.ntp.org` entries and the other TACACS VM listed; chronyd on **`0.0.0.0:123`** (only `127.0.0.1:323` = not serving) | PASS 4 Nov (VM102) |
| 7.14 | Cisco devices: `show ntp associations`; `show ntp status` | Both `.44` and `.45` with `reach` > 0; `Clock is synchronized`. Either may be `*`; the other should be a candidate | PASS 4 Nov on R1: `.45` `*` stratum 3 reach 377, `.44` stratum 4 reach 7 (rising). R2 synced to `.45`. Other devices to check |
| 7.15 | ASA: `show ntp associations`; `show ntp status`. PA: `show ntp` | Synchronized to `.44` / `.45` | |
| 7.16 | VM102: `sudo chronyc clients` | The OOB devices (`.33`-`.42`) appear | |

**Diagnosing 7.1 (P19):**
1. Ping sweep `.33`-`.43` from VM102. All down -> the Proxmox bond (`cat /proc/net/bonding/bond0` on the host: active slave `eno6`, both slaves up; LAN 4 cabled to DSW1 Gi1/0/24, not to a laptop). Only `.33` down -> R1.
2. R1: `show ip interface brief | include 0/1/0`; `show run interface GigabitEthernet0/1/0` (must be `vrf forwarding OOB`, `172.16.0.33 255.255.255.240`); `show ip arp vrf OOB`. Check for the lab image's `Mgmt-vrf` quirk.
3. DSW1: `show run interface GigabitEthernet1/0/22` (VLAN 99); `show mac address-table vlan 99`.

---

## Stage 8: Server outbound and special cases

| ID | Test | Expected | Result |
|----|------|----------|--------|
| 8.1 | Txn `.76`: `curl -m 5 https://www.google.com` outside TXN-WINDOW, **after** the temporary rules are removed | **Fails** | |
| 8.2 | ASA: `packet-tracer input server tcp 172.16.0.76 12345 8.8.8.8 443` with TXN-WINDOW covering now (edit ASA `time-range` and PA `schedule`, then restore) | Allow on both firewalls | |
| 8.3 | ASA: `packet-tracer input server udp 172.16.0.76 12345 172.16.0.83 53` | Should allow in window; **expected drop at `SERVER_IN` line 1 (P3)** | |
| 8.4 | Any server: `sudo apt update` | Works **only** while the temporary rules exist | PASS 4 Nov (installs done) |
| 8.5 | After removing the temporary rules: repeat 8.4 | **Fails** | |
| 8.6 | Jump `.84`: `chronyc sources` / `ntpdate -q 0.sg.pool.ntp.org` | Works (PA `DMZ-OUTBOUND-NTP`) | |
| 8.7 | Jump `.84`: `dig @8.8.8.8 google.com +short` | Works | |
| 8.8 | VM101: `dig @172.18.4.32 google.com +short` | Works (PA `DNS-SVR-FORWARD` -> `ICT-DNS`) | PASS 4 Nov |
| 8.9 | VM101, **after** removing the temporary rules: `dig @127.0.0.1 google.com +short` and PA `test security-policy-match from dmz to outside source 172.16.0.83 destination 172.18.4.32 protocol 17 destination-port 53 application dns-base` | Answer; rule `DNS-SVR-FORWARD` | |

---

## Stage 9: Redundancy (optional, after Stages 0-8)

One at a time, restore after each.

| ID | Action | Expected | Result |
|----|--------|----------|--------|
| 9.1 | DSW2: `interface Vlan61` -> `shutdown` | DSW1 takes over VLAN 61; HR `ping 8.8.8.8 -t` drops a few packets then recovers | |
| 9.2 | Unplug one member of SW3's port-channel to DSW2 | No loss | |
| 9.3 | Unplug R1 G0/0/1 (R1 <-> PA) | PA reroutes via R2; internet recovers after OSPF converges | |
| 9.4 | ASA E1/1 `shutdown` | **Full outage (P8)**. Don't bring the bypass ports up as a "fix". | |
| 9.5 | Restore; re-run 0.1, 0.3, 0.6, 2.1 | Baseline | |

---

## Stage 10: Final hardening re-test (last step)

1. Remove the temporary rules (HANDOVER.md 8.4) if not already done, and run 8.5 and 8.9.
2. Remove the temporary ICMP permits on DSW1 and DSW2 (marked `TODO (FINAL HARDENING)`):
   ```
   ip access-list extended ACL_HR_IN
    no permit icmp 172.16.0.56 0.0.0.7 any
   ip access-list extended ACL_IT_IN
    no permit icmp 172.16.0.64 0.0.0.7 any
   ```
3. Tighten PA `HR-IT-INTERNET` to `application [ web-browsing ssl dns ping ] service application-default`.
4. Re-run 2.1-2.3, 2.6, 3.5, 4.5, 5.6-5.9, 5.12b, 6.1-6.4.

| ID | Test | Expected after hardening | Result |
|----|------|--------------------------|--------|
| 10.1 | HR <-> IT ping | **Now fails** | |
| 10.2 | HR `ping 172.16.0.57` / `.97` / `.90` | **Now fail** (inside 172.16.0.0/25) | |
| 10.3 | HR `ping 8.8.8.8` | Still works | |
| 10.4 | HR -> File ping + share | Still works (explicit permits) | |
| 10.5 | HR/IT browsing and DNS | Still works with tightened App-ID. If a site breaks, check the PA traffic log for the App-ID it needed. | |

---

## Results summary

| Stage | Status (4 Nov) |
|-------|----------------|
| 0 Infrastructure | All pass; re-check 0.3 after the P10 fix |
| 1 Addressing | HR, IT, Staff, ATM pass |
| 2 Internet | HR pass; IT, deny tests not run |
| 3 VLAN isolation | Not run |
| 4 Internal servers | File share and Staff -> Accounts pass; deny tests and ATM pending |
| 5 DMZ | Web and DNS from HR/IT pass, incl. other racks' names; jump and deny tests not run |
| 6 Inbound | HTTPS, DNS, jump SSH pass; 6.1, 6.3a, deny tests pending |
| 7 OOB | VM102 -> R1 reachable; NTP from both TACACS VMs works on R1 (check the other devices); SSH to each device (7.4-7.8) to run |
| 8 Server outbound | Installs and DNS forwarding pass; post-cleanup tests pending |
| 9 Redundancy | Not run |
| 10 Hardening | Not started |
