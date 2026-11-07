# Demo Day Checklist

**Last updated:** 7 November 2026

Things that can only be done once the final details are given on the day.
The attack demo runbook is **section 3**; the TXN-window and readiness items are below it.

---

## 1. Set the final TXN-WINDOW (two devices)

The window controls when the Txn server (VM105, `172.16.0.76`) may reach the internet on tcp/80, tcp/443 and DNS. It is defined **twice**, and both must match, because traffic passes both firewalls:

| Device | Object | Current value (macro) | Where it is used |
|--------|--------|-----------------------|------------------|
| ASA | `time-range TXN-WINDOW` | `periodic weekdays 01:00 to 03:00` | `SERVER_IN` lines for `TXN-SVR` (www, https, DNS) |
| PA | `schedule TXN-WINDOW` | weekly Mon-Fri `01:00-03:00` | rule `TXN-INTERNET-TIMED` |

Both devices must show the correct time before you start (check with the commands below). All times are Singapore time (SGT, UTC+8).

### 1.1 Check the clocks first
```
ASA:  show clock
PA:   show clock
```
If either is wrong, fix NTP first (`show ntp status` on the ASA, `show ntp` on the PA). A wrong clock makes the window open at the wrong time.

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
```
- **Inside the window:** prints a status code (for example `200` or `302`).
- **Outside the window:** times out (`000`).

Test by IP, not by name: the Txn server can't resolve names through `172.16.0.83` (`SERVER_IN` line 1 denies it first, P3 in `TEST-PLAN.md`).

Device checks while testing:
```
ASA:  packet-tracer input server tcp 172.16.0.76 12345 8.8.8.8 443
ASA:  show access-list SERVER_IN | include TXN        (hit counts rise in the window)
PA:   show session all filter source 172.16.0.76
```

### 1.5 Dry run before the day (optional)
Set the window so that it covers "now", run 1.4, then set it back. This proves both devices work with the schedule syntax above.

### 1.6 If a device is reloaded
The macros rebuild the old value (`weekdays 01:00-03:00`). Redo 1.2 and 1.3 afterwards. The PA keeps its committed config across reloads, but the ASA does not.

---

## 2. Other demo-day items

- Make sure the temporary internet rules are removed (`HANDOVER.md` 8.4) and run `TEST-PLAN.md` 8.5 and 8.9.
- Remove the temporary ICMP permits on both DSWs (Stage 10 in `TEST-PLAN.md`).
- Re-enable ASA `aaa authorization command TACACS LOCAL` only if tac_plus is working (`HANDOVER.md` 8.11, P13).
- Confirm both TACACS/jump hosts and the DHCP, DNS and web VMs are running in Proxmox.
- Record the new ASA password if you changed it on first login. The macro resets it after a reload.

---

## 3. Attack demo runbook

Full attack catalogue (all 16, with mechanism and every verification command) is in `ATTACKS.md`. This section is the **presentation script**: six scenes, one per security tier, each with a visible before/after. Run them in order; the story moves from the edge of the LAN inward to management.

### 3.0 Before the audience arrives (pre-flight)

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
| h | Wazuh dashboard reachable, agents reporting | mgmt laptop | agents green |
| i | Kali attacker has a port in the right VLAN | rack | link up |
| j | Console session open on the device you demo AAA on | ASA / a switch | logged in as `bigDonkBonkAdmin` |

Lab laptops: run `ipconfig` on the outside test laptop. If it's `10.x` or `192.168.x`, see the `WAN-IN` warning in `HARDENING.md` 2.1 before any inbound scene.

Keep two screens up the whole time: the **device CLI** (left) and the **Wazuh dashboard** (right). Every scene ends by pointing at the Wazuh alert.

### 3.1 Opening (2 min, no attack)

1. Topology slide (`TOPOLOGY.md`): PTSO, dual ISP, two firewalls, OOB VRF.
2. One working path: HR laptop loads `https://gruppesechs.sitict.net`, then an outside laptop loads `https://203.149.210.189`. Same server, internal name vs public IP.
3. One sentence on the design: every user zone sits behind both a distribution ACL and a firewall, management is out-of-band only, and all of it is logged to Wazuh.

### Scene 1 — Layer 2: DHCP starvation (ATTACKS.md #5)

- **Shows:** DHCP snooping, port-security, auto-recovery. One attack, three defenses named.
- **Attacker (Kali on an access port, e.g. SW1 Gi1/0/5):**
  ```
  sudo dhcpig -i eth0
  ```
- **Audience sees:** the port drops within seconds; the attacker gets no lease.
- **Defense (on that switch):**
  ```
  show interfaces status err-disabled
  show ip dhcp snooping
  show logging | include DHCP_SNOOPING|PSECURE
  ```
- **Say:** untrusted ports can't flood or run a rogue server; the port auto-recovers after 300 s (`errdisable recovery`), so no truck-roll.
- **Wazuh:** err-disable / port-security event on that switch.

### Scene 2 — Layer 2 MitM: ARP poisoning (ATTACKS.md #6)

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
- **Say:** every ARP is cross-checked against the DHCP-snooping binding; forged replies are dropped, bursts err-disable the port.
- **Wazuh:** DAI drop count rising.

### Scene 3 — Authorization + App-ID (ATTACKS.md #8)

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

### Scene 4 — Perimeter: recon then SYN flood (ATTACKS.md #9 + #11)

Main Palo Alto showcase. Two attacks back to back from **outside**.

- **Recon:**
  ```
  nmap -sS -Pn 203.149.210.189
  nmap -sU -Pn 203.149.210.184/29
  ```
  Audience sees mostly `filtered`; only `80/443` open on `.189`. Then the scanner is block-listed:
  ```
  show dos-protection zone outside blocked source
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
- **Say:** scans are quarantined for 300 s; the SYN flood is absorbed by SYN cookies on the PA (the public server is behind the PA, so this is the PA's job, not the ASA's). Legit users never notice.
- **Wazuh:** PA threat log — scan block + flood.

### Scene 5 — Slowloris (ATTACKS.md #12)

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
- **Say:** the PA caps one source IP at 50 sessions (`DOS-PER-CLIENT`, committed 6 Nov), and nginx drops any client that stalls its headers past 10 s.
- **Wazuh:** PA DoS rule hit; nginx 429s.

### Scene 6 — AAA + brute force (ATTACKS.md #15 + #14)

Closes on the management plane. No flashy tooling, strong enterprise signal.

- **Command authorization:** SSH in as `donkBonkEng1` (via a jump host), then:
  ```
  configure terminal      -> rejected
  reload                  -> rejected
  show ip interface brief -> allowed
  ```
  Audience sees "Command authorization failed". Then on VM102:
  ```
  sudo tail -n 20 /var/log/tac_plus.acct
  ```
  the attempted command is logged with user, time and source IP.
- **Brute force:** from outside, hammer the jump host:
  ```
  hydra -l admin -P rockyou.txt ssh://203.149.210.185:2222
  ```
  then:
  ```
  sudo fail2ban-client status sshd
  ```
  the attacker IP is banned (note it keeps its real IP through R1/R2 static NAT, so the ban hits the attacker, not the router).
- **Say:** juniors are limited to show/ping by TACACS and everything is accounted; external guessing is banned at the host after 5 tries; management is OOB-only, so none of this is even reachable in-band (Scene reference: `ATTACKS.md` #13).
- **Wazuh:** TACACS command-authorization-failed + fail2ban ban.

### 3.2 Closing (1 min)

- Failover: with an HR ping running to `8.8.8.8`, drop the ISP1 link (or R1); traffic fails over to ISP2/R2 (HANDOVER section 2 has the table). Resilience is part of a secure design.
- One line: every scene you just saw raised an alert on the Wazuh board on the right — defense in depth **and** visibility.

### 3.3 Timing and fallbacks

- Budget ~3 min per scene + 2 open + 1 close = ~20 min of live demo. If short on time, the must-keep scenes are **3 (authorization/App-ID)**, **4 (PA perimeter)** and **6 (AAA)** — they cover the three graded themes (firewall/App-ID, DoS, AAA) and Wazuh ties them together.
- Any scene that misbehaves live: switch to the pre-captured screenshot/log for that scene and move on. Don't debug in front of the audience.
- Reset between runs: `clear port-security sticky interface <if>` and `clear ip dhcp snooping binding` for L2 scenes; scan block-list on the PA clears itself after 300 s.
