# Missing Defenses: Commands and Where to Put Them

Written 6 Nov 2026 from a comparison of the `.ttl` macros against `REVISION.docx` and `revision 2.md`.

Each item is tagged **[notes]** (covered in your lab notes) or **[extra]** (not in your notes, but a real gap in this network).

## How to use this file

- **Cisco and ASA wipe on reload.** Paste every Cisco/ASA block live now **and** add it to the macro, or it's gone after the next reload.
- **PA keeps its committed config.** Add the lines to `pa440.ttl` (so a rebuild has them) and paste them live, then `commit`.
- In a Cisco/ASA macro, each CLI line becomes:
  ```
  sendln '<command>'
  wait '#'
  ```
  In `pa440.ttl`, use `wait '[edit]'` instead of `wait '#'`.
- "After `X`" means: insert the new lines directly below the existing `sendln 'X'` / `wait` pair in that macro.

### Shared keys used below

| Purpose | Key | Where it must match |
|---|---|---|
| NTP | `GsNtpKey2026` | chrony on VM102 + VM106, all Cisco devices, ASA, PA |
| OSPF MD5 | `GsOspf2026` (OSPF MD5 keys max 16 characters) | Both ends of every OSPF link |
| HSRP MD5 | `GsHsrp2026` | DSW1 and DSW2 |

Change them if you like, but use the same value everywhere. They're cleartext in the repo, like the other passwords.

### Order and risk

| Order | Change | Risk if done wrong |
|---|---|---|
| 1 | Login protection, misc IOS hygiene, `no ip unreachables`, CDP, DHCP rate limit, errdisable recovery, storm control | Low |
| 2 | ASA MPF, threat detection, PA zone protection / DoS / OSPF passive, BIND, nginx, SSH/fail2ban | Low to medium (check the PA syntax, see section 6) |
| 3 | WAN-IN anti-spoofing, uRPF | Medium: a wrong line blocks inbound tests (read the 172.16/12 warning) |
| 4 | NTP authentication | Medium: a key mismatch stops time sync (the TXN window and log times depend on it) |
| 5 | OSPF and HSRP authentication | **High:** a mismatch drops adjacencies (outage) or makes both DSWs Active. Do this when you have time to fix it, not right before the demo. |

---

## 1. All Cisco IOS devices (R1, R2, DSW1, DSW2, SW1-SW4)

### 1.1 Login brute-force protection [notes: AAA part 1]

**Where:** in every IOS macro, in the AAA/SSH section, after `ip ssh authentication-retries 3` (before `line con 0`).

```
ip access-list standard QUIET-EXEMPT
 permit host 172.16.0.44
 deny any
exit
login delay 2
login block-for 120 attempts 3 within 60
login quiet-mode access-class QUIET-EXEMPT
login on-failure log
login on-success log
```

- After 3 failed logins within 60 s, the device refuses all SSH logins for 120 s, except from the hosts in `QUIET-EXEMPT`.
- `QUIET-EXEMPT` deliberately holds only **VM102 (.44)**. `MGMT-ONLY` already limits SSH to .44/.45. If quiet mode exempted both, it would never block anything that can reach the VTY lines. This way, brute force from VM106 is blocked while VM102 still gets in. The console isn't affected by `login block-for`.
- **Demo:** from VM106, SSH to a switch and fail the password 3 times. Then run `show login` on the device ("in Quiet-Mode"). SSH from VM102 still works.

### 1.2 Misc hygiene [extra]

**Where:** in every IOS macro, in "Base System Setup", after `service timestamps log datetime msec localtime`.

```
no ip source-route
no service pad
service tcp-keepalives-in
service tcp-keepalives-out
logging buffered 64000 informational
```

`tcp-keepalives-in` clears dead SSH sessions off the 3 VTY lines (vty 0 2), so stale sessions can't lock you out. `logging buffered` keeps logs on the device if the syslog servers are down (`show logging`).

### 1.3 NTP authentication and NTP ACL [notes: NTP]

Do chrony first (section 8), then one device, check it, then the rest.

**Where:** in every IOS macro, **replace** the two existing `ntp server ...` lines with the following.

R1, R2, DSW1, DSW2 (OOB VRF):
```
ntp authentication-key 1 md5 GsNtpKey2026
ntp trusted-key 1
ntp authenticate
ntp server vrf OOB 172.16.0.44 key 1 prefer
ntp server vrf OOB 172.16.0.45 key 1
ip access-list standard NTP-SERVERS
 permit host 172.16.0.44
 permit host 172.16.0.45
 deny any
exit
ntp access-group peer NTP-SERVERS
ntp logging
```

SW1-SW4 (no VRF): the same, but the two server lines are:
```
ntp server 172.16.0.44 key 1 prefer
ntp server 172.16.0.45 key 1
```

- `ntp access-group peer NTP-SERVERS` means the device only accepts time from (and answers NTP to) .44/.45.
- **Check:** `show ntp associations detail`. The line for .44 should say `authenticated` and `sane`. `show ntp status` should say `synchronized`.
- **If it won't sync:** the key differs from chrony's. Rolling back with `no ntp authenticate` restores sync straight away.

---

## 2. R1 and R2 (edge routers)

### 2.1 WAN-IN anti-spoofing (RFC 2827 / RFC 3330) [notes: lab 2.4]

> **Warning: don't deny `172.16.0.0/12`.** The lab "internet" uses 172.17.x and 172.18.x: both ISP gateways (`172.17.9.94`, `172.17.10.94`), the ICT DNS `172.18.4.32` and the lab Wi-Fi resolver `172.18.18.36`. Denying /12 cuts off IP SLA, DNS forwarding and inbound tests. Only your own `172.16.0.0/24` is denied below.
>
> **Check first:** on the lab Wi-Fi laptop you test inbound with, run `ipconfig`. If its address is in `10.x` or `192.168.x`, leave out that line, or your inbound web/DNS/SSH tests will fail.

**Live (R1):** named ACL entries are numbered 10, 20, 30...; confirm with `show ip access-lists WAN-IN`. Numbers 1-9 put the new lines above all of them.
```
ip access-list extended WAN-IN
 1 deny ip 0.0.0.0 0.255.255.255 any log
 2 deny ip 10.0.0.0 0.255.255.255 any log
 3 deny ip 127.0.0.0 0.255.255.255 any log
 4 deny ip 169.254.0.0 0.0.255.255 any log
 5 deny ip 172.16.0.0 0.0.0.255 any log
 6 deny ip 192.0.2.0 0.0.0.255 any log
 7 deny ip 192.168.0.0 0.0.255.255 any log
 8 deny ip 224.0.0.0 31.255.255.255 any log
 9 deny ip 203.149.210.184 0.0.0.7 any log
 no permit icmp any 203.149.210.184 0.0.0.7
exit
```
**Live (R2):** the same lines 1-8. Line 9 is `9 deny ip 129.126.142.184 0.0.0.7 any log`. R2 has no `permit icmp any <block>` line to remove.

**Macro (`r1.ttl`, `r2.ttl`):** put the same 9 deny lines (without the numbers) straight after `sendln 'ip access-list extended WAN-IN'`, before `deny tcp any host ... eq 22 log`. In `r1.ttl`, also delete the `permit icmp any 203.149.210.184 0.0.0.7` line. It's redundant because the next line permits all IP to the block.

Line 9 drops packets from the internet that claim to come from your own public block, which is exactly the lab 2.4 smurf case. Line 8 covers multicast and the reserved 240/4 range.

### 2.2 uRPF, no ICMP unreachables, no CDP to the ISP [notes: lab 2.3 / BCP 38] + [extra]

**Where:** `interface GigabitEthernet0/0/0` block (WAN), after `ip access-group WAN-IN in`. Same on both routers.
```
interface GigabitEthernet0/0/0
 ip verify unicast source reachable-via rx allow-default
 no ip unreachables
 no cdp enable
exit
interface Null0
 no ip unreachables
exit
```
- uRPF strict mode with `allow-default` passes internet sources (they match the default route out G0/0/0). It drops sources whose route points elsewhere, such as your inside 172.16.0.x (learned via OSPF on G0/0/1) or your public block (Null0). Without `allow-default`, strict mode drops all internet traffic.
- `no ip unreachables` stops scan probes getting "port unreachable" answers, so nmap reports `filtered` (lab 2.3).
- CDP on the WAN tells the ISP side your hostname, model and IOS version.

### 2.3 Null0 route for the public block [extra, missing from both macros]

HANDOVER section 5 and 10.1b list it, but neither `r1.ttl` nor `r2.ttl` has it. Without it, packets for unused public IPs are sent back to the ISP by the default route and loop until the TTL expires.

**Where:** after `ip route 0.0.0.0 0.0.0.0 ... track 1`.
```
R1:  ip route 203.149.210.184 255.255.255.248 Null0
R2:  ip route 129.126.142.184 255.255.255.248 Null0
```
The NAT static mappings still work: NAT translates before routing, so .185/.189/.190 are never routed to Null0.

**Check (2.1-2.3):** `show ip access-lists WAN-IN` (hit counters), `show ip interface G0/0/0 | include verify|unreach`, and `show ip traffic | include RPF`. HR/IT internet, inbound HTTPS/DNS/2222 and `show track 1` must all still work.

---

## 3. DSW1 and DSW2

### 3.1 HSRP MD5 authentication [extra]

`ACL_STAFF_IN` / `ACL_ATM_IN` permit HSRP hellos from the **whole** user subnet (needed for the peer DSW). Without authentication, any Staff or ATM host can send a priority-255 hello and become the gateway (a man-in-the-middle).

**Where:** in each `interface VlanXX` block, after `standby XX preempt`. Do this on **both** DSWs.
```
interface Vlan23
 standby 23 authentication md5 key-string GsHsrp2026
interface Vlan46
 standby 46 authentication md5 key-string GsHsrp2026
interface Vlan61
 standby 61 authentication md5 key-string GsHsrp2026
interface Vlan67
 standby 67 authentication md5 key-string GsHsrp2026
```
**Live:** paste on DSW1, then straight away on DSW2. Between the two, both switches are briefly Active. **Check:** `show standby brief` shows one Active and one Standby per VLAN. `show standby | include Authentication` shows MD5.

### 3.2 OSPF MD5 on DSW links — see section 7.

### 3.3 Final-hardening items already tracked in HANDOVER [notes]

These are the `TODO (FINAL HARDENING)` lines. Remove them on both DSWs when testing is done, and delete the same lines from `dsw1.ttl` / `dsw2.ttl`:
```
ip access-list extended ACL_HR_IN
 no permit icmp 172.16.0.56 0.0.0.7 any
ip access-list extended ACL_IT_IN
 no permit icmp 172.16.0.64 0.0.0.7 any
```

---

## 4. SW1-SW4 (access switches)

User port ranges: **SW1** `Gi1/0/5 - 16`, **SW2** `Gi1/0/5 - 7`, **SW3** `Gi1/0/5 - 6`, **SW4** `Gi1/0/5 - 7`.

### 4.1 DHCP snooping rate limit [notes: lab 1.5]

**Where:** in the end-user access-port block, after `ip arp inspection limit rate 15`.
```
ip dhcp snooping limit rate 10
```
A DHCPig starvation flood exceeds 10 packets per second and err-disables the port.

### 4.2 Errdisable auto-recovery [notes: lab 1.4]

**Where:** global, after `ip arp inspection vlan XX` (section 4 of each macro).
```
errdisable recovery cause psecure-violation
errdisable recovery cause bpduguard
errdisable recovery cause dhcp-rate-limit
errdisable recovery cause arp-inspection
errdisable recovery interval 300
```

**About `maximum 5` (your choice, kept):** sticky MACs are written to the running-config and **never age out**. After the 5th different laptop, the 6th shuts the port. Errdisable recovery brings it back after 300 s, but the 6th laptop shuts it again. To free the slots, run:
```
clear port-security sticky interface GigabitEthernet1/0/5
```
(Or `clear port-security sticky` for all ports.) Check what's stored with `show port-security address`.

Also fix the stale comments: `sw3.ttl` line 142 and `sw4.ttl` line 142 say "Max 2 MAC". Change them to "Max 5 MAC".

### 4.3 DAI extra validation [extra]

**Where:** global, after `ip arp inspection vlan XX`.
```
ip arp inspection validate src-mac dst-mac ip
```
This also drops ARP packets whose ARP-body MAC doesn't match the Ethernet header, and ARPs with invalid IPs (0.0.0.0, broadcast, multicast).

### 4.4 Storm control and CDP on user ports [extra]

**Where:** in the end-user access-port block, after `ip dhcp snooping limit rate 10`.
```
storm-control broadcast level 5.00
storm-control multicast level 5.00
storm-control action trap
no cdp enable
```
Storm control limits a broadcast storm (or a macof-style flood) to 5% of the port. `action trap` logs it without shutting the port. `no cdp enable` stops a user laptop reading the switch model/IOS version from CDP.

### 4.5 IPSG with MAC check (optional) [notes: lab 1.7]

Your macros use `ip verify source` (IP only). Lab 1.7 also shows MAC checking:
```
ip verify source mac-check
```
This **replaces** `ip verify source` in the access-port block. Try it on one port first (`interface Gi1/0/5`), then check that a laptop still gets a DHCP lease and passes traffic (`show ip verify source`). If DHCP breaks, go back with `ip verify source`. Port-security already limits MAC spoofing, so this is optional.

---

## 5. ASA (Firepower 1010)

### 5.1 SYN flood + Slowloris limits for the internal servers [notes: labs 2.5, 2.6]

The **public** servers (web/DNS) sit behind the PA, not the ASA (section 6 covers them). On the ASA, this protects the internal servers (File, Accounts, Txn) from inside users.

**Where:** `firepower1010.ttl`, after section 12 (the `policy-map global_policy` block), before section 13.
```
access-list MPF_INTERNAL_SVRS extended permit tcp any object SERVERS
class-map CM_SVR_PROTECT
 match access-list MPF_INTERNAL_SVRS
exit
policy-map PM_INSIDE_PROTECT
 class CM_SVR_PROTECT
  set connection conn-max 500 embryonic-conn-max 50 per-client-max 50 per-client-embryonic-max 10
  set connection timeout embryonic 0:00:15
 exit
exit
service-policy PM_INSIDE_PROTECT interface inside
service-policy PM_INSIDE_PROTECT interface inside2
threat-detection basic-threat
threat-detection statistics tcp-intercept
```
- `embryonic-conn-max` / `per-client-embryonic-max` cover the SYN flood (lab 2.5). `per-client-max 50` covers Slowloris (lab 2.6).
- Interface policies work alongside `global_policy`, so ICMP inspection keeps working.
- **Check:** `show service-policy interface inside`, `show threat-detection statistics top tcp-intercept`.

### 5.2 Anti-spoofing on the server interface [extra]

**Where:** after the `access-group ... in interface server` line.
```
ip verify reverse-path interface server
```
This stops a compromised internal server sending with a fake source. It's **not** on `inside`/`inside2`: the user subnets can legitimately arrive on either DSW link during HSRP failover, and ASA uRPF is strict, so it would drop that traffic.

### 5.3 Local account lockout [notes: AAA, slow down brute force]

**Where:** section 13, after `aaa authentication enable console TACACS LOCAL`.
```
aaa local authentication attempts max-fail 5
```
After 5 failed logins, a local account is locked (privilege-15 accounts are exempt, so `bigDonkBonkAdmin` can't be locked out). Unlock with `clear aaa local user lockout username donkBonkEng1`.

### 5.4 NTP authentication [notes: NTP]

**Where:** section 13, **replace** the two `ntp server` lines.
```
ntp authentication-key 1 md5 GsNtpKey2026
ntp trusted-key 1
ntp authenticate
ntp server 172.16.0.44 key 1 source management prefer
ntp server 172.16.0.45 key 1 source management
```
**Check:** `show ntp associations detail` (authenticated), `show ntp status`.

### 5.5 Already tracked [notes]

After tac_plus is verified (P13), un-comment `aaa authorization command TACACS LOCAL`. Keep a console session open while you do.

---

## 6. PA-440

> **Syntax note:** the ASA/IOS commands above are standard. For the PAN-OS lines I wasn't able to confirm every keyword against the official CLI reference. Before committing, enter each line with `?` or Tab at the end of each level, the same way you found `reject-default-route`. If a keyword differs, the web UI path is given so you can do it there instead. Run `validate full` before `commit`.

### 6.1 Zone protection on `outside`: SYN cookies, floods, scans, spoofing [notes: labs 2.3, 2.5] + [extra]

This is the PA equivalent of the ASA `embryonic-conn-max` / TCP intercept + SYN cookies from your notes, plus port-scan/host-sweep blocking (lab 2.3). It protects web/DNS/jump traffic arriving from the internet.

**Where:** `pa440.ttl`, after section 5 (security zones). Web UI: Network > Network Profiles > Zone Protection.
```
set network profiles zone-protection-profile ZP-OUTSIDE flood tcp-syn enable yes
set network profiles zone-protection-profile ZP-OUTSIDE flood tcp-syn syn-cookies alarm-rate 100 activate-rate 200 maximal-rate 2000
set network profiles zone-protection-profile ZP-OUTSIDE flood udp enable yes
set network profiles zone-protection-profile ZP-OUTSIDE flood udp red alarm-rate 500 activate-rate 1000 maximal-rate 5000
set network profiles zone-protection-profile ZP-OUTSIDE flood icmp enable yes
set network profiles zone-protection-profile ZP-OUTSIDE flood icmp red alarm-rate 50 activate-rate 100 maximal-rate 500
set network profiles zone-protection-profile ZP-OUTSIDE scan 8001 action block-ip track-by source duration 300
set network profiles zone-protection-profile ZP-OUTSIDE scan 8001 interval 2 threshold 100
set network profiles zone-protection-profile ZP-OUTSIDE scan 8002 action block-ip track-by source duration 300
set network profiles zone-protection-profile ZP-OUTSIDE scan 8002 interval 10 threshold 100
set network profiles zone-protection-profile ZP-OUTSIDE scan 8003 action block-ip track-by source duration 300
set network profiles zone-protection-profile ZP-OUTSIDE scan 8003 interval 2 threshold 100
set network profiles zone-protection-profile ZP-OUTSIDE discard-ip-spoof yes
set zone outside network zone-protection-profile ZP-OUTSIDE
```
- 8001 = TCP port scan, 8002 = host sweep, 8003 = UDP port scan. A scanning source is blocked for 300 s.
- The rates count **new** sessions per second entering `outside`. Return traffic for HR/IT browsing doesn't count, so these low lab values won't affect users. An `hping3 --flood` or `tcpsynflood.py` run goes far over them.
- **Demo:** run `nmap -sS 203.149.210.189` from outside, then `show dos-protection zone outside blocked source` (or Monitor > Block IP List). For the SYN flood, use `show counter global filter delta yes | match syn`.

### 6.2 Zone protection on `dmz`: contain a compromised server [extra]

```
set network profiles zone-protection-profile ZP-DMZ discard-ip-spoof yes
set network profiles zone-protection-profile ZP-DMZ scan 8001 action block-ip track-by source duration 300
set network profiles zone-protection-profile ZP-DMZ scan 8001 interval 2 threshold 100
set network profiles zone-protection-profile ZP-DMZ scan 8002 action block-ip track-by source duration 300
set network profiles zone-protection-profile ZP-DMZ scan 8002 interval 10 threshold 100
set zone dmz network zone-protection-profile ZP-DMZ
```

### 6.3 Per-client session limit for the DMZ servers (Slowloris) [notes: lab 2.6]

This is the PA equivalent of ASA `per-client-max`. SYN flood is **not** enabled in this profile: PAN-OS shouldn't run SYN protection in both the zone profile and a DoS profile, and 6.1 already handles it. R1/R2 keep the real client source IP on inbound static NAT, so "per client" really is per client.

**Where:** after 6.1. Web UI: Objects > Security Profiles > DoS Protection, and Policies > DoS Protection.
```
set network profiles dos-protection DOS-PER-CLIENT type classified
set network profiles dos-protection DOS-PER-CLIENT resource sessions enabled yes
set network profiles dos-protection DOS-PER-CLIENT resource sessions max-concurrent-limit 50
set rulebase dos rules DOS-DMZ-SERVERS from zone outside
set rulebase dos rules DOS-DMZ-SERVERS to zone dmz
set rulebase dos rules DOS-DMZ-SERVERS source any destination [ WEB-SVR DNS-SVR JUMP-SVR ] source-user any service any
set rulebase dos rules DOS-DMZ-SERVERS action protect
set rulebase dos rules DOS-DMZ-SERVERS protection classified profile DOS-PER-CLIENT classification-criteria address source-ip-only
```
**Demo:** run `slowhttptest -c 1000 -u http://203.149.210.189` from outside. With this rule active, the PA caps that source at 50 sessions and the site stays up for others.

### 6.4 Stop OSPF on the DMZ interface [extra]

`vlan.92` is `passive no`, so a compromised web/DNS server could form an OSPF neighbor with the PA and inject routes (for example a fake default). Intrazone traffic is allowed by default, so `ALLOW-OSPF` doesn't stop it. Passive mode still advertises the DMZ subnet.

**Where:** in `pa440.ttl` section 6, change the `vlan.92` line from `passive no` to `passive yes`:
```
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.92 enable yes passive yes metric 10
```
**Check:** `show routing protocol ospf neighbor` (no neighbor on vlan.92; R1/R2/ASA/DSW neighbors unchanged), and the ASA still has `172.16.0.80/29` via `.105`.

### 6.5 Admin lockout and idle timeout [notes: AAA, brute force]

**Where:** section 2, after the `permitted-ip` lines.
```
set deviceconfig setting management admin-lockout failed-attempts 5 lockout-time 5
set deviceconfig setting management idle-timeout 10
```

### 6.6 NTP authentication [notes: NTP]

**Where:** section 2, after the two `ntp-servers` lines. Web UI: Device > Setup > Services > NTP > Authentication Type: Symmetric Key, MD5.
```
set deviceconfig system ntp-servers primary-ntp-server authentication-type symmetric-key key-id 1 algorithm md5 authentication-key GsNtpKey2026
set deviceconfig system ntp-servers secondary-ntp-server authentication-type symmetric-key key-id 1 algorithm md5 authentication-key GsNtpKey2026
```
**Check:** `show ntp` shows `authentication-type: symmetric-key` and `status: synched`.

Then `commit`.

---

## 7. OSPF MD5 authentication: every OSPF link [extra]

High risk: an adjacency goes down until **both** ends have the same key. Do one link at a time, both ends straight after each other, and check `show ip ospf neighbor` before moving on. Do R1 + R2 + PA (shared `vlan.10` segment) together.

| Link | Ends |
|---|---|
| Routers <-> PA (vlan.10) | R1 `G0/0/1`, R2 `G0/0/1`, PA `vlan.10` |
| R1 <-> ASA | R1 `G0/1/1`, ASA `Ethernet1/5` |
| R2 <-> ASA | R2 `G0/1/1`, ASA `Ethernet1/2` |
| PA <-> ASA | PA `ethernet1/3`, ASA `Ethernet1/1` |
| ASA <-> DSW1 | ASA `Ethernet1/3`, DSW1 `Gi1/0/11` |
| ASA <-> DSW2 | ASA `Ethernet1/4`, DSW2 `Gi1/0/11` |
| PA <-> DSW1 | PA `ethernet1/6`, DSW1 `Gi1/0/12` |
| PA <-> DSW2 | PA `ethernet1/7`, DSW2 `Gi1/0/12` |
| DSW1 <-> DSW2 | DSW1 `Vlan100`, DSW2 `Vlan100` |

**IOS (R1, R2, DSW1, DSW2):** in each listed interface block, after `ip ospf cost ...`:
```
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 GsOspf2026
```

**ASA:** in each listed interface block, after `ospf cost ...`:
```
ospf authentication message-digest
ospf message-digest-key 1 md5 GsOspf2026
```

**PA:** after the OSPF interface lines in section 6 (Web UI: Network > Virtual Routers > default > OSPF > Auth Profiles):
```
set network virtual-router default protocol ospf auth-profile OSPF-MD5 md5 1 key GsOspf2026 preferred yes
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/3 authentication OSPF-MD5
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/6 authentication OSPF-MD5
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/7 authentication OSPF-MD5
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.10 authentication OSPF-MD5
```

**Check:** `show ip ospf interface G0/0/1 | include authentication` ("Message digest authentication enabled"), all neighbors FULL, `TEST-PLAN.md` Stage 0.

---

## 8. chrony on VM102 and VM106 (NTP server side) [notes: NTP]

Do this **before** section 1.3 / 5.4 / 6.6.

```bash
echo '1 MD5 GsNtpKey2026' | sudo tee /etc/chrony/chrony.keys
sudo chown root:_chrony /etc/chrony/chrony.keys
sudo chmod 640 /etc/chrony/chrony.keys
grep -q '^keyfile' /etc/chrony/chrony.conf || echo 'keyfile /etc/chrony/chrony.keys' | sudo tee -a /etc/chrony/chrony.conf
sudo systemctl restart chrony
sudo ss -ulpn | grep chronyd      # must still show 0.0.0.0:123
```
chrony answers with an authenticated reply automatically when a client asks with key 1. Clients without the key still get time until they turn on `ntp authenticate`, so this step on its own breaks nothing. Optional: add `key 1` to each VM's `server <other TACACS> iburst` line.

---

## 9. VM101 DNS (BIND) [notes: DNS amplification/reflection] + [extra]

**Where:** `/etc/bind/named.conf.options`, inside `options { }`:
```
version "not disclosed";
allow-transfer { none; };
minimal-responses yes;
```
**Where:** `/etc/bind/named.conf.local`, inside `view "external" { }` (after `recursion no;`):
```
rate-limit {
    responses-per-second 10;
    window 5;
};
```
- `rate-limit` (Response Rate Limiting) stops `.190` being used as a reflector/amplifier against someone else, which is the DNS case of the amplification attacks in your notes. It sits in the external view only, so HR/IT lookups aren't limited.
- `allow-transfer { none; }` [extra]: without it, anyone may be able to run `dig AXFR` and download the whole zone. In the internal view, that includes your private IPs.
- `version` hides the BIND version from `dig version.bind chaos txt`.

```bash
sudo named-checkconf && sudo systemctl restart named
dig @127.0.0.1 gruppesechs.sitict.net +short          # still answers
dig @172.16.0.83 gruppesechs.sitict.net AXFR          # from IT/DMZ: "Transfer failed."
```

---

## 10. VM100 web server (nginx): Slowloris on the host [notes: lab 2.6, host side] + [extra]

**Where:** `/etc/nginx/sites-available/gruppesechs`, at the **top of the file, above `server {`** (site files are included inside nginx's `http { }`, which is where these two lines must be):
```
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_req_zone  $binary_remote_addr zone=reqs:10m rate=20r/s;
```
**Where:** inside `server { }`, after `server_tokens off;`:
```
client_header_timeout 10s;
client_body_timeout   10s;
keepalive_timeout     15s;
send_timeout          10s;
limit_conn perip 20;
limit_req zone=reqs burst=40 nodelay;
limit_conn_status 429;
limit_req_status 429;
```
`client_header_timeout` is the direct Slowloris fix: a client that hasn't sent its full headers within 10 s is dropped. `limit_conn` caps one IP at 20 open connections.

```bash
sudo nginx -t && sudo systemctl reload nginx
sysctl net.ipv4.tcp_syncookies        # must be 1 (Ubuntu default); your lab set it to 0 to show the attack
```
Optional: the same block on VM104 (Accounts portal) with its own file.

---

## 11. VM102 / VM106 jump hosts: SSH from the internet [extra]

`.84` takes SSH from the whole internet (`203.149.210.185:2222`). Nothing limits password guessing.

**Where:** new file `/etc/ssh/sshd_config.d/99-hardening.conf`:
```
PermitRootLogin no
MaxAuthTries 3
LoginGraceTime 30
MaxStartups 10:30:60
X11Forwarding no
```
```bash
sudo sshd -t && sudo systemctl restart ssh     # keep your current session open until a new login works
```

**fail2ban** (install while the 8.4 temporary rules are still up):
```bash
sudo apt install -y fail2ban
sudo tee /etc/fail2ban/jail.local >/dev/null <<'EOF'
[sshd]
enabled  = true
backend  = systemd
maxretry = 5
findtime = 10m
bantime  = 15m
ignoreip = 127.0.0.1/8 172.16.0.32/28 172.16.0.64/29
EOF
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```
`ignoreip` keeps the OOB network and IT from being banned. Outside clients keep their real source IP through R1/R2's static NAT, so fail2ban bans the attacker, not the router.

---

## 12. Already covered (no action)

MAC flooding (port-security), switch spoofing (`nonegotiate`, static modes), double tagging (native VLAN 821, unused VLAN 727 shut), STP (BPDU guard + root guard), DHCP snooping, DAI, IPSG, OOB VRF, SSH v2 + VTY ACLs (`vrf-also`), AAA (TACACS+ + local fallback), privilege levels, parser views, accounting, syslog, NetFlow, ASA ICMP inspection, internal anti-spoofing (user-SVI ACLs only permit the VLAN's own subnet as source), `no ip directed-broadcast` (IOS default since 12.0).
