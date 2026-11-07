# Lab Coverage: Labs 1.1 to 3.5 Mapped to the Project Configs

Source notes: `REVISION.docx` and `revision 2.md`. Config sources: `config/*.ttl` (macros), `HANDOVER.md`, `HARDENING.md`, `plan.md`.

Each lab lists **covers** (what the lab teaches), **config** (how it is applied in this project), **prevents** (the threat or goal), and **where** (files). Labs 2.1, 2.2 and 3.1 are partly setup/routing rather than pure security.

Device key: SW1-SW4 = access switches (Staff/ATM/HR/IT), DSW1/DSW2 = distribution switches, R1/R2 = edge routers, ASA = Firepower 1010, PA = PA-440, VM102/VM106 = TACACS+/jump/chrony hosts, VM101 = DNS, VM100 = web.

---

## Module 1: Layer 2 switch security

### Lab 1.1: MAC flooding (`macof`)
- **Covers:** CAM table exhaustion makes the switch flood unknown unicast like a hub, so an attacker can sniff. Defence is port security.
- **config:** on end-device access ports
  - `switchport mode access`
  - `switchport port-security`
  - `switchport port-security maximum 5`
  - `switchport port-security mac-address sticky`
  - `switchport port-security violation shutdown`
  - `storm-control broadcast level 5.00`
  - `storm-control multicast level 5.00`
  - `storm-control action trap`
  - `errdisable recovery cause psecure-violation` (global, recovers after 300 s)
- **prevents:** MAC flooding / CAM exhaustion and rogue devices on a user port.
- **where:** `sw1.ttl` (Gi1/0/5-16), `sw2.ttl` (5-7), `sw3.ttl` (5-6), `sw4.ttl` (5-7).
- **Note:** `maximum 5` is a deliberate choice. Sticky MACs never age out; free slots with `clear port-security sticky interface <if>`.

### Lab 1.2: Switch spoofing (Yersinia DTP)
- **Covers:** default `dynamic auto` ports accept DTP and can be negotiated into a trunk, exposing all VLANs.
- **config:**
  - On all access, unused and trunk ports:
    - `switchport nonegotiate`
  - Every port hard-set to `switchport mode access` or `switchport mode trunk`
  - Trunks pruned:
    - `switchport trunk allowed vlan 23,99` (SW1 Po1/Po2)
    - `switchport trunk allowed vlan 23,46,61,67,99,100` (Po5)
  - Unused ports:
    - `switchport mode access`
    - `switchport access vlan 727` (`UNUSED_BLACKHOLE`)
    - `shutdown`
- **prevents:** DTP switch spoofing, unauthorised trunking, use of dormant ports.
- **where:** `sw1-4.ttl`, `dsw1.ttl`, `dsw2.ttl`.

### Lab 1.3: Double tagging (native VLAN hopping)
- **Covers:** attacker on the native VLAN sends outer+inner 802.1Q tags; the first switch strips the outer tag and the inner tag reaches the victim VLAN (one-way).
- **config:**
  - Dedicated unused native VLAN 821 (`NATIVE_TRUNK`):
    - `vlan 821`
    - `name NATIVE_TRUNK`
  - On every trunk (Po1-Po5):
    - `switchport trunk native vlan 821`
  - No access port is in VLAN 1 or 821, and 821 is not in any allowed list
- **prevents:** 802.1Q double-tagging VLAN hopping.
- **where:** `sw1-4.ttl`, `dsw1.ttl`, `dsw2.ttl`.

### Lab 1.4: STP attack (rogue root bridge)
- **Covers:** attacker bridges two ports and sends superior BPDUs (lower priority) to become root and MitM traffic.
- **config:**
  - All switches:
    - `spanning-tree mode rapid-pvst`
  - DSW1 (roots aligned with HSRP):
    - `spanning-tree vlan 23,46,99,100 root primary`
    - `spanning-tree vlan 61,67 root secondary`
  - DSW2 (reverse of DSW1):
    - `spanning-tree vlan 61,67 root primary`
    - `spanning-tree vlan 23,46,99,100 root secondary`
  - User and OOB access ports:
    - `spanning-tree portfast`
    - `spanning-tree bpduguard enable`
  - DSW downlinks Po1-Po4 (not Po5):
    - `spanning-tree guard root`
  - SW1-SW4 (global):
    - `errdisable recovery cause bpduguard`
    - `errdisable recovery interval 300`
- **prevents:** root bridge hijack, L2 MitM, L2 loops from rogue switches.
- **where:** `sw1-4.ttl`, `dsw1.ttl`, `dsw2.ttl`.

### Lab 1.5: DHCP starvation and rogue DHCP (DHCPig, Metasploit, DNSChef)
- **Covers:** starvation randomises `chaddr` to exhaust the pool; a rogue server then wins the offer race and poisons gateway/DNS. Defence is DHCP snooping (trust boundaries, rate limit, option 82 handling).
- **config:**
  - SW1-SW4 (global):
    - `ip dhcp snooping`
    - `ip dhcp snooping vlan <23|46|61|67>`
    - `no ip dhcp snooping information option`
    - `errdisable recovery cause dhcp-rate-limit`
  - Uplink Port-Channels only (Po1, Po2):
    - `ip dhcp snooping trust`
  - User access ports:
    - `ip dhcp snooping limit rate 10`
  - DSW SVIs (Vlan23, 46, 61, 67):
    - `ip helper-address 172.16.0.77` (ISC DHCP server on VM107)
- **prevents:** pool exhaustion and rogue DHCP/DNS/gateway spoofing.
- **where:** `sw1-4.ttl`, `dsw1.ttl`, `dsw2.ttl`, VM107 `dhcpd.conf` (HANDOVER 8.5).

### Lab 1.6: ARP poisoning (`bettercap`)
- **Covers:** forged gratuitous ARP replies poison victim and gateway caches. Defence is Dynamic ARP Inspection using the snooping binding table (or ARP ACLs when no DHCP).
- **config:**
  - SW1-SW4 (global):
    - `ip arp inspection vlan <23|46|61|67>`
    - `ip arp inspection validate src-mac dst-mac ip`
    - `errdisable recovery cause arp-inspection`
  - Uplinks (Po1, Po2):
    - `ip arp inspection trust`
  - User access ports:
    - `ip arp inspection limit rate 15`
- **prevents:** ARP cache poisoning and L2 MitM.
- **where:** `sw1-4.ttl`. `sw3.ttl` keeps a commented-out static `arp access-list` / `ip source binding` block (old static HR host `.60`; HR now uses DHCP).

### Lab 1.7: Address spoofing (IP and MAC), IP Source Guard
- **Covers:** spoofing another host's IP (and MAC) to pass IP-based ACLs. Defence is IPSG (`ip verify source`, optional `mac-check`) fed by DHCP snooping or static bindings.
- **config:** on every end-user access port
  - `ip verify source`
  - sticky port-security (see Lab 1.1) covers the MAC side
- **prevents:** source IP spoofing from user ports.
- **where:** `sw1-4.ttl`. `ip verify source mac-check` is optional and not applied (HARDENING 4.5).

---

## Module 2: Layer 3/4, firewalls and DoS

### Lab 2.1: Inter-VLAN routing, OSPF and ACLs (routing setup + filtering)
- **Covers:** SVIs, routed ports, OSPF area 0, standard/extended/named/time-based/object-group ACLs applied inbound closest to the source.
- **config:**
  - OSPF area 0 router IDs:
    - R1 `1.1.1.1`
    - R2 `2.2.2.2`
    - DSW1 `3.3.3.3`
    - DSW2 `4.4.4.4`
    - ASA `5.5.5.5`
    - PA `6.6.6.6`
  - OSPF design:
    - cost 10 on primary paths
    - cost 500 on bypass links
    - `passive-interface default`
    - MD5 authentication
  - Inbound extended named ACLs on DSW SVIs:
    - `ACL_STAFF_IN` (Staff/ATM reach only Accounts .75 and Txn .76)
    - `ACL_ATM_IN` (same pattern as Staff)
    - `ACL_HR_IN` (file server .74 tcp/445, DNS .83, web .82, internet)
    - `ACL_IT_IN` (DNS, web, jump hosts tcp/22, internet)
    - `deny ip ... 172.16.0.0 0.0.0.127` blocks VLAN-to-VLAN traffic
  - ASA objects and time range:
    - `object network` / `object-group network` (`STAFF-ATM`, `HR-IT`, `DSW-SVIS`, `JUMP-SVRs`)
    - `time-range TXN-WINDOW`
    - `periodic weekdays 01:00 to 03:00` (used in `SERVER_IN`)
  - PA equivalent:
    - `schedule TXN-WINDOW` on rule `TXN-INTERNET-TIMED`
- **prevents:** unauthorised inter-VLAN and server access, Staff/ATM internet access, Txn internet access outside its window.
- **where:** `dsw1.ttl`, `dsw2.ttl`, `r1.ttl`, `r2.ttl`, `firepower1010.ttl`, `pa440.ttl`.
- **Open item:** temporary `permit icmp ... any` lines in `ACL_HR_IN` / `ACL_IT_IN` must be removed at final hardening.

### Lab 2.2: Stateful firewalls and MPF
- **Covers:** ASA `nameif` / security levels, zone ACLs, removing default NAT, MPF (class-map, policy-map, service-policy) for stateful ICMP inspection, state table, packet processing order.
- **config:**
  - Two firewalls in series (PTSO): PA then ASA
  - ASA cleanup and zones:
    - `no nat (any,outside) dynamic interface` (removes factory PAT)
    - `Vlan1` shut down
    - `inside` / `inside2`: security-level 100
    - `server`: security-level 60
    - `outside` / `wan-backup` / `wan-backup2`: security-level 0
  - ASA MPF:
    - `policy-map global_policy`
    - `class inspection_default`
    - `inspect icmp`
    - `inspect icmp error`
  - ASA ACLs bound per interface:
    - `INSIDE_IN`
    - `SERVER_IN`
    - `OUTSIDE_IN`
  - PA zones:
    - `outside`
    - `inside`
    - `dmz`
  - PA App-ID rules:
    - `INBOUND-WEB` (web-browsing + ssl)
    - `INBOUND-DNS`
    - `JUMP-SSH-IN`
    - `ALLOW-OSPF`
    - ... ending in `DENY-ALL`
- **prevents:** unsolicited inbound traffic, protocol tunnelling over allowed ports (App-ID), single-firewall compromise reaching every zone (containment).
- **where:** `firepower1010.ttl`, `pa440.ttl`.

### Lab 2.3: Ping sweep and port scan (`nmap`)
- **Covers:** host discovery and port scans; router `established` ACLs fail against `-PA` ACK scans, stateful firewalls do not; `no ip unreachables` makes ports appear `filtered`.
- **config:**
  - R1/R2 `WAN-IN` ingress ACL on G0/0/0:
    - `deny tcp any host <router WAN IP> eq 22 log`
    - `deny tcp any host <router WAN IP> eq 23 log`
    - `permit icmp host <ISP gateway> host <router WAN IP> echo-reply`
    - `permit ip any <public block>`
    - `permit tcp any host <router WAN IP> established`
    - `deny ip any any log`
  - R1/R2 G0/0/0 and `Null0`:
    - `no ip unreachables`
  - Port-specific static NAT only:
    - 80/443 to web
    - 53 to DNS
    - 2222 to jump host
    - unused public IPs routed to `Null0`
  - PA zone protection `ZP-OUTSIDE` and `ZP-DMZ`:
    - scan 8001 (TCP port scan)
    - scan 8002 (host sweep)
    - scan 8003 (UDP scan, outside only)
    - `action block-ip track-by source duration 300`
  - Discovery protocols:
    - `no cdp enable` (router WAN and user ports)
    - `lldp enable no` (PA L2 ports)
- **prevents:** reconnaissance of hosts and services from the internet or DMZ.
- **where:** `r1.ttl`, `r2.ttl`, `pa440.ttl`, `firepower1010.ttl`, `sw1-4.ttl`.

### Lab 2.4: Amplification and reflection (smurf), BCP 38
- **Covers:** spoofed source + directed broadcast turns intermediaries into reflectors. Defence: no directed broadcast and RFC 2827 / RFC 3330 filtering.
- **config:**
  - `WAN-IN` on R1/R2 starts with source denies for:
    - `0.0.0.0/8`
    - `10.0.0.0/8`
    - `127.0.0.0/8`
    - `169.254.0.0/16`
    - `172.16.0.0/24` (own range only, not `/12`, because the lab "internet" is 172.17/172.18)
    - `192.0.2.0/24`
    - `192.168.0.0/16`
    - `224.0.0.0/3`
    - router's own public block (`203.149.210.184/29` on R1, `129.126.142.184/29` on R2)
  - uRPF on R1/R2 G0/0/0:
    - `ip verify unicast source reachable-via rx allow-default`
  - PA zones `ZP-OUTSIDE` / `ZP-DMZ`:
    - `discard-ip-spoof yes`
  - Egress anti-spoofing:
    - each DSW user ACL only permits that VLAN's own subnet as source
    - `no ip directed-broadcast` (IOS default)
    - `no ip proxy-arp` on L3 interfaces
    - `no ip redirects` on L3 interfaces
  - VM101 BIND external view:
    - `recursion no;`
    - `rate-limit { responses-per-second 10; window 5; }`
- **prevents:** inbound/outbound IP spoofing, smurf reflection, DNS open-resolver amplification.
- **where:** `r1.ttl`, `r2.ttl`, `pa440.ttl`, `dsw1.ttl`, `dsw2.ttl`, VM101 `named.conf.*` (HANDOVER 8.10, HARDENING 9).

### Lab 2.5: TCP SYN flood
- **Covers:** spoofed SYNs fill the half-open table; ACLs cannot help. Defence is embryonic connection limits (TCP intercept) and SYN cookies.
- **config:**
  - ASA policy `PM_INSIDE_PROTECT` on `inside` and `inside2` (ACL `MPF_INTERNAL_SVRS` matches the server subnet):
    - `set connection conn-max 500`
    - `set connection embryonic-conn-max 50`
    - `set connection per-client-max 50`
    - `set connection per-client-embryonic-max 10`
    - `set connection timeout embryonic 0:00:15`
    - `threat-detection basic-threat`
    - `threat-detection statistics tcp-intercept`
  - PA `ZP-OUTSIDE`:
    - `flood tcp-syn enable yes`
    - `syn-cookies alarm-rate 100 activate-rate 200 maximal-rate 2000`
    - UDP flood limits (alarm 500, activate 1000, max 5000)
    - ICMP flood limits (alarm 50, activate 100, max 500)
  - Servers:
    - `net.ipv4.tcp_syncookies = 1` (the lab set it to 0 only to show the attack)
- **prevents:** SYN-flood exhaustion of DMZ and internal servers.
- **where:** `firepower1010.ttl`, `pa440.ttl`.
- **Check:** `show service-policy interface inside`, `show threat-detection statistics top tcp-intercept`.

### Lab 2.6: Slowloris
- **Covers:** many connections with trickled, incomplete HTTP headers. Defence: per-client connection limits.
- **config:**
  - PA DoS profile `DOS-PER-CLIENT`:
    - `type classified`
    - `resource sessions max-concurrent-limit 50`
  - PA rule `DOS-DMZ-SERVERS`:
    - outside to dmz
    - destinations WEB-SVR, DNS-SVR, JUMP-SVR
    - `classification-criteria address source-ip-only`
  - ASA `PM_INSIDE_PROTECT`:
    - `set connection per-client-max 50`
  - VM100 nginx:
    - `client_header_timeout 10s`
    - `client_body_timeout 10s`
    - `keepalive_timeout 15s`
    - `send_timeout 10s`
    - `limit_conn perip 20`
    - `limit_req zone=reqs burst=40 nodelay`
- **prevents:** low-and-slow connection exhaustion from a single IP.
- **where:** `pa440.ttl`, `firepower1010.ttl`, VM100 `/etc/nginx/sites-available/gruppesechs` (HARDENING 10).

---

## Module 3: OOB management, AAA and monitoring

### Lab 3.1: OOB management, VRF, SSH and jump host
- **Covers:** Proxmox bridges and VMs, separate management LAN using VRF on routers, dual-homed jump host, SSHv2, VTY ACL with `vrf-also`.
- **config:**
  - Proxmox bridges:
    - `vmbr1`: eno2 direct to ASA E1/6 (VLAN 69)
    - `vmbr2`: eno5 direct to PA eth1/4 (VLAN 92)
    - `vmbr3`: `bond0` active-backup (OOB VLAN 99)
  - Jump hosts VM102/VM106:
    - `ens18` in OOB (`.44` / `.45`, no gateway)
    - `ens19` in the DMZ (`.84` / `.85`, gateway `.81`)
    - reached from the internet at `203.149.210.185:2222` (R1/R2 static NAT to `.84:22`, PA rule `JUMP-SSH-IN`)
    - hardened with `sshd_config.d/99-hardening.conf` and fail2ban
  - OOB VRF:
    - `vrf definition OOB` on R1 (G0/1/0 `.33`), R2 (`.34`), DSW1/DSW2 (Vlan99 `.35` / `.36`)
    - ASA `Management1/1` (management-only, `.42`)
    - PA MGT (`.41`)
    - SW1-SW4 Vlan99 (`.37`-`.40`) with `ip default-gateway 172.16.0.33`
  - SSH on IOS:
    - `crypto key generate rsa modulus 2048`
    - `ip ssh version 2`
    - `ip ssh time-out 60`
    - `ip ssh authentication-retries 3`
  - VTY access control:
    - `ip access-list standard MGMT-ONLY` (permit `172.16.0.44` and `172.16.0.45`)
    - `line vty 0 2` with `access-class MGMT-ONLY in vrf-also`
    - `line vty 0 2` with `transport input ssh`
    - `line vty 3 15` / `line vty 16 31` with `transport input none`
    - `no ip http server`
  - ASA:
    - `ssh 172.16.0.44 255.255.255.255 management`
    - `ssh 172.16.0.45 255.255.255.255 management`
    - `ssh timeout 10`
  - PA:
    - `permitted-ip 172.16.0.44/32`
    - `permitted-ip 172.16.0.45/32`
    - telnet and http disabled
- **prevents:** in-band management exposure, direct SSH/Telnet from user or WAN networks, management/production routing leaks.
- **where:** all `.ttl` macros, HANDOVER sections 4 and 8.

### Lab 3.2: AAA part 1, authentication
- **Covers:** local `secret` / `algorithm-type`, brute-force slowing, centralised TACACS+ (and RADIUS) with local fallback.
- **config:**
  - Local credentials (IOS):
    - `enable algorithm-type scrypt secret ...`
    - `username ... algorithm-type scrypt secret ...`
    - `service password-encryption`
  - TACACS+ on IOS:
    - `aaa new-model`
    - `tacacs server TACACS1` (`.44`)
    - `tacacs server TACACS2` (`.45`)
    - `aaa group server tacacs+ tac-group` (vrf OOB on routers/DSWs, source Vlan99 or G0/1/0)
    - `aaa authentication login default group tac-group local`
    - `aaa authentication login CONSOLE group tac-group local`
    - `line vty 0 2` uses `login authentication default`
    - `line con 0` uses `login authentication CONSOLE`
  - ASA:
    - `aaa authentication ssh console TACACS LOCAL`
    - `aaa authentication enable console TACACS LOCAL`
    - `aaa local authentication attempts max-fail 5`
  - PA:
    - `admin-lockout failed-attempts 5 lockout-time 5`
    - `idle-timeout 10`
  - Jump hosts:
    - fail2ban `sshd` jail
  - Dropped: IOS `login block-for` / quiet-mode (its ACL has no `vrf-also`, so it would block all OOB SSH, including `.44`)
- **prevents:** weak or cleartext credentials, credential sprawl, brute force, lockout when one TACACS server fails.
- **where:** all macros, `firepower1010.ttl`, `pa440.ttl`, tac_plus config in `plan.md`.

### Lab 3.3: AAA part 2, authorization (privilege levels and CLI views)
- **Covers:** privilege levels, `parser view` / superviews, TACACS+/RADIUS attributes (`priv-lvl`, `cli-view-name`).
- **config:**
  - Roles:
    - Network Manager `bigDonkBonkAdmin`: IOS priv 15 / ASA priv 15 / PA superuser
    - Network Engineers `donkBonkEng1/2`: `eng-view` / ASA priv 5 / PA superreader
  - `parser view eng-view`:
    - `commands exec include all show`
    - `commands exec include ping`
    - `commands exec include traceroute`
    - `commands exec include clear`
    - `commands exec include configure terminal`
    - `commands configure include interface`
    - `commands interface include shutdown`
    - `commands interface include no shutdown`
    - `commands interface include switchport` (switches only)
  - ASA privilege level 5:
    - `privilege show level 5 mode exec command running-config`
    - `privilege show level 5 mode exec command route`
    - `privilege show level 5 mode exec command ospf`
    - `privilege show level 5 mode exec command interface`
    - `privilege show level 5 mode exec command access-list`
    - `privilege cmd level 5 mode exec command ping`
    - `privilege cmd level 5 mode exec command traceroute`
  - IOS authorization:
    - `aaa authorization exec default group tac-group local`
    - `aaa authorization config-commands`
    - `aaa authorization commands 15 default group tac-group local`
  - ASA authorization:
    - `aaa authorization command TACACS LOCAL` (re-enabled 7 Nov; keep a console open when rebuilding)
  - tac_plus groups `net_manager` and `net_engineers` (see `plan.md`)
- **prevents:** privilege escalation and unauthorised config changes (least-privilege RBAC).
- **where:** all macros (Phase 2 section), `plan.md`.

### Lab 3.4: AAA part 3, accounting
- **Covers:** audit trail of sessions and commands.
- **config:**
  - IOS:
    - `aaa accounting exec default start-stop group tac-group`
    - `aaa accounting commands 15 default stop-only group tac-group`
  - ASA:
    - `aaa accounting command TACACS`
- **prevents:** unaudited admin sessions and repudiation of privileged commands.
- **where:** all IOS macros, `firepower1010.ttl`.

### Lab 3.5: Network security monitoring (NTP, syslog, NetFlow v9)
- **Covers:** NTP modes, stratum, NTP authentication and ACLs, syslog severity and destinations, Flexible NetFlow.
- **config:**
  - NTP server side (VM102/VM106):
    - chrony syncs to `0.sg.pool.ntp.org` (PA rule `DMZ-OUTBOUND-NTP`)
    - `1 MD5 bigBoyNTPkey` in `/etc/chrony/chrony.keys`
  - NTP on devices:
    - `ntp authentication-key 1 md5 bigBoyNTPkey`
    - `ntp trusted-key 1`
    - `ntp authenticate`
    - `ntp server [vrf OOB] 172.16.0.44 key 1 prefer`
    - `ntp server [vrf OOB] 172.16.0.45 key 1`
    - `ip access-list standard NTP-SERVERS` (permit `.44`, `.45`)
    - `ntp access-group peer NTP-SERVERS`
    - `ntp logging`
    - ASA adds `source management`; PA uses symmetric-key MD5
  - Syslog:
    - `service timestamps log datetime msec localtime`
    - `logging trap informational`
    - `logging host 172.16.0.44` and `logging host 172.16.0.45` (`vrf OOB` on routers/DSWs; `management` on ASA; `SYSLOG-JUMP` profile on PA)
  - NetFlow (R1/R2/DSW1/DSW2):
    - `flow record bank-flow-record`
    - `flow exporter bank-flow-exporter-1` (`.44`, udp/2055, `vrf OOB`)
    - `flow exporter bank-flow-exporter-2` (`.45`, udp/2055, `vrf OOB`)
    - `flow monitor bank-flow-monitor`
    - `ip flow monitor bank-flow-monitor input` on routed ports and SVIs
- **prevents:** NTP time-spoofing (protects `TXN-WINDOW` and log correlation), log loss on reload, blind spots in traffic visibility.
- **where:** all macros, VM102/VM106 chrony (HANDOVER 8.11).
- **Check:** `show ntp associations detail` (`authenticated`, `sane`). Wazuh and NetFlow analyzer on the jump hosts are still open items.

---

## Extra defences beyond the labs

### OSPF MD5 (HARDENING 7)
- **config:**
  - `ip ospf authentication message-digest` (IOS)
  - `ip ospf message-digest-key 1 md5 GsOspf2026` (IOS)
  - `ospf authentication message-digest` and `ospf message-digest-key 1 md5 GsOspf2026` (ASA)
  - PA auth profile `OSPF-MD5` on eth1/3, eth1/6, eth1/7, vlan.10
  - PA `vlan.92` set passive
  - applied on all 9 OSPF links
- **prevents:** rogue OSPF neighbours, route injection (e.g. fake default from a DMZ server).

### HSRP MD5 (HARDENING 3.1)
- **config:**
  - `standby <vlan> authentication md5 key-string GsHsrp2026` on DSW1/DSW2 (Vlan23, 46, 61, 67)
- **prevents:** gateway takeover via forged HSRP hellos.

### HA and failover (HANDOVER 2-3)
- **config:**
  - two ISPs with IP SLA `track 1` (R1 metric 10, R2 metric 50)
  - routed /30s: ASA E1/3 + E1/4 to DSWs (cost 10)
  - routed /30s: PA eth1/6 + eth1/7 to DSWs (cost 500)
  - routed /30s: ASA E1/5 + E1/2 to R1/R2 (cost 500)
  - HSRP `track 10 decrement 30`
  - LACP Po1-Po5
  - Proxmox `bond0` active-backup
- **prevents:** single points of failure, L2 loops, asymmetric routing.

### Server hardening (HANDOVER 8, HARDENING 9-11)
- **config:**
  - Samba (VM103): `hosts allow`, tcp/445 only, SMB2+
  - Accounts/Txn nginx: TLS, IP `allow`, basic auth
  - BIND (VM101): views, `allow-transfer { none; }`, hidden version
  - sshd hardening + fail2ban on jump hosts
- **prevents:** unauthorised file/portal access, zone transfer leaks, SSH brute force.

---

## Known gaps (from HANDOVER section 11.4)

- Temporary ICMP permits on DSW `ACL_HR_IN` / `ACL_IT_IN` still to remove.
- Temporary server-internet rules (HANDOVER 8.4) still live.
- Inbound-from-internet tests not re-run since hardening.
- R1 `WAN-IN` redundant `permit icmp any 203.149.210.184 0.0.0.7` still live (removed from macro).
- Open issue P6: R1/R2 `WAN-IN` tcp/22 and tcp/23 denies reference `.94` instead of the router's own `.93` (R1).
