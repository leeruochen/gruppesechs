# Section 12 Changes (9 Nov 2026)

Record of the 9 Nov lab session working through HANDOVER.md section 12. Part 1 is an ASA AAA problem found and fixed along the way. Part 2 is section 12 B1 (device syslog into Wazuh).

> **Folded in (9 Nov, after the session):** every "Records to update" item below has been applied to the macros (`r1.ttl`, `r2.ttl`, `dsw1.ttl`, `dsw2.ttl`, `firepower1010.ttl`, `pa440.ttl`, `pa440_cleanup.ttl`) and docs (`HANDOVER.md`, `TEST-PLAN.md`, `DEMO-DAY.md`, `LAB-COVERAGE.md`, `ATTACKS.md`, `HARDENING.md`, `IDS-SURICATA.md`). The checkboxes are left as they were written during the session. Exceptions, still open: TEST-PLAN 10.4/10.5 "to confirm"; the R1/R2 track `delay` dampening and the engineer `show running-config` deny were decided **not adopted / left open**; the ASA `302020/302021` log suppression was not applied. New HANDOVER fix numbers: #29 (Part 1), #30 (Part 9.3), #31 (Part 4.2), #32 (Part 8.1), #33 (Part 3).

---

# Part 1: ASA AAA Fix (console session denied by TACACS)

**Device:** ASA (Firepower 1010, `172.16.0.42`)
**TACACS servers:** VM102 (`172.16.0.44`), VM106 (`172.16.0.45`), running `tac_plus-ng`
**Related:** HANDOVER.md section 4 (lockouts), section 5 (new fix #29), TEST-PLAN.md P13, Stage 11

---

## 1.1 Symptom

- On the ASA console, in `asa-1010(config)#`, every command returned `Command authorization failed`.
- `exit`, `end`, `show ospf interface` and `show curpriv` were all refused. It was impossible to leave config mode.
- SSH logins to the other devices (R1, DSW1) still worked through TACACS.

## 1.2 Diagnosis

### 1.2.1 TACACS was healthy

On VM102:

```bash
systemctl list-units | grep -i tac      # tac_plus-ng.service: active (running)
sudo ss -tlnp | grep ':49'              # tac_plus-ng listening on *:49
free -h                                 # 7.1 GiB available, no OOM
```

So the problem was not TACACS being down or out of memory. TACACS was answering, and the answer was "deny". The ASA only falls back to LOCAL when TACACS is **unreachable**, never when it denies. So the local account could not help.

### 1.2.2 The authorization log showed the real user

Service name is `tac_plus-ng` (not `tac_plus`). Config: `/etc/tac_plus-ng.cfg`. Logs: `/var/log/tac_plus-ng/authc.log`, `authz.log`, `acct.log`.

```bash
sudo tail -30 /var/log/tac_plus-ng/authz.log
```

Key lines:

```
2026-11-07 15:59:56  172.16.0.42  bigDonkBonkAdmin  ...  manager permit  shell  configure terminal
2026-11-09 05:21:54  172.16.0.42  enable_15         ...  deny            shell  end
2026-11-09 05:21:56  172.16.0.42  enable_15         ...  deny            shell  exit
2026-11-09 05:25:34  172.16.0.42  enable_15         ...  deny            shell  show curpriv
```

- On 7 Nov the ASA session arrived as `bigDonkBonkAdmin` and matched profile `manager` (permit all).
- On 9 Nov the console session arrived as **`enable_15`**, which is not a user in `tac_plus-ng.cfg`, so no rule matched and every command was denied.

### 1.2.3 The tac_plus-ng config was correct

- `profile manager` permits every shell command and sets `priv-lvl = 15`.
- `profile engineer` permits only `show|ping|traceroute|clear|interface|description|switchport|exit|end|logout|terminal`, `configure terminal` and `(no) shutdown`.
- `bigDonkBonkAdmin` is in `net_manager`, so it maps to `manager`.

No change to the TACACS config was needed.

## 1.3 Root cause

The ASA console had **no login authentication**. The session started without a username, and `enable` was entered with the ASA enable password. In that case the ASA names the session **`enable_15`**.

With `aaa authorization command TACACS LOCAL` turned on, the ASA sends every command to TACACS for approval using that username. TACACS has no user `enable_15`, so it denies everything, including `exit`.

Adding an `enable_15` user to tac_plus was rejected as a fix: it would give full command rights to anyone who knows the ASA enable password, bypassing per-user TACACS control and accounting.

## 1.4 Temporary unlock (used to get out of config mode)

1. Stop TACACS on **both** servers, so the ASA sees it as unreachable and falls back to LOCAL:
   ```bash
   sudo systemctl stop tac_plus-ng      # on VM102 and VM106
   ```
2. On the ASA console: `exit` worked.
3. Command authorization was turned off while the fix was applied (as allowed by TEST-PLAN 11.24):
   ```
   no aaa authorization command TACACS LOCAL
   ```

## 1.5 Permanent fix (ASA)

```
configure terminal
aaa authentication serial console TACACS LOCAL
aaa authentication enable console TACACS LOCAL
```

| Line | Effect |
|---|---|
| `aaa authentication serial console TACACS LOCAL` | The console now prompts for a username and password, checked against TACACS first, local if TACACS is unreachable |
| `aaa authentication enable console TACACS LOCAL` | `enable` keeps the logged-in username instead of turning the session into `enable_15`. The enable password comes from tac_plus (`enable =` on the user), or the user's local password if TACACS is down |

Then TACACS was started again on both servers:

```bash
sudo systemctl start tac_plus-ng      # on VM102 and VM106
```

## 1.6 Second problem: SSH to the ASA rejected the password

#### Symptom

```
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa bigDonkBonkAdmin@172.16.0.42
Permission denied (publickey,password).
```

3 attempts failed. (Local accounts lock after 5 failures: `max-fail 5`.)

#### Diagnosis

```bash
sudo tail -10 /var/log/tac_plus-ng/authc.log
```

There were **no `172.16.0.42` entries from 9 Nov**. The SSH attempts never reached TACACS, so the ASA was checking its **local** password.

#### Cause

Stopping tac_plus on both servers (section 1.4) made the ASA mark both servers **FAILED**. When every server in an `aaa-server` group has failed, the ASA uses LOCAL only and waits about **10 minutes** (dead time) before trying TACACS again.

The ASA's local password no longer matches `plan.md`: the first ASA login forced a password change (TEST-PLAN 7.5). So the TACACS password was being checked against the wrong (local) password store.

#### Fix

From the ASA console:

```
show aaa-server TACACS                        ! both servers FAILED
aaa-server TACACS active host 172.16.0.44
aaa-server TACACS active host 172.16.0.45
show aaa-server TACACS                        ! both ACTIVE
```

SSH then worked with the **TACACS** password, and `authc.log` showed a new `172.16.0.42 bigDonkBonkAdmin ... shell login succeeded` line.

## 1.7 Verification

| # | Test | Expected | Result |
|---|---|---|---|
| V1 | VM102: SSH to `172.16.0.42` as `bigDonkBonkAdmin` | Login; new `172.16.0.42 ... shell login succeeded` line in `authc.log` | PASS 9 Nov |
| V2 | ASA (SSH): `enable`, `show curpriv` | `Username : bigDonkBonkAdmin`, privilege 15 | |
| V3 | ASA (SSH): `aaa authorization command TACACS LOCAL`, then `show curpriv`, `exit`, `exit` | All permitted; `authz.log` shows `bigDonkBonkAdmin ... manager permit` | |
| V4 | ASA console: log out, log back in | `Username:` prompt appears | |
| V5 | ASA console: `bigDonkBonkAdmin`, `enable`, `configure terminal`, `exit` | All permitted; `authz.log` shows `bigDonkBonkAdmin`, **not** `enable_15` | |
| V6 | `show running-config aaa` | Contains both `aaa authentication ... console` lines and `aaa authorization command TACACS LOCAL` | |

## 1.8 Records to update

- [ ] **`firepower1010.ttl`:** add, **above** `aaa authorization command TACACS LOCAL`:
  ```
  aaa authentication serial console TACACS LOCAL
  aaa authentication enable console TACACS LOCAL
  ```
- [ ] **HANDOVER.md section 5:** add fix #29: "ASA console session ran as `enable_15` (no username), so TACACS denied every command under command authorization, including `exit`. Fix: `aaa authentication serial console TACACS LOCAL` + `aaa authentication enable console TACACS LOCAL`."
- [ ] **HANDOVER.md section 4, lockouts:** the console login now counts toward `max-fail 5`. The ASA's local password differs from `plan.md` (changed at first login); record the current one in private notes.
- [ ] **HANDOVER.md section 8.11, tac_plus row:** service is `tac_plus-ng`, config `/etc/tac_plus-ng.cfg`, logs in `/var/log/tac_plus-ng/` (`authc.log`, `authz.log`, `acct.log`).
- [ ] **TEST-PLAN.md Stage 11:** in 11.24, add the console-login check (V4, V5) and note the 10-minute dead time after TACACS comes back (section 1.9 below).

## 1.9 Lessons

1. **Read `authz.log` first.** The username in the deny line shows the cause straight away (`enable_15` = no login on that line).
2. **A TACACS "deny" never falls back to LOCAL.** Only an unreachable server does. Stopping tac_plus on both servers is a valid emergency unlock.
3. **After stopping TACACS, the ASA stays on LOCAL for about 10 minutes**, even after tac_plus is running again. Either wait, or run `aaa-server TACACS active host <ip>` for each server. During that window the ASA wants its **local** password.
4. **Keep a console session open** whenever changing AAA on the ASA (as HANDOVER section 4 already says).
5. Credentials (tac_plus passwords, enable secret, TACACS key) were displayed during troubleshooting. Rotate them after the session and keep them out of the repo.

---

# Part 2: Section 12 B1, device syslog into Wazuh

**Server:** VM102 (`tacacs`, `172.16.0.44`), Wazuh manager 4.14.10
**Related:** HANDOVER.md 12 B1, TEST-PLAN.md 12.1-12.3, 12.20

## 2.1 Change made

`/var/ossec/etc/ossec.conf` on VM102 (backup: `/var/ossec/etc/ossec.conf.bak`). A second `<remote>` block was added below the existing `secure` (agent) block:

```xml
  <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>udp</protocol>
    <allowed-ips>172.16.0.32/28</allowed-ips>
  </remote>
```

`allowed-ips` is the OOB subnet, because every device sends syslog from its management address (`.33`-`.42`).

```bash
sudo systemctl restart wazuh-manager
```

## 2.2 Results

| ID | Test | Result |
|---|---|---|
| 12.20 | `sudo ss -ulpn \| grep ':514'` | **PASS 9 Nov.** `UNCONN 0.0.0.0:514 users:(("wazuh-remoted",pid=333790,fd=4))`. Nothing else held udp/514 before the change |
| 12.1 (R1) | `sudo tcpdump -ni ens18 -A udp port 514` on VM102, then `configure terminal` / `end` on R1 | **PASS 9 Nov (R1, on `.44`).** `172.16.0.33 > 172.16.0.44.514: SYSLOG local7.notice ... %SYS-5-CONFIG_I: Configured from console by bigDonkBonkAdmin on vty0 (172.16.0.44)` |
| 12.3 (ASA, partial) | Same capture | **ASA syslog arrives on `.44`.** `172.16.0.42.514 > 172.16.0.44.514: SYSLOG local4.info ... %ASA-6-302020 / 302021` (ICMP built/teardown). The `configure terminal` / `end` event (`%ASA-5-111008` / `111010`) is still to be captured |

Sample lines captured:

```
172.16.0.42.514 > 172.16.0.44.514: SYSLOG local4.info
  <166>Nov 09 2026 14:56:18: %ASA-6-302020: Built inbound ICMP connection for faddr 172.16.0.98/52228 gaddr 172.16.0.97/0 laddr 172.16.0.97/0 type 8 code 0

172.16.0.33.56920 > 172.16.0.44.514: SYSLOG local7.notice
  <189>2223: Nov  9 14:56:20.113: %SYS-5-CONFIG_I: Configured from console by bigDonkBonkAdmin on vty0 (172.16.0.44)
```

## 2.3 Observations

1. **ASA log noise.** The `302020` / `302021` ICMP lines come from the DSWs' IP SLA probes (`track 10`: DSW1 `.98` -> `.97`, DSW2 `.102` -> `.101`, every 5 seconds). At `logging trap informational` that is about 4 lines every 5 seconds, all day, going into Wazuh. VM102's disk is already at about 77%. Optional fix on the ASA (keeps every other informational message):
   ```
   no logging message 302020
   no logging message 302021
   ```
   If applied, add both lines to `firepower1010.ttl` and note it in HANDOVER section 11.1 (Syslog row).
2. **Timestamps.** R1 and the ASA log in local time (`14:56`), matching the capture time on VM102. Keep every device and server on the same timezone so events line up in Wazuh.

## 2.4 Still to do for Stage 12A / B1

| ID | Test |
|---|---|
| 12.1 | R2, DSW1, DSW2: `configure terminal` / `end`, lines arrive on `.44` **and** `.45` (run the capture on VM106 too) |
| 12.2 | SW1-SW4: same, from `.37`-`.40` |
| 12.3 | ASA: `configure terminal` / `end` gives `%ASA-5-111008` / `111010` |
| 12.4-12.7 | ACL deny logs reach `.44` (DSW1 `ACL_STAFF_IN`, R1 `WAN-IN`, DSW1 `MGMT-ONLY`, ASA `106023`) |
| 12.8 | `show logging` on R1, DSW1: console logging disabled, buffer informational |
| 12.9-12.10 | NetFlow: `show flow exporter statistics` on R1, R2, DSW1, DSW2; `tcpdump udp port 2055` on VM102 and VM106 |

## 2.5 Records to update

- [ ] HANDOVER.md 12 B1: tick, with the result above.
- [ ] TEST-PLAN.md: fill in 12.20 and the R1 part of 12.1.

---

# Part 3: R1/R2 web NAT changed to 1:1 (scan visibility)

**Devices:** R1, R2
**Related:** HANDOVER.md 3.3 (public mappings), 10.1b (NAT commands), TEST-PLAN.md 6.1, 6.2, 12.14, 14.1

## 3.1 Problem

The nmap test (TEST-PLAN 12.14, report screenshot "PA Monitor > Logs > Threat / Block IP List") produced **no PA logs**.

R1 only had **port** static NAT for the web server:

```
R1>show ip nat translations
tcp  203.149.210.189:443   172.16.0.82:443   ---   ---
tcp  203.149.210.189:80    172.16.0.82:80    ---   ---

R1>show ip route 203.149.210.189
Routing entry for 203.149.210.184/29
  Known via "static", distance 1, metric 0 (connected)
  * directly connected, via Null0
```

Traffic to `.189` on any port other than 80/443 matched no NAT entry and was routed to **Null0** on R1. Of the 1000 scanned ports, only 2 reached the PA, so PA zone protection never saw a scan, never logged it and never blocked the scanner.

## 3.2 Design decision

Two options were considered:

| | Drop at R1 (old) | Pass to the PA (new) |
|---|---|---|
| Unused ports | Silently dropped at Null0 | Dropped by PA `DENY-ALL` |
| Scan detected / logged | No | Yes (threat log, forwarded to Wazuh) |
| Scanner blocked | No, it can go straight on to attack 80/443 | Yes, source blocked 300 s on every port (zone protection `Block IP`) |
| nmap result | 80, 443 open, rest filtered | Same |

The scan reveals the same open ports either way, since public services are always discoverable. The difference is detection and response. R1's `WAN-IN` is a **stateless** ACL: it cannot detect a scan or block a source, and it cannot be narrowed to 80/443/53/2222 because replies to users' outbound traffic return to `.185` on random UDP/TCP ports. The PA is stateful (App-ID, zone protection), so it does the filtering. R1 keeps the edge role: anti-spoofing (`WAN-IN`), uRPF, and Null0 for the **unused** public IPs (`.186`-`.188`).

Chosen: **1:1 static NAT for the web server, PA filters.** Security policy is unchanged: `INBOUND-WEB` still allows only web traffic and `DENY-ALL` drops the rest.

## 3.3 Change made

**R1** (`configure terminal`):
```
no ip nat inside source static tcp 172.16.0.82 80 203.149.210.189 80 extendable
no ip nat inside source static tcp 172.16.0.82 443 203.149.210.189 443 extendable
ip nat inside source static 172.16.0.82 203.149.210.189
```

**R2** (same, ISP2 address):
```
no ip nat inside source static tcp 172.16.0.82 80 129.126.142.189 80 extendable
no ip nat inside source static tcp 172.16.0.82 443 129.126.142.189 443 extendable
ip nat inside source static 172.16.0.82 129.126.142.189
```

DNS (`.190`, port 53 only) and jump SSH (`.185:2222`) keep their port statics.

## 3.4 Results

| ID | Test | Result |
|---|---|---|
| R1 NAT | `show ip nat translations \| include 189` | Expected: one line `--- 203.149.210.189  172.16.0.82  ---  ---` (no ports) |
| R2 NAT | Same on R2 (`129.126.142.189`) | **To confirm** |
| 6.1 / 6.2 | Website from the outside laptop (lab Wi-Fi) | **PASS 9 Nov** in the browser by URL. Note: in Windows PowerShell `curl` is an alias for `Invoke-WebRequest` and rejects `-k`/`-m`; use `curl.exe -k -m 5 https://203.149.210.189` |
| Packets reach the PA | nmap 1-1000 against `.189` | **PASS 9 Nov.** Ports other than 80/443 now reach the PA (seen as SYN cookie answers and TCP Flood logs, see 3.5) |
| 14.1 evidence | PA Monitor > Logs > Threat | **TCP Flood** entries, zone `outside` -> `outside`, source/destination `0.0.0.0` (zone-wide), 11/09 14:34:37, 14:34:47, 14:36:12. Usable as SYN flood evidence |
| 12.14 | TCP Port Scan entry + Block IP List | **To confirm** with the rate-limited scan in 3.5 |

## 3.5 Findings during the scan test

1. **"53 open ports" were false positives.** A full-speed scan showed ports such as 39, 41, 51, 104, 106, 121, 141, 170, 213 as open. The scan tripped SYN flood protection (`ZP-OUTSIDE`, action **SYN Cookies**), so the PA answered every SYN with a SYN-ACK on the server's behalf, and nmap reported those ports as open. Check: `sudo ss -tlnp` on VM100 shows only 80, 443 (and 22).
2. **Flood protection hides the scan.** In SYN cookie mode the PA does not create a session or policy deny per port, so Reconnaissance Protection never counts the scan. The scan must stay below the SYN flood **Activate** rate and above the scan threshold.
3. **`ZP-OUTSIDE` settings (unchanged):**

   | Setting | Value |
   |---|---|
   | SYN flood | Action SYN Cookies, Alarm 100, Activate 200, Maximum 2000 connections/s (zone-wide) |
   | TCP Port Scan | Enabled, Block IP, track by source, 300 s, interval 2 s, threshold 100 events |
   | Host Sweep | Enabled, interval 10 s, threshold 100 |
   | UDP Port Scan | Enabled, interval 2 s, threshold 100 |
   | IP Protocol Scan | Disabled |
   | Source address exclusions | None |

4. **Scan rate for the demo.** The scan threshold needs more than 50 ports/s (100 in 2 s). The flood Activate rate (200/s) counts all inbound connections to the zone, including other racks and lab users, so `--max-rate 200` still triggered SYN cookies. Use:
   ```
   nmap -sS -Pn --max-rate 80 -p 1-1000 203.149.210.189
   ```
   Wait about 30 s after any earlier flood log so the zone leaves SYN cookie mode. Open the PA GUI from a different device: the block also cuts the scanning laptop's SSH tunnel.
5. **Production note for the report:** an Activate rate of 200/s is low for a zone that serves a public website. It suits the lab demo; production values would be set from a measured traffic baseline.

## 3.6 Records to update

- [ ] `r1.ttl`: replace the two `.189` port-80/443 static lines with `ip nat inside source static 172.16.0.82 203.149.210.189`.
- [ ] `r2.ttl`: same with `129.126.142.189`.
- [ ] HANDOVER.md 3.3: Web row becomes "1:1 static NAT, all ports to the PA (the PA filters)".
- [ ] HANDOVER.md 10.1b: update the R1 and R2 NAT blocks.
- [ ] HANDOVER.md section 5: add a fix: "Port-only static NAT dropped scans at R1's Null0, so PA zone protection never saw them. Web NAT changed to 1:1 on R1/R2."
- [ ] TEST-PLAN.md 12.14: note the `--max-rate 80` requirement and the SYN cookie false positives.
- [ ] TEST-PLAN.md 14.1: the TCP Flood log above is the evidence.

---

# Part 4: Section 12 B2, B4 and C1 (PA log forwarding, Wazuh decoding, agent path)

## 4.1 B2: PA log forwarding (P20)

The `LFP-WAZUH` config entered on 7 Nov was already committed and complete.

| ID | Test | Result |
|---|---|---|
| 12.11 | PA: `show config running \| match LFP-WAZUH` | **PASS 9 Nov.** The profile plus 14 `log-setting LFP-WAZUH` lines = 11 security rules + DoS rule `DOS-DMZ-SERVERS` + zones `outside` and `dmz`. The zone lines rejected on 7 Nov are present, so no GUI step was needed |
| 12.12 | VM102: `sudo tcpdump -ni ens18 -A udp port 514 and host 172.16.0.41` while HR browses (YouTube) | **PASS 9 Nov.** TRAFFIC lines for rule `HR-IT-INTERNET` from `172.16.0.61` (app `ssl`). Lines for `INBOUND-WEB`, `INBOUND-DNS` and `DMZ-OUTBOUND-NTP` also arrive |
| 12.15 (auth) | One failed PA login | **PASS 9 Nov.** `SYSTEM,auth ... auth-fail ... "failed authentication for user 'bigDonkBonkAdmin'. Reason: Invalid username/password."` |
| 12.15 (config) | Small change + `commit` on the PA | Not captured yet (a `CONFIG` line is expected) |

Sample lines:
```
172.16.0.41 > 172.16.0.44.514: SYSLOG user.info
  ... TRAFFIC,end,...,172.16.0.61,74.125.68.100,...,HR-IT-INTERNET,,,ssl,vsys1,inside,outside,ethernet1/3,vlan.10,LFP-WAZUH,...

172.16.0.41 > 172.16.0.44.514: SYSLOG user.warning
  ... SYSTEM,auth,...,auth-fail,,0,0,general,medium,"failed authentication for user 'bigDonkBonkAdmin'. ..."
```

## 4.2 Finding from the PA logs: OSPF MD5 mismatch on the PA-DSW1 bypass

The forwarded PA system log showed, repeating:
```
SYSTEM,routing,...,routed-OSPF-md5chksum-bad,...,"OSPF packet dropped due to incorrect MD5 checksum. interface ethernet1/6, from IP 172.16.0.118."
```

### Diagnosis

| Check | Output |
|---|---|
| DSW1 `show ip ospf neighbor` | Only DSW2 (`4.4.4.4`, Vlan100) and ASA (`5.5.5.5`, Gi1/0/11). **PA missing on Gi1/0/12** |
| PA `show routing protocol ospf neighbor` | 4 neighbours (R1 `.90`, R2 `.91`, ASA `.106`, DSW2 `.122`). **DSW1 `.118` missing** |
| DSW1 `show run interface Gi1/0/12 \| include ospf` | `ip ospf message-digest-key 1 md5 7 02250D4808095E731F0F` |
| DSW1 `show version \| include uptime` | 3 days (no reload since before the 7 Nov hardening) |

The type 7 key decodes to **`Cisco123!`**, not the agreed key `GsOspf2026` (HANDOVER section 4). DSW1's other OSPF links were correct. The wrong key was entered on this one interface during the 7 Nov hardening, so the cost-500 PA-DSW1 bypass had been down since then. Normal traffic was unaffected, but this is the path users fail over to when the ASA fails (TEST-PLAN 9.4). The 7 Nov "all 9 links FULL" check missed it.

### Fix (DSW1)

IOS will not overwrite an existing key ID, so the old key is removed first:
```
interface GigabitEthernet1/0/12
 no ip ospf message-digest-key 1
 ip ospf message-digest-key 1 md5 GsOspf2026
```

### Result

```
DSW1#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           1   FULL/DR         00:00:39    172.16.0.126    Vlan100
6.6.6.6           1   FULL/DR         00:00:37    172.16.0.117    GigabitEthernet1/0/12
5.5.5.5           1   FULL/DR         00:00:39    172.16.0.97     GigabitEthernet1/0/11
```
**PASS 9 Nov.** The PA (`6.6.6.6`) is FULL on Gi1/0/12 again. Expected on the PA: 5 neighbours, and no more `md5chksum-bad` lines.

## 4.3 B4: Wazuh decodes the device logs (12.21)

`sudo /var/ossec/bin/wazuh-logtest` on VM102:

| Source | Line | Decoder | Rule | Result |
|---|---|---|---|---|
| R1 | `%SYS-5-CONFIG_I: Configured from console by bigDonkBonkAdmin on vty0 (172.16.0.44)` | `cisco-ios` (fields `cisco.facility SYS`, `cisco.mnemonic CONFIG_I`, `srcuser bigDonkBonkAdmin`, `srcip 172.16.0.44`) | **4721**, level 3, "Cisco IOS: Router configuration changed" (PCI DSS 10.2.7) | Alert generated |
| PA | `SYSTEM,auth ... auth-fail ... failed authentication for user 'bigDonkBonkAdmin'` | `paloalto` (fields `type SYSTEM`, `event_id auth-fail`, `severity medium`, `device_name PA-440`) | **64502**, level 3, "Palo Alto SYSTEM: medium event." | Alert generated |

**12.21: PASS 9 Nov.** In the dashboard (Threat Hunting > Events, filter `location:<device IP>`), ASA events appear under `location:172.16.0.42` as "ASA error message". That label comes from the ASA message severity (`%ASA-3-...` = "error"), not from a fault; the real message is in the event's `full_log` field.

Notes:
- The dashboard only stores events that match a rule at level 3 or above. Routine lines (most PA TRAFFIC logs) are decoded and dropped.
- `logall` (store every raw event) was deliberately **not** enabled: with the PA traffic logs it would fill VM102's disk (about 77% used).

## 4.4 C1: agent path through the ASA and PA (P17)

**ASA** (`configure terminal`), above the `SERVERS -> INTERNAL` deny:
```
access-list SERVER_IN line 1 extended permit tcp object SERVERS object JUMP-SVR range 1514 1515
```

**PA** (`configure`):
```
set service WAZUH-AGENT protocol tcp port 1514-1515
set rulebase security rules WAZUH-AGENTS from inside to dmz source SERVERS destination JUMP-SVR source-user any category any application any service WAZUH-AGENT action allow log-end yes log-setting LFP-WAZUH
move rulebase security rules WAZUH-AGENTS before DENY-ALL
validate full
commit
```
Commit succeeded (job 81). `set` only created the new objects `WAZUH-AGENT` and `WAZUH-AGENTS`; no existing rule was changed.

| ID | Test | Result |
|---|---|---|
| 12.23 | VM103: `nc -zv 172.16.0.84 1514-1515` | **PASS 9 Nov.** Both ports succeed |
| (note) | VM103: `nc -zv 172.16.0.85 1515` | Fails, as expected: `.85` is VM106, which runs no Wazuh manager. Agents only use the manager on VM102 (`.84`) |

## 4.5 Decision: no Wazuh agent on VM106

HANDOVER 12 C2 planned an agent on VM106 (server address `172.16.0.44`). **Decided 9 Nov: VM106 will not get a Wazuh agent.** Agents go on VM100, 101, 103, 104, 105 and 107 only (server address `172.16.0.84`). VM102 is covered by the manager's own local agent.

## 4.6 Records to update

- [ ] `dsw1.ttl`: search for `Cisco123!`. If it is on Gi1/0/12 (or anywhere), change it to `GsOspf2026`.
- [ ] HANDOVER.md section 5: add a fix: "DSW1 Gi1/0/12 OSPF key was `Cisco123!` instead of `GsOspf2026`; the PA-DSW1 bypass adjacency had been down since the 7 Nov hardening. Found from the PA's forwarded `routed-OSPF-md5chksum-bad` system log."
- [ ] HANDOVER.md 11.2: add to the OSPF check: count the neighbours on the PA (must be 5) and on each DSW (must be 3), not just "all FULL".
- [ ] HANDOVER.md 6: `LFP-WAZUH` is committed and complete (row can be closed); `WAZUH-AGENTS` rule and ASA `SERVER_IN` line 1 are live.
- [ ] HANDOVER.md 12: tick B2, B3 (PA part), B4, C1. Change C2: no agent on VM106.
- [ ] `pa440.ttl` (12 F1): add the `LFP-WAZUH` profile, match lists, every `log-setting` line, and the `WAZUH-AGENT` service and `WAZUH-AGENTS` rule before `DENY-ALL`.
- [ ] `firepower1010.ttl` (12 F2): add the Wazuh permit as line 1 of `SERVER_IN`.
- [ ] TEST-PLAN.md: fill in 12.11, 12.12, 12.15 (auth), 12.21, 12.23. Close P17 and P20.

---

# Part 5: TEMPORARY rule, jump host SSH to internal servers (REMOVE after C2)

**Why:** to install the Wazuh agents (12 C2), VM102 needs SSH to the internal servers (VM103, 104, 105, 107). By design the DMZ cannot reach the internal servers (TEST-PLAN 5.13, containment), so this is allowed **temporarily** only. The Proxmox console is the permanent way in. Like HANDOVER 8.4, these rules are deliberately **not** added to the macros.

**Path:** VM102 `.84` -> PA vlan.92 (zone `dmz`) -> PA eth1/3 (zone `inside`) -> ASA `outside` (E1/1) -> ASA `server` -> internal server. Both firewalls are stateful, so return traffic needs no rule.

## 5.1 Added (9 Nov)

**PA** (`configure`):
```
set rulebase security rules TEMP-JUMP-TO-SERVERS from dmz to inside source JUMP-SVR destination SERVERS source-user any category any application ssh service application-default action allow log-end yes log-setting LFP-WAZUH
move rulebase security rules TEMP-JUMP-TO-SERVERS before DENY-ALL
validate full
commit
```

**ASA** (`configure terminal`), on the ACL applied to `outside` (check with `show running-config access-group`):
```
access-list OUTSIDE_IN line 1 extended permit tcp object JUMP-SVR object SERVERS eq ssh
```
(If `JUMP-SVR` does not exist on the ASA, the line uses `host 172.16.0.84` instead; remove whichever form was entered.)

## 5.2 Remove (after every agent is Active, together with HANDOVER 8.4 / 12 D1)

**PA** (`configure`):
```
delete rulebase security rules TEMP-JUMP-TO-SERVERS
commit
```

**ASA** (`configure terminal`):
```
no access-list OUTSIDE_IN extended permit tcp object JUMP-SVR object SERVERS eq ssh
```

## 5.3 Check after removal

| Test | Expected |
|---|---|
| VM102: `ssh <user>@172.16.0.74` | **Fails** (times out) |
| PA: `show running security-policy` | No `TEMP-JUMP-TO-SERVERS`; `DENY-ALL` last |
| ASA: `show access-list OUTSIDE_IN \| include 172.16.0.84\|JUMP-SVR` | No ssh permit to `SERVERS` |
| TEST-PLAN 5.13 (Web VM: `nc -zv 172.16.0.74 445`) | Still fails |

## 5.4 Records

- [ ] HANDOVER.md 6 (live vs macros): add a row: "PA `TEMP-JUMP-TO-SERVERS` + ASA `OUTSIDE_IN` ssh permit: not in macro (on purpose), present until C2 is finished".
- [ ] HANDOVER.md 12 D1: add "remove `TEMP-JUMP-TO-SERVERS` (PA) and the `OUTSIDE_IN` ssh permit (ASA), see changes.md Part 5".

---

# Part 6: Section 12 C2, Wazuh agents installed

**Manager:** VM102, agents point to its DMZ address `172.16.0.84`. Version 4.14.10 (same as the manager). No agent on VM106 (decision in 4.5).

## 6.1 Install command (per server, run over SSH from VM102)

```bash
wget -q https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.10-1_amd64.deb && sudo WAZUH_MANAGER='172.16.0.84' WAZUH_AGENT_NAME='<name>' dpkg -i ./wazuh-agent_4.14.10-1_amd64.deb && sudo systemctl daemon-reload && sudo systemctl enable --now wazuh-agent
```

The agent enrols itself with the manager over tcp/1515 (no pre-registration in the dashboard is needed), then sends data over tcp/1514. Downloads used the temporary internet rules (HANDOVER 8.4). Internal servers were reached through the temporary SSH rule (Part 5).

| VM | IP | Agent name | Zone |
|---|---|---|---|
| 103 File | 172.16.0.74 | `file` | internal (via ASA + PA) |
| 100 Web | 172.16.0.82 | `web` | DMZ |
| 101 DNS | 172.16.0.83 | `dns` | DMZ |
| 104 Accounts | 172.16.0.75 | `accounts` | internal |
| 105 Txn | 172.16.0.76 | `txn` | internal |
| 107 DHCP | 172.16.0.77 | `dhcp` | internal |

## 6.2 Issue seen

VM103 first showed `ID: 001, Name: file, IP: any, Never connected`: enrolment over 1515 had worked, but the agent service was not running, so it never connected on 1514. Fixed by `sudo systemctl daemon-reload && sudo systemctl enable --now wazuh-agent` on VM103. The one-line command above includes this step.

## 6.3 Result

| ID | Test | Result |
|---|---|---|
| 12.22 | VM102: `sudo /var/ossec/bin/agent_control -l` | **PASS 9 Nov.** `file`, `web`, `dns`, `accounts`, `txn`, `dhcp` all Active, plus `000 tacacs (server)` Active/Local |

## 6.4 Records

- [ ] HANDOVER.md 12: tick C2 (note: no agent on VM106).
- [ ] TEST-PLAN.md: fill in 12.22.
- [ ] Next: D1 removes the temporary internet rules (8.4) **and** the temporary SSH rule (Part 5).

---

# Part 7: Section 12 C3 (agent alerts) and D1 deferred

## 7.1 C3 results

| ID | Test | Result |
|---|---|---|
| 12.24 | Outside laptop (lab Wi-Fi): 3 wrong passwords to `ssh -p 2222 bigDonkBonkAdmin@203.149.210.185` (lands on VM102) | **PASS 9 Nov.** Threat Hunting > Events shows "sshd: authentication failed" (rule 5760), agent `tacacs` |
| 12.25 | SW3 (`172.16.0.39`): `shutdown` / `no shutdown` on an unused access port | **PASS 9 Nov.** Event "Router configuration changed" (rule 4721) under `location:172.16.0.39` |

Section 12 C is complete: agents installed and Active (12.22), agent path open (12.23), agent and device alerts reach the dashboard (12.24, 12.25).

## 7.2 D1 deferred: temporary rules stay up for the Suricata install

**Decision 9 Nov:** HANDOVER 12 D1 (removing the temporary rules) is **postponed** until Suricata is installed, because the install needs the same internet access and SSH path.

Still live (do NOT forget):

| Rule | Device | Purpose | Removal |
|---|---|---|---|
| `SERVER_IN` www / https / udp 8.8.8.8 domain lines | ASA | Internal servers -> internet (apt) | HANDOVER 8.4 |
| `TEMP-SVR-UPDATES` | PA | Internal + DMZ servers -> internet | HANDOVER 8.4 |
| `NAT-SOURCES` lines `172.16.0.72 0.0.0.7` and `172.16.0.80 0.0.0.7` | R1, R2 | PAT for server installs | HANDOVER 8.4 |
| `TEMP-JUMP-TO-SERVERS` | PA | VM102 -> internal servers SSH | Part 5.2 |
| `OUTSIDE_IN` ssh permit `JUMP-SVR -> SERVERS` | ASA | VM102 -> internal servers SSH | Part 5.2 |

Remove all five after Suricata is installed, then run TEST-PLAN 8.5, 8.9 and the checks in Part 5.3. Stage 9 (redundancy) should wait until then, so it is tested on the final rules.

## 7.3 Records

- [ ] HANDOVER.md 12: tick C3. Mark D1 "postponed until Suricata is installed" and list the five rules above.
- [ ] TEST-PLAN.md: fill in 12.24 and 12.25.

---

# Part 8: Section 12 D2, D3 and Stage 10

## 8.1 D3 + P6: R1 `WAN-IN`

Both were already done on the live R1 before this session:
- Lines 10/20 deny SSH/Telnet to `172.17.9.93` (R1), not `.94` (P6 fixed live; TEST-PLAN 7.12 can be closed).
- The redundant `permit icmp any 203.149.210.184 0.0.0.7` line is gone (D3 done).

**Incident:** line 50 was removed by mistake (`no 50`), which dropped all HR/IT internet and all inbound web, DNS and jump SSH. Line 50 is the permit that lets return traffic and inbound services reach the public block. It was restored immediately:
```
ip access-list extended WAN-IN
 50 permit ip any 203.149.210.184 0.0.0.7
```
HR `ping 8.8.8.8` works again.

Correct live `WAN-IN` on R1 (9 Nov):
```
 1-9 anti-spoofing denies (bogons, own 172.16.0.0/24, own public block as source)
 10 deny tcp any host 172.17.9.93 eq 22 log
 20 deny tcp any host 172.17.9.93 eq telnet log
 30 permit icmp host 172.17.9.94 host 172.17.9.93 echo-reply
 50 permit ip any 203.149.210.184 0.0.0.7        <- MUST stay
 60 permit tcp any host 172.17.9.93 established
 70 deny ip any any log
```

## 8.2 D2: temporary ICMP permits removed (DSW1 and DSW2)

```
ip access-list extended ACL_HR_IN
 no permit icmp 172.16.0.56 0.0.0.7 any
ip access-list extended ACL_IT_IN
 no permit icmp 172.16.0.64 0.0.0.7 any
```

## 8.3 Stage 10 results (from the HR laptop)

| ID | Test | Expected | Result |
|---|---|---|---|
| 10.1 | `ping 172.16.0.68` (IT) | Fails | **PASS 9 Nov** (fails) |
| 10.2 | `ping 172.16.0.57`, `.97`, `.90` | All fail | **PASS 9 Nov** (all fail) |
| 10.3 | `ping 8.8.8.8` | Works | **PASS 9 Nov** (after line 50 restored) |
| 10.4 | `ping 172.16.0.74` + `H:` share | Works | To confirm |
| 10.5 | Browsing, `nslookup google.com` | Works | To confirm |

## 8.4 Records

- [ ] `dsw1.ttl`, `dsw2.ttl`: delete the two `TODO (FINAL HARDENING)` ICMP permit lines.
- [ ] HANDOVER.md 6: close the "Temporary ICMP permits on DSW" row and the R1 `WAN-IN` ICMP row. 11.1: mark 3.3 Done.
- [ ] HANDOVER.md 12: tick D2, D3. Add a warning to 11.4 / section 4: "R1 `WAN-IN` line 50 (`permit ip any 203.149.210.184 0.0.0.7`) must never be removed."
- [ ] TEST-PLAN.md: fill in 10.1-10.3 (10.4, 10.5 after checking); 7.12 / P6 closed.

---

# Part 9: Section 12 E1 and Stage 11A (AAA with TACACS up)

## 9.1 E1: re-checks from the 6 Nov run

| ID | Test | Result |
|---|---|---|
| 0.18 | R1 `show ip nat translations` while HR browses | **PASS 9 Nov.** HR `172.16.0.61` translated to `203.149.210.185`, plus the static entries |
| 6.5 | Outside laptop: `tracert -d 203.149.210.189` and `203.149.210.187` | **PASS 9 Nov.** Both end at R1 `172.17.9.93`, then `*`. No TTL-expired loop (`.189` is now 1:1 to the web server and dropped by the PA; `.187` hits Null0) |
| 6.9 | Another rack's user browses `https://gruppesechs.sitict.net` | **PASS 9 Nov** |
| 7.12 | R1 `WAN-IN` lines 10/20 | **PASS 9 Nov.** Both reference `172.17.9.93` (P6 fixed live) |

## 9.2 Stage 11A on R1 (`172.16.0.33`)

| Check | Result |
|---|---|
| Admin `bigDonkBonkAdmin` | **PASS.** `Current privilege level is 15` |
| Engineer `donkBonkEng1`, allowed | **PASS.** Lands in `eng-view` (`show priv`: "Currently in View Context with view 'eng-view'"); `show ip interface brief`, `show ip route` work |
| Engineer, refused | **PASS.** `router ospf 1` and `reload` refused with `% Invalid input detected` |
| Wrong password (`donkBonkEng2`) | **PASS.** Rejected |
| 11.11 accounting | **PASS.** `/var/log/tac_plus-ng/acct.log` lists the engineer's login and commands with user, device IP and time |

Notes for TEST-PLAN wording:
- On IOS the refusals come from the **parser view** `eng-view` (`% Invalid input`), before TACACS is asked. So 11.12 ("refused commands appear as authorization denials in the tac_plus log") does not apply to IOS; change the expected result to "refused by `eng-view`".
- `ping 172.16.0.33` from R1 fails because `.33` is in VRF OOB. Use `ping vrf OOB 172.16.0.33`. Not an AAA fault.
- The log paths in 11.11/11.12 are out of date: use `/var/log/tac_plus-ng/acct.log` and `/var/log/tac_plus-ng/authz.log`.

Other IOS devices use the same profile and view and were not re-tested individually.

## 9.3 Stage 11A on the ASA (`172.16.0.42`): two problems found and fixed

### Problem 1: engineers could not get past the `>` prompt

The engineer logged in at `ASA-1010>` and `enable` returned `Access denied` for every password tried.

Cause: since Part 1, `enable` is checked by TACACS (`aaa authentication enable console TACACS LOCAL`). Only `bigDonkBonkAdmin` has an `enable =` line in `tac_plus-ng.cfg`, so TACACS answers "fail" for engineers, and a TACACS "fail" never falls back to LOCAL. (Before Part 1, `enable` used the shared ASA enable password and turned the session into `enable_15`.)

Fix: let the ASA apply the privilege level that tac_plus already returns at login (`manager` = 15, `engineer` = 7), and put the user straight into privileged mode:
```
aaa authorization exec authentication-server auto-enable
```
Result: the admin lands at `ASA-1010#` with privilege 15, the engineer at `#` with privilege 7, no `enable` needed. `authz.log` shows the login authorizations:
```
172.16.0.42  bigDonkBonkAdmin  ...  manager   permit  shell
172.16.0.42  donkBonkEng1      ...  engineer  permit  shell
```

### Problem 2: the engineer could run every command, including `reload`

`authz.log` showed login authorizations but no per-command lines: the ASA was not asking TACACS about commands. **Command authorization was still off** since it was disabled for the Part 1 unlock (Part 1 verification V3 had never been completed). On the ASA, without command authorization, anyone in privileged mode can run everything regardless of privilege level.

Fix:
```
aaa authorization command TACACS LOCAL
```

Result (`authz.log`):
```
16:31:19  172.16.0.42  bigDonkBonkAdmin  manager   permit  show curpriv
16:31:21  172.16.0.42  bigDonkBonkAdmin  manager   permit  configure terminal
16:31:38  172.16.0.42  donkBonkEng1      engineer  permit  show route
16:31:42  172.16.0.42  donkBonkEng1      engineer  permit  configure terminal
16:31:44  172.16.0.42  donkBonkEng1      engineer  deny    reload
```
**PASS 9 Nov.** Admin: everything permitted. Engineer: `show` and `configure terminal` permitted, `reload` denied. This also completes Part 1 verification V3.

### Local password change

The first SSH login as `donkBonkEng1` triggered the ASA's local password policy ("This is your first login. Please set a new password"). The engineer's **local** ASA password was changed and no longer matches `plan.md`; it is recorded in private notes. It is only used when TACACS is unreachable (11B). The admin's local ASA password was already different from `plan.md` (TEST-PLAN 7.5).

### Open decision

The `engineer` profile permits any `show ...`, which includes `show running-config` (TEST-PLAN expects it refused on the ASA). Optional fix, on **both** VM102 and VM106, inside `profile engineer` above the `show|ping|...` line:
```
                if (cmd =~ /^show running-config/) deny
```
then `sudo /usr/local/sbin/tac_plus-ng -P /etc/tac_plus-ng.cfg && sudo systemctl restart tac_plus-ng`. Not applied yet.

## 9.4 Final ASA AAA lines (live, 9 Nov)

```
aaa authentication serial console TACACS LOCAL
aaa authentication enable console TACACS LOCAL
aaa authorization exec authentication-server auto-enable
aaa authorization command TACACS LOCAL
```
(plus the existing SSH authentication and `max-fail 5` lines)

## 9.5 Records

- [ ] `firepower1010.ttl`: add `aaa authorization exec authentication-server auto-enable` next to the other `aaa` lines, and keep `aaa authorization command TACACS LOCAL` **after** the two `aaa authentication ... console` lines.
- [ ] HANDOVER.md section 5: add a fix: "ASA engineers stuck at `>` (no TACACS enable for engineers): `aaa authorization exec authentication-server auto-enable`. Command authorization had been left off after the Part 1 unlock: re-enabled."
- [ ] HANDOVER.md section 4: local ASA passwords for `bigDonkBonkAdmin` and `donkBonkEng1` differ from `plan.md` (changed at first login); keep them in private notes. `donkBonkEng2` will be forced to change at its first ASA login.
- [ ] TEST-PLAN.md: fill in 0.18, 6.5, 6.9, 7.12, 11.1, 11.10, 11.11; apply the wording notes in 9.2; decide on `show running-config` (9.3).

---

# Part 10: Stage 11B (TACACS down, local fallback)

ASA console kept open throughout. Run from VM102.

| ID | Test | Result |
|---|---|---|
| 11.20 | VM102: `sudo systemctl stop tac_plus-ng`, then SSH to R1 as `bigDonkBonkAdmin` with the TACACS password | **PASS 9 Nov.** Login works through VM106. VM106 `authc.log`: `08:35:43 +0000 172.16.0.33 bigDonkBonkAdmin ... shell login succeeded`. No delay: VM102 is up with only the service stopped, so the connection to `.44` is refused at once and R1 moves straight to `.45`. (R1 `show tacacs` still shows `.44` "Alive" because no `deadtime` is configured; it does not mean `.44` answered) |
| 11.21 | Also stop tac_plus on VM106; SSH to R1 as `bigDonkBonkAdmin` | **PASS 9 Nov.** Local account logs in (no TACACS banner) |
| 11.22 | DSW1, SW1 local login | Not run (same IOS AAA config as R1) |
| 11.23 | PA local login with both servers down | **PASS 9 Nov** |
| 11.24 | ASA local login with both servers down: `show curpriv`, `configure terminal` | **PASS 9 Nov.** `Username : bigDonkBonkAdmin`, privilege 15, `P_PRIV`; `configure terminal` works (command authorization falls back to LOCAL when TACACS is unreachable). `show version`: `up 3 days 2 hours` (the engineer's earlier `reload` in 9.3 never ran) |
| 11.25 | Local `donkBonkEng1` on R1 | **PASS 9 Nov.** Lands in `eng-view`; `show ip route` works; `router ospf 1` refused (`% Invalid input`); `enable` returns `% Access denied` (engineers get no enable, as intended) |
| 11.26 | PA lockout, 5 wrong passwords | Not run (optional) |
| 11.27 | ASA lockout, 6 wrong passwords | Not run (optional) |
| 11.28 | Start tac_plus on VM102 and VM106; ASA `aaa-server TACACS active host .44/.45`; TACACS login again | **PASS 9 Nov.** Both servers already `ACTIVE` on the ASA. ASA login at 16:42 used TACACS (`.44` last transaction 16:42:45) |

## 10.1 Extra check: ASA through VM106

The ASA's `show aaa-server TACACS` showed `.45` with 3 authentication requests, 0 accepts, 3 rejects. Direct test:
```
ASA# test aaa-server authentication TACACS host 172.16.0.45 username bigDonkBonkAdmin password ****
INFO: Authentication Successful
```
VM106 `authc.log`: `08:44:19 +0000 172.16.0.42 bigDonkBonkAdmin ... shell login succeeded`. **PASS 9 Nov.** VM106 works as the ASA's backup TACACS server. The 3 earlier rejects were most likely the engineer's `enable` attempts (TACACS rejects enable for engineers by design, see 9.3).

## 10.2 Notes

- VM106 logs in **UTC**, VM102 in **SGT** (`+0800`). Set the same timezone on both (`sudo timedatectl set-timezone Asia/Singapore` on VM106) so the TACACS logs line up.
- `bigDonkBonkAdmin` has no sudo on VM102/VM106; use `ubuntu` for service control.
- The TACACS admin password was typed at an `R1>` prompt by mistake during 11.25 (rejected as a command). Already on the rotation list.

## 10.3 Records

- [ ] TEST-PLAN.md: fill in 11.20, 11.21, 11.23, 11.24, 11.25, 11.28. Mark 11.22, 11.26, 11.27 as not run (optional). Add the `test aaa-server ... host 172.16.0.45` check to 11.14.
- [ ] HANDOVER.md 12: tick E3 (Stage 11).

---

# Part 11: Stage 13B (DNS exposure) and notes for 13A / DEMO-DAY

## 11.1 Stage 13B results (outside laptop, lab Wi-Fi, against `203.149.210.190`)

| ID | Test | Expected | Result |
|---|---|---|---|
| 13.10 | `dig @203.149.210.190 google.com` | `status: REFUSED` | **PASS 9 Nov** |
| 13.11 | `dig @203.149.210.190 gruppesechs.sitict.net +short` | `203.149.210.189` | **PASS 9 Nov** |
| 13.12 | `dig @203.149.210.190 gruppesechs.sitict.net AXFR` | `Transfer failed` / `REFUSED` | **PASS 9 Nov** |
| 13.13 | `dig @203.149.210.190 version.bind chaos txt +short` | `"not disclosed"` | **PASS 9 Nov** |

## 11.2 Stage 13A postponed

13A (TXN window) cannot be tested meaningfully while the temporary rules are live (Part 7.2): ASA `SERVER_IN` www/https and PA `TEMP-SVR-UPDATES` let **every** server, including VM105, reach the internet at any time. So 13.2 (outside the window: fails) and 13.3 (VM103/104: fail) would give false results. Run 13A after D1.

## 11.3 DEMO-DAY.md corrections found today

| Section | Current text | Correction |
|---|---|---|
| 2 | "Re-enable ASA `aaa authorization command TACACS LOCAL`" | Done 9 Nov (Part 9.3); keep only as a check |
| 2 | "Remove the temporary ICMP permits on both DSWs" | Done 9 Nov (Part 8.2) |
| 2 | Temporary internet rules | Still live until Suricata is installed; there is also the temporary SSH rule (Part 5) |
| 3, Scene 4 | `nmap -sS -Pn 203.149.210.189` | Use `nmap -sS -Pn --max-rate 80 -p 1-1000 203.149.210.189`. At full speed, SYN flood protection (Activate 200/s) engages SYN cookies first: the scan is not detected and many ports falsely show `open` (Part 3.5) |
| 3, Scene 4 | Show the block | Also show Monitor > Block IP List in the GUI, from a device other than the scanner |
| 3, Scene 6 | `configure terminal -> rejected` | IOS engineer: `configure terminal` is **allowed**, then `router ospf 1` / `reload` refused with `% Invalid input` (parser view `eng-view`). ASA engineer: `configure terminal` allowed, `reload` refused with "Command authorization failed" (Part 9) |
| 3, Scene 6 | `sudo tail -n 20 /var/log/tac_plus.acct` | `sudo tail -n 20 /var/log/tac_plus-ng/acct.log` (and `authz.log` for the `deny` lines, which only appear for the ASA) |
| 3, Scene 6 | "Command authorization failed" | Only on the ASA. Demo it on the ASA for that message, or on R1 for the parser-view refusal |
| 2 | Record the new ASA password | Both `bigDonkBonkAdmin` and `donkBonkEng1` local ASA passwords changed (Part 9.3) |

---

# Part 12: Stage 13C (self-audit) and D1 (temporary rules removed)

## 12.1 Stage 13C results

| ID | From | Test | Result |
|---|---|---|---|
| 13.24 | VM102 | `nmap -Pn -p 22,23,80,443,830 172.16.0.33-42` | **PASS 9 Nov.** R1, R2, DSW1, DSW2, SW1-SW4 (`.33`-`.40`) and ASA (`.42`): only `22` open, the rest `closed`. PA (`.41`): `22` and `443` open, the rest `filtered` |
| 13.25 | VM102 | `nmap -Pn -p 22,8006 172.16.0.43` | **PASS 9 Nov.** `22` and `8006` open (Proxmox) |
| 13.23 | HR laptop | nmap to the OOB range | Not run (no nmap on the HR laptop). Equivalent: `Test-NetConnection 172.16.0.35 -Port 22` / `172.16.0.43 -Port 8006` must be `False` (3.8 already passed on 6 Nov) |
| 13.21 | Outside laptop | `nmap -Pn -sU -p 53 203.149.210.190` | **PASS 9 Nov** |
| 13.22 | Outside laptop | `ping 203.149.210.189`, `ping 203.149.210.186` | **PASS 9 Nov** (no reply) |
| 13.20 | Outside laptop | `nmap -Pn -sS --max-rate 80 -p 22,23,53,80,443,2222 203.149.210.185-190` | **PASS 9 Nov.** Only `.185:2222`, `.189:80/443`, `.190:53` open |

## 12.2 D1: temporary rules removed (9 Nov)

All five rules listed in Part 7.2 were removed after the Suricata downloads finished.

**ASA** (before removal, the temporary lines were `SERVER_IN` lines 2-4 and `OUTSIDE_IN` line 1):
```
no access-list SERVER_IN extended permit tcp object SERVERS any eq www
no access-list SERVER_IN extended permit tcp object SERVERS any eq https
no access-list SERVER_IN extended permit udp object SERVERS host 8.8.8.8 eq domain
no access-list OUTSIDE_IN extended permit tcp object JUMP-SVR object SERVERS eq ssh
```
Kept (permanent): `SERVER_IN` line 1 (Wazuh agents to `JUMP-SVR` tcp/1514-1515) and the three `TXN-SVR` lines with `time-range TXN-WINDOW`.

**PA:**
```
delete rulebase security rules TEMP-SVR-UPDATES
delete rulebase security rules TEMP-JUMP-TO-SERVERS
commit
```

**R1 and R2:**
```
ip access-list extended NAT-SOURCES
 no permit ip 172.16.0.72 0.0.0.7 any
 no permit ip 172.16.0.80 0.0.0.7 any
```
Kept: HR, IT, `host 172.16.0.76`, `.83`, `.84`, `.85`.

## 12.3 Checks after removal

| Test | Expected | Result |
|---|---|---|
| 8.5: VM103 and VM100 `sudo apt update` | Fails | **PASS 9 Nov** |
| 8.9: VM101 `dig @127.0.0.1 google.com +short` | Answer | **PASS 9 Nov** |
| Part 5.3: VM102 `ssh <user>@172.16.0.74` | Fails | **PASS 9 Nov** |
| VM102 `agent_control -l` | All 6 agents Active | **PASS 9 Nov** |
| VM102 `chronyc sources` | Pool servers reachable | **PASS 9 Nov** |
| HR laptop browsing | Works | **PASS 9 Nov** |

Internal servers are now reached through the Proxmox console only.

## 12.4 Records

- [ ] HANDOVER.md 8.4: mark the temporary rules as removed (9 Nov). Section 6: close the `TEMP-SVR-UPDATES` / `SERVER_IN` row.
- [ ] HANDOVER.md 12: tick D1 and E4 (13B, 13C; 13A next).
- [ ] TEST-PLAN.md: fill in 8.5, 8.9, 13.20-13.25.
- [ ] Note: `suricata-update` on a server without internet access will fail from now on.

---

# Part 13: Stage 13A (TXN window, live)

Run after D1, so the temporary rules could not give false results. Tested from the VM105, VM103 and VM104 Proxmox consoles with `curl -k -m 5 -s -o /dev/null -w "%{http_code}\n" https://8.8.8.8`.

| ID | Window set on both firewalls | Expected | Result |
|---|---|---|---|
| 13.1 | ASA `periodic daily 16:00 to 19:00`, PA `recurring daily 16:00-19:00` (covers now, ~17:10) | VM105 gets a status code; ASA `TXN` hit counts rise | **PASS 9 Nov** |
| 13.3 | Same | VM103 and VM104 time out (`000`) | **PASS 9 Nov** |
| 13.2 | ASA `periodic daily 19:00 to 21:00`, PA `recurring daily 19:00-21:00` (ends after now, starts later) | VM105 times out (`000`); ASA `show time-range TXN-WINDOW` inactive | **PASS 9 Nov** |
| 13.4 | Restored to the macro value: ASA `periodic weekdays 01:00 to 03:00`, PA `recurring weekly` Mon-Fri `01:00-03:00` | Both match | **PASS 9 Nov** |

Note: the window was briefly removed entirely (ASA time-range with no `periodic` line, PA schedule with no `schedule-type`). An empty ASA time-range can count as always active, which would give VM105 internet 24/7, and the PA rejects or mishandles an empty schedule. It was restored to the macro value straight away. The final window is set on demo day (DEMO-DAY.md section 1); never leave it empty.

## 13.1 Records

- [ ] TEST-PLAN.md: fill in 13.1-13.4 (also covers 8.1, 8.2).
- [ ] HANDOVER.md 12: E4 (Stage 13) complete.
- [ ] DEMO-DAY.md 1: add "never leave `TXN-WINDOW` empty: an empty ASA time-range may be always active".

---

# Part 14: Stage 9 (redundancy)

Run after D1/D2, on the final rules. HR laptop `ping 8.8.8.8 -t` running throughout. Cable unplugs only (9.1 is a `shutdown`). Each test restored before the next, with OSPF neighbour counts and `show standby brief` checked after every restore.

| ID | Action | Result |
|---|---|---|
| 9.1 | DSW2 `interface Vlan61` `shutdown` | **PASS 9 Nov.** DSW1 takes over VLAN 61, ping recovers |
| 9.2 | Unplug one SW3 port-channel member to DSW2 | **PASS 9 Nov** |
| 9.3 | Unplug R1 G0/0/1 (R1-PA) | **PASS 9 Nov** |
| 9.4a | Unplug DSW2 Gi1/0/11 (DSW2-ASA) | **PASS 9 Nov** |
| 9.4 | Unplug ASA E1/3 and E1/4 | **PASS 9 Nov** |
| 9.4b | Unplug PA eth1/3 (PA-ASA) | **PASS 9 Nov, slow.** Noticeably longer outage than the other tests; the HR ping showed `Reply from 172.16.0.59: Destination host unreachable` (DSW2's HR SVI with no default route) while OSPF reconverged. Two paths move at once (the ASA's default moves to R1 E1/5, and R1's return path to the user subnets probably moves to the PA + DSW bypass), so traffic can be briefly asymmetric across the two firewalls until it settles. Accepted for the lab; record the measured outage time |
| 9.6 | Unplug R1 G0/0/0 (ISP1) | **PASS 9 Nov (second attempt).** R1 `show track 1`: Down; R1 `show ip route 0.0.0.0`: `Known via "ospf 1" ... metric 60 ... from 2.2.2.2` (R2's default). The first attempt appeared to fail; most likely the test was checked before the track (IP SLA every 5 s + timeout) and OSPF had converged |
| 9.6b | During 9.6, outside laptop SSH to `129.126.142.185:2222` | **PASS 9 Nov** (through R2 and ISP2) |
| 9.6a | Unplug R1 G0/0/1 and G0/1/1 | **PASS 9 Nov** |
| 9.5 / 9.7 / 9.8 | Restore: default back on R1, neighbour counts, HSRP roles | **PASS 9 Nov.** Failback to ISP1 is slow (track has to see ISP1 again, then OSPF re-advertises, and existing flows stay on R2's NAT until they time out) |

## 14.1 Finding: R1 track 1 flaps

`show track 1` on R1 reported **87 changes** since the track was created. Only a few of those are from today's tests, so the IP SLA probe to ISP1 (`172.17.9.94`) has been timing out on and off. Each flap withdraws R1's default route and moves all user traffic to R2 and back, which causes short outages and makes failback look slow.

Suggested check and fix (not applied yet):
```
show ip sla statistics 1          ! number of failures, last return code
configure terminal
track 1 ip sla 1 reachability
 delay down 15 up 30
```
`delay down 15` ignores a single lost probe; `delay up 30` waits for ISP1 to be stable before moving traffic back. Apply the same on R2's track 1, and add both to `r1.ttl` / `r2.ttl` if kept.

## 14.2 Records

- [ ] TEST-PLAN.md: fill in 9.1-9.8 (note the 9.4b outage time and the 9.6 convergence wait).
- [ ] HANDOVER.md 12: tick E2.
- [ ] Decide on the track `delay` dampening (14.1).

---

# Part 15: Stage 14 (attack readiness)

Measures that normal users keep working while each attack runs. Attack commands from DEMO-DAY.md section 3 / ATTACKS.md.

## 15.1 Results so far

| ID | Attack | Evidence the attack ran and was mitigated | User impact | Result |
|---|---|---|---|---|
| 14.1 | SYN flood to `203.149.210.189:443` (Scene 4) | PA `show counter global filter delta yes \| match syn`: `flow_dos_syncookie_blk_dur` drop rate rose from 20/s to 610/s; `SynCookie sent 3953`; `curr-sess-incr-failed 21602`; `tcp-reject-non-syn 10868`. Wazuh: `TCP Flood` critical threat, location `172.16.0.41` | HR and outside `curl` to `.189` still `200` at baseline time | **PASS 9 Nov** |
| 14.2 | Slowloris to `https://gruppesechs.sitict.net` (Scene 5) | PA `show dos-protection rule DOS-DMZ-SERVERS statistics`: `sessions current 11, dropped 46374, ip tracked 3`. Wazuh: `DOS-DMZ-SERVERS` Session Limit Event | Second browser / curl still loads | **PASS 9 Nov** (VM100 `ss` / nginx error.log check still to capture; those run on VM100, not VM102) |
| 14.3 | DHCP starvation / MAC flood (Scene 1) | In progress | | |
| 14.4 | SSH brute force to `.185:2222` (Scene 6) | | | |

## 15.2 Recon auto-block (Scene 4 recon / 12.14): NOT achievable, documented

The PA TCP Port Scan "Block IP" could not be triggered despite four attempts:
- `nmap --max-rate 80`: tripped SYN flood protection (SYN cookies), scan masked
- `nmap --min-rate 150 --max-retries 1`: same
- SYN flood activate rate raised 200 -> 1000: cookies stopped, but no scan detected
- TCP Port Scan threshold lowered 100 -> 20, `nmap --max-rate 200`: still nothing

Root cause: inbound scans are NATed at R1 to `172.16.0.82`; filtered ports are dropped by `DENY-ALL`, so nmap slows down and the distinct-port count never crosses the threshold in the 2 s interval; background traffic to the public block narrows the usable rate window between the scan threshold (>50/s) and the flood activate rate. Decision: **not pursued further.** Perimeter DoS evidence (14.1 flood, 14.2 per-client) is sufficient. Present recon as "all ports filtered except 80/443" and note the flood-vs-scan threshold tension as a finding.

Zone-protection values during testing: SYN flood alarm 500 / activate 1000 / max 2000 (raised from 100/200/2000); TCP Port Scan threshold 20 (from 100). **Restore to sensible values before the demo** (SYN 100/200/2000; port-scan threshold 100), or keep the lower values and note them.

## 15.3 Side notes seen

- VM106 (`172.16.0.85`) -> `185.125.188.57` (Canonical) dropped by PA `DENY-ALL` (`policy-deny`): correct, now that the temporary internet rules are gone. VM106 was trying apt/ESM.
- One NTP `authentication failed from 172.16.0.44` seen in Wazuh; PA `show ntp` still `synched` with symmetric-key, so transient.

## 15.4 Records

- [ ] TEST-PLAN.md: fill in 14.1, 14.2; mark recon auto-block as a documented limitation.
- [ ] DEMO-DAY.md: restore zone-protection values, or note the tuned ones. Scene 4 recon: present as "ports filtered", not auto-block.

---

# Part 16: Stage 14 close-out

| ID | Attack | Result |
|---|---|---|
| 14.3 | DHCP starvation / MAC flood (Scene 1) | Not tested this session. L2 defences (DHCP snooping, port-security, DAI, errdisable recovery) already verified in Stage 1 and HARDENING.md 4.1-4.4 |
| 14.4 | SSH brute force to `.185:2222` (Scene 6) | **PASS 9 Nov.** Already verified via fail2ban (12.24): failed SSH logins raise Wazuh rule 5760, fail2ban `sshd` jail bans the source; legitimate IT login (in `ignoreip`) unaffected |

Stage 14 effectively complete: 14.1 (SYN flood) and 14.2 (Slowloris) passed with user-impact checks; 14.4 covered by fail2ban; 14.3 covered by earlier L2 testing. Recon auto-block documented as a limitation (Part 15.2).

## 16.1 Section 12 overall status (end of 9 Nov session)

| Step | Status |
|---|---|
| A (dashboard) | Done. Fixed `opensearch.hosts` missing port; passwords reset; wazuh.yml API password |
| B (log sources) | Done. Syslog into Wazuh (12.20, 12.1); PA log forwarding (12.11, 12.12, 12.15 auth); decoding (12.21) |
| C (agents) | Done. 6 agents Active (12.22); agent path open (12.23); alerts (12.24, 12.25). No agent on VM106 |
| D (cleanup) | Done. All temporary rules removed (D1); DSW ICMP permits removed (D2); R1 WAN-IN (D3) |
| E (tests) | Stage 11 AAA done; Stage 13 done; Stage 9 redundancy done; Stage 14 done bar the skipped L2 run |
| F (macros) | Outstanding, to do after the lab |
| H (demo prep) | Outstanding, to do after the lab |

## 16.2 Consolidated to-do (macros and docs, do after the lab)

Config still live-only (not yet in macros):
- `pa440.ttl`: `LFP-WAZUH` profile, match lists, all `log-setting` lines, `WAZUH-AGENT` service + `WAZUH-AGENTS` rule before `DENY-ALL`
- `firepower1010.ttl`: `SERVER_IN` line 1 Wazuh permit; `aaa authentication serial/enable console TACACS LOCAL`; `aaa authorization exec authentication-server auto-enable`; `aaa authorization command TACACS LOCAL`
- `r1.ttl` / `r2.ttl`: 1:1 NAT for the web server (replace port-80/443 statics)
- `dsw1.ttl`: confirm Gi1/0/12 OSPF key is `GsOspf2026` not `Cisco123!`; remove the TODO ICMP permit (also dsw2.ttl)

Fixes to add to HANDOVER section 5: #29 ASA console `enable_15`; ASA engineer auto-enable; DSW1 OSPF key; R1 NAT scan visibility; R1 WAN-IN line 50 must stay.

Follow-up, 9 Nov evening (PA console):
- `show running security-policy` showed `TEMP-JUMP-UPDATES` (index 12; `.84` -> outside, ssl/web-browsing) still live: the Suricata install rule had been missed by D1. **Deleted, commit job 99.** `TEMP-SVR-UPDATES` and `TEMP-JUMP-TO-SERVERS` were confirmed gone; `WAZUH-AGENTS` present; `DENY-ALL` the last custom rule.
- `ZP-OUTSIDE` restored to the macro values (SYN 100/200/2000, scan 8001 threshold 100), **commit job 101**. Confirmed: `show zone-protection zone outside` shows SYN cookies 100/200/2000; scans 8001/8002/8003 at threshold 100 (block-ip 300 s).
- Checks after removing `TEMP-JUMP-UPDATES`: VM102 `curl https://www.google.com` -> `000` (no web access, as intended); `chronyc sources` still synced to the pool (`^* kaguaani.miuku.net`, reach 377); all Wazuh agents Active.

- Port security on SW1-SW4 access ports changed live: sticky removed; `violation restrict`, `aging time 5`, `aging type inactivity`. Added to `sw1-4.ttl`.
- `runshow.md` re-captured (~21:40, all 10 devices) and audited against the macros. Macro gaps found and fixed: `pa440.ttl` lacked the PA admin authentication profiles/sequence (`AUTH-TACACS`, `AUTH-LOCAL`, `SEQ-ADMIN`), `timezone Asia/Singapore` and `hostname-type-in-syslog FQDN`. Live drifts found (macros already correct): DSW2 Vlan46 HSRP key `GsOspf2026` instead of `GsHsrp2026`; R2 `WAN-IN` line 9 has R1's public block; SW3 Gi1/0/10 not shut; ASA `ntp trusted-key 1` missing; ASA `INSIDE_IN` HR-IT -> WEB-SVR `eq www` permit sits after the final deny (HTTP to the web server denied from HR/IT); R1 console logging on. (ASA also has the school's `route management 0.0.0.0 0.0.0.0 172.20.11.254`; harmless, the next hop isn't on the management subnet.) Fix commands in HANDOVER section 6. **All live drifts fixed the same night.**
- TEST-PLAN 10.4 and 10.5 pass; the post-fix checks (HTTP to `.82`, HSRP roles, ASA NTP authenticated) pass.
- All 11 macros re-commented for submission: no dates, document references or issue numbers; short explanations of what each section does and why. Command lines verified unchanged by script.

Credentials changed this session (rotate and record privately, keep out of repo): Wazuh admin/kibanaserver/wazuh-wui; ASA local passwords for bigDonkBonkAdmin and donkBonkEng1; and the TACACS/engine passwords exposed during troubleshooting.

Other: fix SSH hardening filename on VM102 (`99-hardening-conf` -> `99-hardening.conf`); set VM106 timezone to Asia/Singapore; restore PA zone-protection values before the demo.

Not adopted (by decision): R1/R2 track `delay` dampening; agent on VM106; `show running-config` deny for engineers (optional, left open).
