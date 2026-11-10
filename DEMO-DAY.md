## 1. Set the final TXN-WINDOW (two devices)

| Device | Object | Current value (macro) | Where it is used |
|--------|--------|-----------------------|------------------|
| ASA | `time-range TXN-WINDOW` | `periodic weekdays 01:00 to 03:00` | `SERVER_IN` lines for `TXN-SVR` (www, https, DNS) |
| PA | `schedule TXN-WINDOW` | weekly Mon-Fri `01:00-03:00` | rule `TXN-INTERNET-TIMED` |


### 1.1 Check the clocks first
```
ASA:  show clock
PA:   show clock
```
If either is wrong, fix NTP first (`show ntp status` on the ASA, `show ntp` on the PA).

### 1.2 ASA (`configure terminal`)
Replace the old `periodic` line with the new one. `<days>` is `weekdays`, `daily`, `weekend`, or one day name (`monday`).
```
time-range TXN-WINDOW
 no periodic weekdays 01:00 to 03:00
 periodic <days> <HH:MM> to <HH:MM>
exit
show time-range TXN-WINDOW
```
Expect `status: active` when the current time is inside the window, and `inactive` outside it.

To make the window one specific day only, use an `absolute` range instead of `periodic`:
```
time-range TXN-WINDOW
 no periodic weekdays 01:00 to 03:00
 absolute start <HH:MM> <day> <month> <year> end <HH:MM> <day> <month> <year>
```

### 1.3 PA (`configure`)
Delete the old schedule contents, then set the new ones. The `delete` is needed because `set` only adds a day to the list.
```
configure
delete schedule TXN-WINDOW schedule-type
set schedule TXN-WINDOW schedule-type recurring weekly monday <HH:MM>-<HH:MM> tuesday <HH:MM>-<HH:MM> wednesday <HH:MM>-<HH:MM> thursday <HH:MM>-<HH:MM> friday <HH:MM>-<HH:MM>
commit
```
Daily version (every day, same hours):
```
set schedule TXN-WINDOW schedule-type recurring daily <HH:MM>-<HH:MM>
```
Verify: `show schedule TXN-WINDOW` in configure mode, or `show config running | match TXN-WINDOW`. Do **not** delete the whole `schedule TXN-WINDOW` object: the rule `TXN-INTERNET-TIMED` uses it and the commit would fail.

### 1.4 Prove it works
From the Proxmox console of VM105 (the jump hosts can't reach the internal servers):
```bash
curl -k -m 5 -s -o /dev/null -w "%{http_code}\n" https://8.8.8.8
or ping 8.8.8.8
```
- **Inside the window:** prints a status code (for example `200` or `302`).
- **Outside the window:** times out (`000`).

Test by IP, not by name: the Txn server can't resolve names through `172.16.0.83`.

---

## 3. Attack demo runbook

### 3.0 verify before demo

Run every check here and get the expected result. If one fails, fix it or drop that scene; don't debug live.

| # | Check | Where | Expected |
|---|-------|-------|----------|
| a | `show clock` | ASA, PA | within a minute of real SGT |
| b | `show ntp associations detail` | R1 | `.44` `authenticated` `*`, `.45` candidate |
| c | HR PC browses to `https://gruppesechs.sitict.net` | HR laptop | page loads by name |
| d | `nslookup gruppesechs.sitict.net` | outside (lab Wi-Fi) | `203.149.210.189` |
| e | `show config running \| match DOS-PER-CLIENT` | PA | DoS rule present (committed 6 Nov) |
| f | `show dos-protection rule DOS-DMZ-SERVERS` | PA | rule listed |
| g | Stage 6 inbound tests (`TEST-PLAN.md`) | outside | web `.189`, DNS `.190`, SSH `.185:2222` all work |
| h | Wazuh dashboard reachable, agents reporting | mgmt laptop | 6 agents Active (`file`, `web`, `dns`, `accounts`, `txn`, `dhcp`) + `000 tacacs`; no agent on VM106 by design |
| h2 | `show zone-protection zone outside` | PA | the values you intend to present (see section 2) |
| i | Kali attacker has a port in the right VLAN | rack | link up |
| j | Console session open on the device you demo AAA on | ASA / a switch | logged in as `bigDonkBonkAdmin` |

1 laptop in kali connected to SIT-WIFI
1 laptop in hr/it/staff/atm
1 laptop in 172.16.0.46, show wazuh, proxmox, 
- ssh -p 2222 -L 8443:172.16.0.41:443 bigDonkBonkAdmin@203.149.210.185
- https://localhost:8443, pa gui

### 3.1 

### Layer 2: DHCP starvation (ATTACKS.md #5)

- **Shows:** DHCP snooping, port-security, auto-recovery.
- **Attacker (Kali on an access port, e.g. SW1 Gi1/0/5):**
  ```
  sudo dhcpig -i eth0
  ```
- **results:** the port restricts and violation count goes up, `show port-security` in the switch
- **Defense (on that switch):**
  ```
  show ip dhcp snooping
  show logging | include DHCP_SNOOPING|PSECURE
  ```
- **Wazuh:** port-security event on that switch.

### Layer 2 MitM: ARP poisoning (ATTACKS.md #6)

- **Shows:** Dynamic ARP Inspection against the binding table.
- **Attacker (Kali, same VLAN as a victim):**
  ```
  sudo bettercap -iface eth0
  > net.probe on
  > set arp.spoof.targets 172.16.0.57
  > arp.spoof on
  ```
- **Audience sees:** no interception; the victim's gateway MAC is unchanged.
- **Defense (on the access switch):**
  ```
  show ip arp inspection statistics
  show ip arp inspection
  show logging | include SW_DAI
  ```
- every ARP is cross-checked against the DHCP-snooping binding; forged replies are dropped, bursts err-disable the port.
- **Wazuh:** DAI drop count rising.

### Authorization + App-ID (ATTACKS.md #8)

- **Shows:** DSW SVI ACLs, ASA zones, and the PA doing Layer-7, not just ports.
- **Attacker (Staff host):**
  1. Browse the HR file server `\\172.16.0.74` — denied at the DSW.
  2. Browse the internet — denied (Staff has no internet).
  3. Tunnel SSH over port 80 to a DMZ host — opened at L4, dropped by PA App-ID.
- **Defense:**
  ```
  ! DSW:
  show ip access-lists | include ACL_STAFF_IN
  show logging | include ACL_STAFF
  ! PA:
  show log traffic direction equal backward
  ```
- **Say:** the deny is at the first hop; and on the firewall a port being "open" doesn't mean the application is allowed (SSH-on-80 is blocked by App-ID, not by the port number).
- **Wazuh:** ACL deny + PA threat/traffic deny.

### Perimeter: recon then SYN flood (ATTACKS.md #9 + #11)

Main Palo Alto showcase. Two attacks back to back from **outside**.

- **Recon:**
  ```
  nmap -sS -Pn --max-rate 80 -p 1-1000 203.149.210.189
  nmap -sU -Pn 203.149.210.184/29
  ```
  Audience sees everything `filtered` except `80/443` on `.189` (verified 9 Nov, TEST-PLAN 13.20). Since 9 Nov the web NAT is 1:1, so every probe reaches the PA and is dropped by `DENY-ALL` (and logged to Wazuh), instead of vanishing at R1.
  - **Use `--max-rate 80`.** At full speed the scan trips SYN-flood protection first: the PA answers every SYN with a SYN cookie, so nmap shows dozens of ports falsely `open` and the scan is never counted. Wait ~30 s after any flood log before scanning.
  - **Do not promise an auto-block.** The TCP Port Scan block-ip could not be triggered in four attempts on 9 Nov (`changes.md` Part 15.2). Present recon as "all ports filtered except 80/443, every probe logged", and mention the flood-vs-scan threshold trade-off as a finding. If a block does appear, show it from a **different device** than the scanner (Monitor > Block IP List): the block also cuts the scanner's own sessions.
  ```
  show zone-protection zone outside
  show log threat
  ```
- **SYN flood** (`tcpsynflood.py` from `ATTACKS.md`):
  ```
  sudo python3 tcpsynflood.py 203.149.210.189 443
  ```
  While it runs, the HR laptop keeps loading the site. Show SYN cookies engaging:
  ```
  show counter global filter delta yes | match syn
  show zone-protection zone outside
  ```
  Evidence from the 9 Nov rehearsal (TEST-PLAN 14.1): `flow_dos_syncookie_blk_dur` rose from 20/s to 610/s, `SynCookie sent 3953`; HR and outside `curl` stayed `200` at baseline time.
- every probe reaches the stateful firewall and is dropped and logged; the SYN flood is absorbed by SYN cookies on the PA (the public server is behind the PA, so this is the PA's job, not the ASA's). Legit users never notice.
- **Wazuh:** PA threat log — `TCP Flood` (critical, location `172.16.0.41`) and the denied scan traffic.

### Slowloris (ATTACKS.md #12)

- **Shows:** classified DoS per-client on the PA + nginx host timeouts. Low-rate, so it contrasts with Scene 4's volumetric flood.
- **Attacker (outside):**
  ```
  sudo slowhttptest -c 1000 -H -u https://gruppesechs.sitict.net
  ```
  (the name resolves to `.189` from outside)
- **Audience sees:** the site stays up for a second browser.
- **Defense:**
  ```
  ! PA:
  show dos-protection rule DOS-DMZ-SERVERS
  ! VM100:
  ss -tn state established '( dport = :https )'
  sudo tail -n 20 /var/log/nginx/error.log
  ```
  Use `show dos-protection rule DOS-DMZ-SERVERS statistics` for the counters (9 Nov rehearsal: `sessions current 11, dropped 46374, ip tracked 3`).
- the PA caps one source IP at 50 sessions (`DOS-PER-CLIENT`, committed 6 Nov), and nginx drops any client that stalls its headers past 10 s.
- **Wazuh:** PA `DOS-DMZ-SERVERS` Session Limit Event; nginx 429s.

### AAA + brute force (ATTACKS.md #15 + #14)

Closes on the management plane. No flashy tooling, strong enterprise signal.

- **Command authorization:** SSH in as `donkBonkEng1` (via a jump host). Behaviour differs by platform (verified 9 Nov):
  - **ASA (`172.16.0.42`)** — use this for the "Command authorization failed" message. The engineer lands at `#` with privilege 7 (auto-enable):
    ```
    show route              -> allowed
    configure terminal      -> allowed (engineer profile)
    reload                  -> Command authorization failed
    ```
  - **IOS (e.g. R1 `172.16.0.33`)** — refusal comes from the parser view `eng-view`, before TACACS is asked:
    ```
    show ip interface brief -> allowed
    configure terminal      -> allowed
    router ospf 1 / reload  -> % Invalid input detected
    ```
  Then on VM102:
  ```
  sudo tail -n 20 /var/log/tac_plus-ng/acct.log     # logins and commands: user, device IP, time
  sudo tail -n 20 /var/log/tac_plus-ng/authz.log    # permit/deny lines (the ASA 'engineer deny reload')
  ```
  Use the `ubuntu` account for `sudo` (`bigDonkBonkAdmin` has no sudo on the VMs). Note `show running-config` is currently **allowed** for engineers on the ASA (open decision, HANDOVER 12 G): don't demo it as refused.
- **Brute force:** from outside, hammer the jump host:
  ```
  hydra -l admin -P rockyou.txt ssh://203.149.210.185:2222
  ```
  then:
  ```
  sudo fail2ban-client status sshd
  ```
  the attacker IP is banned (note it keeps its real IP through R1/R2 static NAT, so the ban hits the attacker, not the router).
- juniors are limited to show/ping by TACACS and everything is accounted; external guessing is banned at the host after 5 tries; management is OOB-only, so none of this is even reachable in-band (Scene reference: `ATTACKS.md` #13).
- **Wazuh:** "sshd: authentication failed" (rule 5760, agent `tacacs`; verified 9 Nov) + fail2ban ban. TACACS deny lines are in `authz.log` on VM102 (tac_plus-ng logs are not forwarded to Wazuh).

### 3.2

- Failover: with an HR ping running to `ping -t 8.8.8.8`, drop the ISP1 link (or R1); traffic fails over to ISP2/R2.

---

## 4. Firewall policy reference (ASA and PA)

Use this to answer "what does each firewall allow?" during the demo. Taken from `firepower1010.ttl` and `pa440.ttl`, which match the live configs (9 Nov `runshow.md` audit). User traffic passes the ASA first and then the PA, so a flow must be allowed by both.

### 4.1 ASA (Firepower 1010)

**Allowed from users** (`INSIDE_IN`, on `inside` from DSW1 and `inside2` from DSW2), in rule order:

| Source | Destination | Traffic |
|---|---|---|
| any | any | OSPF |
| DSW SVIs and link IPs (`DSW-SVIS`) | DHCP server .77 | udp/67 (DHCP relay) |
| HR | File server .74 | tcp/445 (SMB) and ICMP |
| Staff, ATM | Accounts .75, Txn .76 | all IP |
| HR, IT | DNS .83, caching DNS .86 | udp and tcp 53 |
| HR, IT | Web .82 | tcp 80, 443 |
| IT | Jump hosts .84, .85 | tcp/22 (SSH) |
| HR, IT | anything else (in practice, the internet) | all IP |

**Allowed from internal servers** (`SERVER_IN`, on `server`):

| Source | Destination | Traffic |
|---|---|---|
| DHCP .77 | DSW SVIs | udp/67 → 67 (DHCP replies) |
| Internal servers (172.16.0.72/29) | Wazuh manager on .84 | tcp 1514-1515 |
| Txn .76 | internet | tcp 80, 443, only in `TXN-WINDOW` (section 1) |

**Allowed from the PA and routers** (`OUTSIDE_IN`, on `outside`, `wan-backup`, `wan-backup2`): OSPF only. Replies to allowed connections return through the stateful connection table.

**Management:** SSH to the ASA only from .44 and .45 on the management interface. HTTP server (ASDM) is off.

**Not allowed** (all denies are logged):
- Users to the internal servers other than the lines above (e.g. HR/IT to Accounts or Txn, IT to the file server, Staff/ATM to the file server).
- Users to the DMZ other than the lines above (e.g. HR to the jump hosts, Staff/ATM to web or DNS, ICMP from users to the DMZ).
- Staff and ATM to anything other than Accounts and Txn, so no internet.
- Internal servers opening connections to anything in 172.16.0.0/25 (users, DMZ, other subnets), except Wazuh to .84.
- Internal servers to the internet, except Txn on 80/443 inside the window.
- Anything starting from outside: the internet, the PA, the DMZ, the routers (e.g. the jump hosts can't SSH into the internal servers).

**Protection:**
- Traffic to the internal servers: max 500 connections, 50 embryonic, 50 per client, 10 embryonic per client, 15 s embryonic timeout.
- Basic threat detection on.
- Stateful ICMP inspection.

### 4.2 PA-440

**Allowed** (rule order):

| # | Rule | From → To | Source → Destination | App / service |
|---|---|---|---|---|
| 1 | ALLOW-OSPF | outside, inside ↔ outside, inside | any | ospf |
| 2 | INBOUND-WEB | outside → dmz | any → Web .82 | web-browsing, ssl (default ports 80/443) |
| 3 | INBOUND-DNS | outside → dmz | any → DNS .83 | dns |
| 4 | JUMP-SSH-IN | outside → dmz | any → Jump1 .84 | ssh (R1/R2 forward public tcp/2222 to port 22) |
| 5 | INTERNAL-TO-DNS | inside → dmz | HR, IT → .83, .86 | dns |
| 6 | INTERNAL-TO-WEB | inside → dmz | HR, IT → .82 | web-browsing, ssl |
| 7 | IT-TO-JUMP-SSH | inside → dmz | IT → .84, .85 | ssh |
| 8 | HR-IT-INTERNET | inside → outside | HR, IT → any | any app, any service |
| 9 | TXN-INTERNET-TIMED | inside → outside, dmz | Txn .76 → any | web-browsing, ssl, dns, only in `TXN-WINDOW` |
| 10 | DMZ-OUTBOUND-NTP | dmz → outside | Jump hosts .84, .85 → any | ntp, dns |
| 11 | DNS-SVR-FORWARD | dmz → outside | .83, .86 → ICT DNS 172.18.4.32, 8.8.8.8, 1.1.1.1 | dns |
| 12 | WAZUH-AGENTS | inside → dmz | internal servers → .84 | tcp 1514-1515 |
| 13 | DENY-ALL | any → any (interzone) | any | deny, logged |

**Management:** the MGT port allows SSH and HTTPS. Telnet and HTTP are disabled.

**Not allowed:**
- From the internet: everything except web .82 on 80/443, DNS .83 on 53 and SSH to .84. Jump2 .85, caching DNS .86 and everything inside are unreachable. App-ID also drops non-web traffic on 80/443.
- Staff and ATM: anything through the PA (no internet, no DMZ).
- DMZ servers starting connections outward: the web server can't reach the internet, the DNS servers only reach their three forwarders, the jump hosts only use NTP and DNS.
- DMZ to inside: no new connections from the DMZ to users or internal servers.
- Internal servers to the internet, except Txn inside its window.
- HR SSH to the jump hosts, and users to the DMZ other than DNS, web and the IT SSH rule.

**Protection:**
- Zone protection on `outside` (`ZP-OUTSIDE`): SYN cookies 100/200/2000 (alarm/activate/max), UDP flood 500/1000/5000, ICMP flood 50/100/500. TCP port scans, host sweeps and UDP scans block the source for 300 s. Spoofed sources dropped.
- Zone protection on `dmz` (`ZP-DMZ`): spoofed sources dropped; port scans and host sweeps block the source for 300 s.
- DoS rule `DOS-DMZ-SERVERS`: max 50 concurrent sessions per source IP to .82, .83 and .84 (Slowloris).