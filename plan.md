> Updated 5 Nov 2026 to match the `.ttl` macros and `HANDOVER.md` (those are the source of truth). Physical cabling and the full addressing table are in `TOPOLOGY.md`.

# **VLAN segregation**

Staff VLAN

- 10 bank tellers, 1 manager, 1 assist manager (12 ports reserved on SW1)  
- no internet access

ATM VLAN

- 3 ATMs (3 ports reserved on SW2)  
- no internet access

HR VLAN

- 1 manager, 1 admin staff (2 ports reserved on SW3)  
- internet access

IT VLAN

- 1 network manager, 2 network engineers (3 ports reserved on SW4)  
- internet access 

No VLANs should be able to access each other

# **Servers**

1 File Server

- access: HR


1 Accounts server

- access: Staff, ATM


1 Transactions server

- access: Staff, ATM  
  - Internet access at certain time window to process external transactions(requirements made known during demo)  
    

1 Web server

- access: “Closed Internet”(other racks in the lab within ICT networks and VPN but not directly from the internet)  
- hostname: GruppeSechs  
- no defense needed


1 Authoritative DNS server

- so web server is accessible via domain name’  
- no defense needed


1 DHCP server

- standalone ISC DHCP server on VM107 (172.16.0.77); both DSWs relay to it with ip helper-address  
- DNS server (.83) is handed out to HR and IT only

# **Devices**

1x Firepower 1010   
1x PA-440   
2x 8200L routers   
2x 9300L Distribution Switches  
4x 9200L Access Switches  
1x Supermicro

- VM100 (Web server)  
- VM101 (DNS)  
- VM102 (TACACS+ 1\)  
- VM103 (File Server)  
- VM104 (Accounts Server)  
- VM105 (Transactions Server)  
- VM106 (TACACS+ 2\)  
- VM107 (DHCP)

1x ISP1

- IP address block: 172.17.9.92/30  
  - ISP1 Router IP: 172.17.9.94  
  - R1 G0/0/0: 172.17.9.93  
  - ISP1 Public block: 203.149.210.184/29

1x ISP2 (added 5 Nov, redundancy only; approved by the prof)

- IP address block: 172.17.10.92/30  
  - ISP2 Router IP: 172.17.10.94  
  - R2 G0/0/0: 172.17.10.93  
  - ISP2 Public block: 129.126.142.184/29

We placed our firewalls in a serial formation where the NGFW(PA-440) intercepts the packets first then the ASA(Firepower 1010\) filters again. The main security benefit here is containment. In the event that the first NGFW gets broken into, the attacker would only have access to our DMZ zone, they would need to break into the ASA in order to get access into our Inside zone. Vice Versa.

# **VLSM**

## **Staff VLAN 23**

IP block: 172.16.0.0/27  
Default Gateway: 172.16.0.1  
Usable IP: 172.16.0.2 to 172.16.0.30

- DSW1/DSW2 HSRP VIP: 172.16.0.1  
- DSW1 int VLAN 23 HSRP Active: 172.16.0.2  
- DSW2 int VLAN 23 HSRP standby: 172.16.0.3  
- Unused/Given by DHCP: 172.16.0.4-30 (12 Staff)

Broadcast: 172.16.0.31

## **OOB VLAN 99**

IP block: 172.16.0.32/28  
Default Gateway: 172.16.0.33  
Usable IP: 172.16.0.34 to 172.16.0.46

- R1: 172.16.0.33  
- R2: 172.16.0.34  
- DSW1: 172.16.0.35  
- DSW2: 172.16.0.36  
- SW1: 172.16.0.37  
- SW2: 172.16.0.38  
- SW3: 172.16.0.39  
- SW4: 172.16.0.40  
- PA: 172.16.0.41  
- ASA: 172.16.0.42  
- Proxmox: 172.16.0.43  
- VM102(TACACS+ 1): 172.16.0.44  
- VM106(TACACS+ 2): 172.16.0.45  
- Management laptop (static, on DSW1 Gi1/0/21): 172.16.0.46  
- VLAN 99 is in VRF OOB on R1, R2 and the DSWs

Broadcast: 172.16.0.47

## **ATM VLAN 46**

IP block: 172.16.0.48/29  
Default Gateway: 172.16.0.49  
Usable IP: 172.16.0.50 to 172.16.0.54

- DSW1 HSRP VIP: 172.16.0.49  
- DSW1 vlan 46 HSRP Active: 172.16.0.50  
- DSW2 vlan 46 HSRP Standby: 172.16.0.51  
- Unused/Given by DHCP: 172.16.0.52-54 (3 ATMs)

Broadcast: 172.16.0.55

## **HR VLAN 61**

IP block: 172.16.0.56/29  
Default Gateway: 172.16.0.57  
Usable IP: 172.16.0.58 to 172.16.0.62

- DSW1/DSW2 HSRP VIP: 172.16.0.57  
- DSW1 int 61 HSRP standby: 172.16.0.58  
- DSW2 int 61 HSRP Active: 172.16.0.59  
- Reserved (old static HR host): 172.16.0.60  
- Given by DHCP: 172.16.0.61-62 (2 HR)

Broadcast: 172.16.0.63

## **IT VLAN 67**

IP block: 172.16.0.64/29  
Default Gateway: 172.16.0.65  
Usable IP: 172.16.0.66 to 172.16.0.70

- DSW1/DSW2 HSRP VIP: 172.16.0.65  
- DSW1 int VLAN 67 HSRP Standby: 172.16.0.66  
- DSW2 int VLAN 67 HSRP Active: 172.16.0.67  
- Unused/Given by DHCP: 172.16.0.68-70 (3 IT)

Broadcast: 172.16.0.71

## **Internal Servers VLAN 69 connected to ASA**

IP block: 172.16.0.72/29  
Default Gateway: 172.16.0.73  
Usable IP: 172.16.0.74 to 172.16.0.78

- ASA Vlan69 SVI (`server`, on E1/6, cabled directly to Proxmox eno2): 172.16.0.73  
- VM103(File): 172.16.0.74  
- VM104(Accounts): 172.16.0.75  
- VM105(Txn): 172.16.0.76  
- VM107(DHCP): 172.16.0.77  
- .78: unused. The PA backup gateway (vlan.69 on eth1/8) was removed 5 Nov. The internal servers belong to the ASA only; if the ASA fails, they are unreachable by design.  
- Servers reach this VLAN through Proxmox eno2 (vmbr1) -> ASA E1/6 directly (no DSW in the path since 5 Nov)

Broadcast: 172.16.0.79

## **DMZ Servers VLAN 92 connected to PA**

IP block: 172.16.0.80/29  
Default Gateway: 172.16.0.81  
Usable IP: 172.16.0.82 to 172.16.0.86

- PA vlan.92 (on eth1/4, cabled directly to Proxmox eno5): 172.16.0.81  
- VM100(Web): 172.16.0.82  
  - public 203.149.210.189 (R1 static NAT; 129.126.142.189 on R2)  
- VM101(DNS): 172.16.0.83  
  - public 203.149.210.190 (R1 static NAT; 129.126.142.190 on R2)  
- VM102(TACACS+ 1 / Jump 1): 172.16.0.84  
  - public 203.149.210.185 tcp/2222 -> .84:22 (R1 static NAT; 129.126.142.185 on R2)  
- VM106(TACACS+ 2 / Jump 2): 172.16.0.85  
- .86: unused. The ASA backup gateway (Vlan92 on E1/7) was removed 5 Nov. The DMZ belongs to the PA only; if the PA fails, the DMZ is unreachable by design.  
- Servers reach this VLAN through Proxmox eno5 (vmbr2) -> PA eth1/4 directly (no DSW in the path since 5 Nov)

Broadcast: 172.16.0.87

## **Routers to PA**

IP block: 172.16.0.88/29  
Usable IP: 172.16.0.89 to 172.16.0.94

- R1 g0/0/1: 172.16.0.90  
- R2 g0/0/1: 172.16.0.91  
- PA vlan.10: 172.16.0.92  
  - L2 ports eth1/5 (to R1 G0/0/1) and eth1/2 (to R2 G0/0/1) in PA VLAN `OUTSIDE`. eth1/1 is unused (admin-down).  
- unused: .89 .93 .94 

Broadcast: 172.16.0.95

## **ASA to Distribution Switches (routed /30s, 5 Nov)**

The old shared VLAN 100 segment (172.16.0.96/29) was split into two point-to-point /30s, one per DSW. Two routed ports can't share a subnet, so each link gets its own. Both are active.

- ASA E1/3 (nameif inside) <-> DSW1 Gi1/0/11 (routed): 172.16.0.96/30, ASA .97, DSW1 .98, broadcast .99  
- ASA E1/4 (nameif inside2) <-> DSW2 Gi1/0/11 (routed): 172.16.0.100/30, ASA .101, DSW2 .102, broadcast .103  
- OSPF cost 10 on both. `inside` and `inside2` share the INSIDE_IN ACL.

## **PA ASA-bypass links (routed /30s, 5 Nov)**

Replaces the old PA vlan.100 (.100) bypass, which bridged eth1/6 and eth1/7 and formed a VLAN 100 loop with Po5.

- PA eth1/6 <-> DSW1 Gi1/0/12: 172.16.0.116/30, PA .117, DSW1 .118, broadcast .119  
- PA eth1/7 <-> DSW2 Gi1/0/12: 172.16.0.120/30, PA .121, DSW2 .122, broadcast .123  
- OSPF cost 500: idle while the ASA path exists, carries user traffic only if the ASA is unreachable from both DSWs. PA zone `inside`.

## **DSW1 to DSW2 transit (VLAN 100 over Po5)**

- 172.16.0.124/30: DSW1 Vlan100 .125, DSW2 Vlan100 .126, broadcast .127  
- OSPF cost 10. VLAN 100 now only exists on Po5 (no firewall ports). It lets a DSW that lost its own ASA link reach the ASA through the other DSW.

## **PA to ASA**

IP block: 172.16.0.104/30  
Usable IP: 172.16.0.105 to 172.16.0.106

- PA e1/3: 172.16.0.105  
- ASA e1/1: 172.16.0.106

Broadcast: 172.16.0.107

## **R1 to ASA bypass (was Router to Router until 5 Nov)**

IP block: 172.16.0.108/30

- R1 g0/1/1: 172.16.0.109  
- ASA E1/5 (nameif wan-backup2): 172.16.0.110  
- OSPF cost 500, only used if the PA fails

The direct R1-R2 link was removed when R2 got its own ISP. R1 and R2 still see each other in OSPF over the PA's VLAN 10 segment.

- each routed port in the router needs it own unique IP address and SUBNET  
- this is why we cant use the spare .93 and .94 from the routers to pa subnet as it will cause overlap

Broadcast: 172.16.0.111

## **R2 to ASA bypass**

IP block: 172.16.0.112/30

- R2 g0/1/1: 172.16.0.113 (moved from g0/0/0, which is now ISP2)  
- ASA E1/2 (nameif wan-backup): 172.16.0.114  
- OSPF cost 500, only used if the PA fails

Broadcast: 172.16.0.115

## **WAN and public addresses**

NAT is done on the routers (5 Nov). Each router translates to its own ISP's block, so traffic always leaves with the right public address. The PA and ASA don't NAT.

- WAN1 172.17.9.92/30: R1 G0/0/0 172.17.9.93, ISP1 172.17.9.94 (primary)  
- WAN2 172.17.10.92/30: R2 G0/0/0 172.17.10.93, ISP2 172.17.10.94 (backup, used only when R1 or ISP1 is down)  
- Public blocks, same layout on both (ISP1 203.149.210.184/29 on R1, ISP2 129.126.142.184/29 on R2):  
  - .185: outbound PAT for HR, IT, Txn, DNS server and jump hosts, plus tcp/2222 -> jump host .84:22  
  - .189: web server .82 (tcp/80, tcp/443)  
  - .190: DNS server .83  
- The lab DNS only knows the ISP1 addresses (.189). With R1 down, outsiders reach the servers by the ISP2 IPs only.  
- Inside users never use the public IPs (the routers don't hairpin NAT). VM101's internal view gives HR/IT 172.16.0.82 for gruppesechs.sitict.net.

## **Other VLANs**

- VLAN 727: blackhole for unused switch ports  
- VLAN 821: native VLAN on all trunks

**OSPF ids**  
R1: 1.1.1.1  
R2: 2.2.2.2  
DSW1: 3.3.3.3  
DSW2: 4.4.4.4  
Firepower 1010: 5.5.5.5  
PA-440: 6.6.6.6

# **Configuration of devices**

- we ***need*** to implement additional security features/network security monitoring not covered in the module.  
  - we added palto firewall

## **Management OOB (VLAN 99\)**

We have considered two approaches in configuring our management oob in our switches:

1. Using the dedicated management ports G0/0 on the 9200L and 9300L switches:

As per our topology, if we were to use the dedicated management ports, we would need to cable the ports(G0/0) from the switches to say G1/0/23-24 of the 9300L in VLAN 99(Default Gateway). We would also need to do a “loop” cable from the 9300L’s own management port to the front of their ports, say G1/0/22 in VLAN 99\. This would allow the packets to flow through the dedicated management ports in the management VLAN. This provides us with a benefit which is a more secure connection for the management data as they are not travelling in the same links as the internal data. However, this setup also has its disadvantages which is no redundancy, and extra cabling:

- The dedicated management port is only 1 cable running from G0/0 to the 9300L.  
- Extra cables needed to connect the management port, whereas the 2nd option runs the traffic through the port channels, mixing it with the internal data.  
2. Using VLAN 99 as a management VLAN for management routing through custom VRF:

Having separate dedicated VLAN 99 ports runs the management traffic through the port channels we have created linking the 9200L and 9300L. This allows for better redundancy through Etherchannels.

Final approach:  
We evaluated using the management ports on the switches. However, because of budget, our rack does not have a separate OOB management switch(well we do, but we can’t use it according to the project brief), cabling G0/0 back into the front panel ports of DSW1 or DSW2 would reduce each access switch from a Etherchannel-ed, 4-link redundant path to a single non-redundant cable, creating a single point of failure if one DSW goes down(losing access to S1/S2 if DSW1 goes down). Therefore, we kept VLAN 99 trunked over Port-Channels with vrf for resilient high availability.   
**Parallel Topologically Serial Operationally**  
Our topology has redundant links between our routers/dsw and firewalls. This allows us to have redundancy in the event a firewall goes down. For example, if our Firepower 1010 goes down, the distributed switches are still able to route their traffic towards PA-440 so that the single point of failure is prevented. This represents the structure of a parallel topology with redundancy, but our traffic still functions serially where the traffic in normal settings still flows through both firewalls for ACL inspection and zone access.

doubts

- does this still keep our main focus of serial firewalls which is containment? if an attacker breaches pa-440, they would have access to the switches which is what we wanted to prevent with serial?  
  - ospf ensures traffic is still routed towards ASA so the attack is still contained in pa-440  
  - but maybe since the attack has successfully attacked the pa-440, it can just simply change the configs?   
- packets still route through the default route, PA \> ASA, due to ospf right? since both firewalls are able to access dmz and servers, do we need acls to prevent like ASA accessing dmz if no routes are down? these acls would go down if it detects the main route is down. or is that all covered by ospf

## **Switches**

SW1 (Staff VLAN)

- vlan 23, vlan 99(management oob), vlan 727(unused ports), vlan 821(native)  
- int vlan 99  
  - ip address 172.16.0.37 255.255.255.240  
  - ip default-gateway 172.16.0.33  
- int g1/0/5-16  
  - vlan 23  
  - port security towards end users  
    - mac-address sticky, violation shutdown, max 2  
  - portfast and bpduguard enabled and ip source guard  
- int g1/0/1-2, towards dsw1  
  - channel-group 1 mode active  
  - trunk  
  - native vlan 821  
- int g1/0/3-4, towards dsw2  
  - channel-group 2 mode active  
  - trunk  
  - native vlan 821  
- port channels 1 and 2 allow vlan 23 and vlan 99  
  - ip dhcp snooping trust, facing towards dhcp server  
  - ip arp inspection trust  
- access links for all except port channels  
- unused ports g1/0/17-24, set to vlan 727 and shutdown  
- ip dhcp/arp snooping for vlan 23  
  - no ip dhcp snooping information option  
  - ip arp inspection limit rate 15

SW2 (ATM Vlan)

- vlan 46, vlan 99(management oob), vlan 727(unused ports), vlan 821(native)  
- int vlan 99  
  - ip address 172.16.0.38 255.255.255.240  
  - ip default-gateway 172.16.0.33  
- int g1/0/5-7  
  - vlan 46  
  - port security towards end users  
    - mac-address sticky, violation shutdown, max 1(atms shouldnt switch devices often)  
  - portfast and bpduguard enabled and ip source guard  
- int g1/0/1-2, towards dsw1  
  - channel-group 1 mode active  
  - trunk  
  - native vlan 821  
- int g1/0/3-4, towards dsw2  
  - channel-group 2 mode active  
  - trunk  
  - native vlan 821  
- port channels 1 and 2 allow vlan 46 and vlan 99  
  - ip dhcp snooping trust, facing towards dhcp server  
  - ip arp inspection trust  
- access links for all except port channels  
- unused ports g1/0/8-24, set to vlan 727 and shutdown  
- ip dhcp/arp snooping for vlan 46  
  - no ip dhcp snooping information option  
  - ip arp inspection limit rate 15

SW3 (HR Vlan)

- vlan 61, vlan 99(management oob), vlan 727(unused ports), vlan 821(native)  
- int vlan 99  
  - ip address 172.16.0.39 255.255.255.240  
  - ip default-gateway 172.16.0.33  
- int g1/0/5-6  
  - vlan 61  
  - port security towards end users  
    - mac-address sticky, violation shutdown, max 2  
  - portfast and bpduguard enabled and ip source guard  
- int g1/0/1-2, towards dsw1  
  - channel-group 1 mode active  
  - trunk  
  - native vlan 821  
- int g1/0/3-4, towards dsw2  
  - channel-group 2 mode active  
  - trunk  
  - native vlan 821  
- port channels 1 and 2 allow vlan 61 and vlan 99  
  - ip dhcp snooping trust, facing towards dhcp server  
  - ip arp inspection trust  
- access links for all except port channels  
- unused ports g1/0/7-24, set to vlan 727 and shutdown  
- ip dhcp/arp snooping for vlan 61  
  - no ip dhcp snooping information option  
  - ip arp inspection limit rate 15

SW4 (IT Vlan)

- vlan 67, vlan 99(management oob), vlan 727(unused ports), vlan 821(native)  
- int vlan 99  
  - ip address 172.16.0.40 255.255.255.240  
  - ip default-gateway 172.16.0.33  
- int g1/0/5-7  
  - vlan 67  
  - port security towards end users  
    - mac-address sticky, violation shutdown, max 2  
  - portfast and bpduguard enabled and ip source guard  
- int g1/0/1-2, towards dsw1  
  - channel-group 1 mode active  
  - trunk  
  - native vlan 821  
- int g1/0/3-4, towards dsw2  
  - channel-group 2 mode active  
  - trunk  
  - native vlan 821  
- port channels 1 and 2 allow vlan 67 and vlan 99  
  - ip dhcp snooping trust, facing towards dhcp server  
  - ip arp inspection trust  
- access links for all except port channels  
- unused ports g1/0/8-24, set to vlan 727 and shutdown  
- ip dhcp/arp snooping for vlan 67  
  - no ip dhcp snooping information option  
  - ip arp inspection limit rate 15

on all access switches

- G0/0 (rear management port) shut down, management is in-band on VLAN 99 over the port-channels  
- ip domain name gruppesechs.sitict.net, no ip domain lookup, service password-encryption, service timestamps log datetime msec localtime, clock timezone SGT 8 0  
- username bigDonkBonkAdmin privilege 15 algorithm-type scrypt secret Venison2Beef  
- enable algorithm-type scrypt secret 4Venison2  
- ntp server 172.16.0.44 prefer, 172.16.0.45  
- spanning-tree mode rapid-pvst  
- vty ACL to allow .44 and .45 to access switch  
- aaa new-model  
  - tacacs server TACACS1, address ipv4 172.16.0.44, key bigBoy@ServerPassword1  
  - tacacs server TACACS2, address ipv4 172.16.0.45, key bigBoy@ServerPassword2  
  - aaa group server tacacs+ tac-group, server name TACACS1, server name TACACS2, ip tacacs source-interface vlan 99  
  - aaa authentication login default group tac-group local  
  - aaa authentication login CONSOLE group tac-group local  
  - aaa authorization exec default group tac-group local  
    - lands user in exec shell  
  - aaa authorization config-commands  
    - perform authorization checks on every command in config t  
  - aaa authorization commands 15 default group tac-group local  
    - validate every priv level 15 command   
  - aaa accounting exec default start-stop group tac-group  
  - aaa accounting commands 15 default stop-only group tac-group  
    - records and send audit trail of every priv 15 command  
- Ip ssh version 2, time-out 60, authentication-retries 3  
- line vty 0 2, access-class MGMT-ONLY in, transport input ssh, login authentication default, exec-timeout 10 0  
- line vty 3 15, transport input none  
- line con 0, login authentication CONSOLE  
- syslog  
  - logging host 172.16.0.44, logging host 172.16.0.45, logging trap informational (no VRF on the access switches: they are Layer 2 with ip default-gateway 172.16.0.33)  
- crypto key generate rsa modulus 2048  
- SW3 has an optional (commented-out) static ARP ACL / ip source binding for a static HR host at .60. HR now uses DHCP, so it stays off.

## **Distribution Switches**

DSW1

- no ip domain lookup, ip routing, ip domain name gruppesechs.sitict.net, service password-encryption, service timestamps log datetime msec localtime, clock timezone SGT 8 0, appropriate banner motd  
- enable algorithm-type scrypt secret 4Venison2  
- username bigDonkBonkAdmin privilege 15 algorithm-type scrypt secret Venison2Beef  
- vrf definition OOB, address-family ipv4  
- G0/0 (rear management port) shut down  
- vlan 23 STAFF, 46 ATM, 61 HR, 67 IT, 99 MGMT\_OOB, 100 DSW\_TRANSIT, 727 UNUSED\_BLACKHOLE, 821 NATIVE\_TRUNK  
- ip sla 10 (icmp-echo 172.16.0.97 source-interface Gi1/0/11, frequency 5), track 10 ip sla 10 reachability  
  - checks this DSW's own link to the ASA. Used by HSRP and by the default route.  
- int vlan 23, 46, 61, 67  
  - standby version 2, standby \<vlan\> track 10 decrement 30, preempt  
    - decrement 30 (not 20): 110 - 20 = 90 ties with the peer's 90, and HSRP breaks a tie on the higher IP, so DSW2 (.59/.67) would never give up 61/67  
  - ip ospf cost 10 on the VLANs this DSW is HSRP Active for (23, 46), 20 on the others (61, 67), so the ASA sends return traffic to the Active DSW  
  - ip helper-address 172.16.0.77  
  - ip access-group ACL\_\<VLAN\>\_IN in, ip flow monitor bank-flow-monitor input, no ip redirects, no ip proxy-arp  
- int vlan 99  
  - ip add 172.16.0.35 255.255.255.240  
  - vrf forwarding OOB, switch from default Mgmt-vrf to OOB  
- int vlan 100 (DSW1 <-> DSW2 transit over Po5 only)  
  - ip add 172.16.0.125 255.255.255.252, ip ospf cost 10, ip flow monitor bank-flow-monitor input  
- hsrp standby 23, 46 with priority 110, and preempt (Active)  
- hsrp standby 61,67 with priority 90, and preempt (Standby)  
- STP rapid-pvst: vlan 23, 46, 99, 100 root primary  
- STP vlan 61, 67 root secondary  
- Etherchannels (LACP active)  
  - int g1/0/1-2 Po1 (towards SW1), allows 23, 99  
  - int g1/0/3-4 Po2 (towards SW2), allows 46, 99  
  - int g1/0/5-6 Po3 (towards SW3), allows 61, 99  
  - int g1/0/7-8 Po4 (towards SW4), allows 67, 99  
  - int g1/0/9-10 Po5 (towards DSW2), allows 23, 46, 61, 67, 99, 100  
  - trunk  
  - switchport nonegotiate  
  - native vlan change 821  
  - spanning-tree guard root on Po1-Po4 (not Po5)  
- g1/0/11 routed (no switchport) to ASA E1/3: 172.16.0.98/30, ip ospf cost 10  
- g1/0/12 routed (no switchport) to PA eth1/6: 172.16.0.118/30, ip ospf cost 500 (ASA bypass)  
- g1/0/22-24, access vlan 99, portfast + bpduguard: g1/0/22 R1 G0/1/0, g1/0/23 PA MGT, g1/0/24 Supermicro eno6 (bond0 primary)  
- unused ports g1/0/13-21 are put in 727 and shutdown (g1/0/13-14 were the VLAN 69 server bridge, g1/0/15 the PA eth1/8 server backup; all removed 5 Nov)  
  - live only (not in macro yet): g1/0/21 is access vlan 99 for a management laptop (172.16.0.46/28, gw .33)  
- OSPF  
  - router-id 3.3.3.3, auto-cost reference-bandwidth 10000, passive-interface default, no passive-interface Vlan100 / Gi1/0/11 / Gi1/0/12  
  - area 0 networks: 172.16.0.0/27, .48/29, .56/29, .64/29, .96/30 (ASA), .116/30 (PA), .124/30 (DSW2)  
- static routes  
  - ip route vrf OOB 0.0.0.0 0.0.0.0 172.16.0.33 (REQUIRED, VRF OOB has no routing protocol)  
  - ip route 0.0.0.0 0.0.0.0 172.16.0.97 track 10 (OPTIONAL, pins the ASA path; withdrawn if this DSW loses its ASA link)  
- ntp server vrf OOB 172.16.0.44 prefer, 172.16.0.45

DSW2

- same as DSW1 except:  
- int vlan 23 .3, vlan 46 .51, vlan 61 .59, vlan 67 .67, vlan 99 .36, vlan 100 .126  
- hsrp standby 23, 46 with priority 90, and preempt (Standby); ip ospf cost 20 on these SVIs  
- hsrp standby 61,67 with priority 110, and preempt (Active); ip ospf cost 10 on these SVIs  
- STP vlan 61, 67 root primary; vlan 23, 46, 99, 100 root secondary  
- Po5 (g1/0/9-10) towards DSW1  
- ip sla 10 pings the ASA at 172.16.0.101 from Gi1/0/11  
- g1/0/11 routed to ASA E1/4: 172.16.0.102/30, ip ospf cost 10  
- g1/0/12 routed to PA eth1/7: 172.16.0.122/30, ip ospf cost 500 (ASA bypass)  
- g1/0/22-24, access vlan 99: g1/0/22 R2 G0/1/0, g1/0/23 ASA Management1/1, g1/0/24 Supermicro eno1 (bond0 backup)  
- unused ports g1/0/13-21 are put in 727 and shutdown (g1/0/13-14 were the VLAN 92 DMZ bridge, g1/0/15 the ASA E1/7 DMZ backup; all removed 5 Nov)  
- OSPF router-id 4.4.4.4, area 0 networks as DSW1 but .100/30 (ASA), .120/30 (PA), .124/30 (DSW1)  
- ip route 0.0.0.0 0.0.0.0 172.16.0.101 track 10  
- DSW2 now has its own ASA link (E1/4), so Po5 is no longer in the HR/IT path. It is only used if one DSW loses its ASA link.

both DSW

- ACL\_STAFF\_IN (Staff VLAN 23 SVI, inbound)  
  - permit HSRP v2 hellos from the peer DSW (udp 172.16.0.0/27 -> 224.0.0.102 eq 1985), otherwise both DSWs go Active  
  - permit DHCP relay replies for the peer DSW (udp host 172.16.0.77 eq bootps -> any eq bootps)  
  - permit DHCP requests (to 255.255.255.255 and to .77 bootps)  
  - permit ip to accounts (.75) and txn (.76) servers  
  - deny ip any any log  
- ACL\_ATM\_IN: same pattern as Staff for 172.16.0.48/29  
- ACL\_HR\_IN  
  - permit DHCP peer replies and DHCP requests  
  - permit tcp/445 and icmp to file server (.74)  
  - permit udp/tcp 53 to DNS (.83), tcp 80/443 to web (.82)  
  - TEMPORARY: permit icmp any (remove at final hardening)  
  - deny ip to 172.16.0.0/25 log (other VLANs and private subnets, anti-smurf), then permit ip any (internet)  
- ACL\_IT\_IN  
  - permit DHCP peer replies and DHCP requests  
  - permit udp/tcp 53 to DNS (.83), tcp 80/443 to web (.82), tcp/22 to jump hosts .84 and .85  
  - TEMPORARY: permit icmp any (remove at final hardening)  
  - deny ip to 172.16.0.0/25 log, then permit ip any (internet)  
- MGMT-ONLY standard acl permits 172.16.0.44 and .45 to access vty ssh only, deny any log  
  - when engineer wants to ssh to the jump host from internet  
1. ssh \-p 2222 bigDonkBonkAdmin@203.149.210.185(public isp ip)  
2. R1 has a static NAT (port forward) that sends this to Jump Host DMZ int 172.16.0.84:22; the PA rule JUMP-SSH-IN allows it  
- if local connect from internal IT VLAN 67  
1. ssh bigDonkBonkAdmin@172.16.0.84 or .85 (jump hosts)  
- engineer authenticates with credentials and lands in shell  
- from there, ssh into the appropriate management ip addresses  
- aaa  
  - aaa new-model  
  - tacacs server TACACS1, address ipv4 172.16.0.44, key bigBoy@ServerPassword1  
  - tacacs server TACACS2, address ipv4 172.16.0.45, key bigBoy@ServerPassword2  
  - aaa group server tacacs+ tac-group, server name TACACS1, server name TACACS2, ip vrf forwarding OOB, ip tacacs source-interface vlan 99  
  - aaa authentication login default group tac-group local  
  - aaa authentication login CONSOLE group tac-group local  
  - aaa authorization exec default group tac-group local  
    - lands user in exec shell  
  - aaa authorization config-commands  
    - perform authorization checks on every command in config t  
  - aaa authorization commands 15 default group tac-group local  
    - validate every priv level 15 command   
  - aaa accounting exec default start-stop group tac-group  
  - aaa accounting commands 15 default stop-only group tac-group  
    - records and send audit trail of every priv 15 command  
- SSH  
  - ip ssh version 2, time-out 60, authentication-retries 3  
  - line vty 0 2, access-class MGMT-ONLY in vrf-also, transport input ssh, login authentication default, exec-timeout 10 0  
  - line vty 3 15, transport input none  
- line con 0, login authentication CONSOLE, exec-timeout 10 0  
- syslog  
  - logging host 172.16.0.44 vrf OOB, logging host 172.16.0.45 vrf OOB, logging trap informational  
- PTSO  
  - Server zones don't pass through the DSWs (5 Nov). Proxmox eno2 (vmbr1, VLAN 69) plugs straight into ASA E1/6 and eno5 (vmbr2, VLAN 92) straight into PA eth1/4. Each zone has one gateway and no backup, so a switch in between added nothing. VLANs 69 and 92 don't exist on the DSWs.  
    - Po5 allows 23, 46, 61, 67, 99, 100  
  - Firewall links (5 Nov): each DSW has its own routed /30 to the ASA (G1/0/11, cost 10) and to the PA (G1/0/12, cost 500). VLAN 100 is only the DSW1 <-> DSW2 transit over Po5.  
    - track 10 (ip sla to this DSW's ASA address) lowers HSRP priority by 30 and withdraws the static default when this DSW loses its ASA link, so the other DSW takes over the user VLANs.  
  - Both firewalls must see both directions of every connection. A backup link is safe when it is its own transit subnet (OSPF cost decides); it is unsafe when it puts a second firewall in a server subnet (connected route beats OSPF). So the server-subnet backups (PA .78, ASA .86) were removed, and the transit backups (PA eth1/6/7, ASA E1/4, ASA <-> R2) stay up and fail over automatically.

## **Routers**

R1 holds the primary WAN connection (G0/0/0 172.17.9.93 to the ISP1 gateway 172.17.9.94) and tracks it with IP SLA 1. R2 holds the backup WAN connection (G0/0/0 172.17.10.93 to the ISP2 gateway 172.17.10.94), tracked the same way. Both advertise a default route into OSPF only while their ISP answers: R1 at metric 10, R2 at metric 50, so ISP1 is always preferred and ISP2 only takes over when R1 or ISP1 fails.

Since 5 Nov each router also does the NAT for its own ISP block (PAT for outbound, static port forwards for web, DNS and jump SSH). Doing NAT on the router that owns the block means no device has to choose a public address based on which ISP is in use, and the public addresses end at the router, so no public-block static routes or PA tracking are needed. The PA still enforces every security rule; NAT itself isn't a security control.

Each router also has a cost-500 link to the ASA (R1 G0/1/1 -> E1/5, R2 G0/1/1 -> E1/2), used only when the PA is down. R2 is the backup OOB management router (172.16.0.34).

R1

- no ip domain lookup, ip domain name gruppesechs.sitict.net, service password-encryption, service timestamps log datetime msec localtime, clock timezone SGT 8 0, banner  
- enable algorithm-type scrypt secret 4Venison2  
- username bigDonkBonkAdmin privilege 15 algorithm-type scrypt secret Venison2Beef  
- vrf definition OOB, address-family ipv4  
- int g0/1/0, management gateway (cabled to DSW1 Gi1/0/22)  
  - ip add 172.16.0.33 255.255.255.240, vrf forwarding OOB.  
- int g0/0/0, connected to isp  
  - ip add 172.17.9.93 255.255.255.252  
  - ip access-group WAN-IN in  
  - ip nat outside  
- int g0/0/1, to PA eth1/5  
  - ip add 172.16.0.90 255.255.255.248, ip nat inside  
  - ospf 1 area 0, cost 10, while r2’s g0/0/1’s costs is 20  
- int g0/1/1, PA bypass to ASA E1/5 (was the R1-R2 link until 5 Nov)  
  - no vrf forwarding Mgmt-vrf first: the lab image puts this port in the factory Mgmt-vrf, which keeps it out of OSPF. That command deletes the IP, so the IP is set straight after.  
  - ip add 172.16.0.109 255.255.255.252, ip nat inside  
  - ospf 1 area 0, cost 500 (only used when the PA is down)  
- NAT (moved from the PA, 5 Nov)  
  - ip access-list extended NAT-SOURCES: HR 172.16.0.56/29, IT 172.16.0.64/29, Txn .76, DNS .83, jump hosts .84/.85 (second layer behind the PA; Staff/ATM can't be translated even if a firewall rule were wrong)  
  - ip nat pool PUB-PAT 203.149.210.185 203.149.210.185 prefix-length 29  
  - ip nat inside source list NAT-SOURCES pool PUB-PAT overload  
  - ip nat inside source static tcp 172.16.0.82 80 / 443 -> 203.149.210.189 80 / 443 extendable  
  - ip nat inside source static udp and tcp 172.16.0.83 53 -> 203.149.210.190 53 extendable  
  - ip nat inside source static tcp 172.16.0.84 22 -> 203.149.210.185 2222 extendable  
- all routed ports: ip flow monitor bank-flow-monitor input, no ip redirects, no ip proxy-arp  
- ip sla 1  
  - ip sla sends periodic icmp pings out g0/0/0 to isp gateway to ensure its working, so we can track to see if the traffic is running  
  - icmp-echo 172.17.9.94 source-interface g0/0/0  
  - frequency 5  
- ip sla schedule 1 life forever start-time now  
- track 1 ip sla 1 reachability  
- ip sla 2 / track 2 (ping the PA .92) removed 5 Nov: they only guarded the public-block routes to the PA, which aren't needed with NAT on the router  
- ip routes  
  - ip route 0.0.0.0 0.0.0.0 172.17.9.94 track 1 (REQUIRED, source of the OSPF default). Must point at the ISP .94, never .93 (R1 itself).  
  - ip route 203.149.210.184 255.255.255.248 Null0 (OPTIONAL, drops traffic for unused public IPs instead of looping it back to the ISP)  
- aaa  
  - tacacs server TACACS1  
    - address ipv4 172.16.0.44  
    - key bigBoy@ServerPassword1  
  - tacacs server TACACS2  
    - address ipv4 172.16.0.45  
    - key bigBoy@ServerPassword2  
  - aaa group server tacacs+ tac-group, server name TACACS1, server name TACACS2, ip vrf forwarding OOB, ip tacacs source-interface G0/1/0  
  - aaa authentication login default group tac-group local  
  - aaa authentication login CONSOLE group tac-group local  
  - aaa authorization exec default group tac-group local  
    - lands user in exec shell  
  - aaa authorization config-commands  
    - perform authorization checks on every command in config t  
  - aaa authorization commands 15 default group tac-group local  
    - validate every priv level 15 command   
  - aaa accounting exec default start-stop group tac-group  
  - aaa accounting commands 15 default stop-only group tac-group  
    - records and send audit trail of every priv 15 command  
- line vty 0 2  
  - login authentication default  
  - access-class MGMT-ONLY in vrf-also  
  - transport input ssh  
  - exec-timeout 10 0  
- line vty 3 15  
  - transport input none  
- line con 0  
  - login authentication CONSOLE  
  - exec-timeout 10 0  
- ip ssh version 2  
  - ip ssh time-out 60  
  - ip ssh authentication-retries 3  
- ntp server vrf OOB 172.16.0.44 prefer, 172.16.0.45  
- crypto key generate rsa modulus 2048  
- syslog  
  - logging host 172.16.0.44 vrf OOB, logging host 172.16.0.45 vrf OOB, logging trap informational  
- ospf   
  - router-id 1.1.1.1, 2.2.2.2 for r2  
  - auto-cost reference-bandwidth 10000  
  - passive-interface default  
  - no passive-interface g0/0/1  
  - no passive-interface g0/1/1  
  - network 172.16.0.88 0.0.0.7 area 0, network 172.16.0.108 0.0.0.3 area 0  
  - default-information originate metric 10 metric-type 1  
    - only advertises the default route while R1 has one, i.e. while track 1 is up. If ISP1 goes down, the tracked static route is withdrawn and R1 stops advertising the default, so everything follows R2's backup default (metric 50) to ISP2.  
    - metric-type 1 (E1): each device adds its own path cost to the metric, so with both ISPs up every device picks R1, and with the PA down the ASA picks R1 over its direct link (10 + 500) before R2 (50 + 500).  
- ![][image1]

R2

- same base, AAA, SSH, NTP, syslog, NetFlow and parser view config as R1, router-id 2.2.2.2  
- int g0/0/0, WAN to ISP2 (5 Nov): ip add 172.17.10.93 255.255.255.252, ip access-group WAN-IN in, ip nat outside  
- int g0/0/1, to PA eth1/2: ip add 172.16.0.91 255.255.255.248, ip nat inside, ospf cost 20  
- int g0/1/0, OOB (cabled to DSW2 Gi1/0/22): ip add 172.16.0.34 255.255.255.240, vrf forwarding OOB  
- int g0/1/1, PA bypass to ASA E1/2: no vrf forwarding Mgmt-vrf, no ip dhcp client client-id, ip add 172.16.0.113 255.255.255.252, ip nat inside, ospf cost 500  
- WAN-IN (same pattern as R1, for .93 and 129.126.142.184/29)  
- ip sla 1 (icmp-echo 172.17.10.94 source g0/0/0), track 1  
- ip route 0.0.0.0 0.0.0.0 172.17.10.94 track 1 (REQUIRED, source of the backup OSPF default)  
- ip route 129.126.142.184 255.255.255.248 Null0 (OPTIONAL)  
- NAT: same NAT-SOURCES ACL and mappings as R1 on 129.126.142.185 / .189 / .190  
- ospf: no passive-interface g0/0/1, g0/1/1; networks 172.16.0.88/29, .112/30; default-information originate metric 50 metric-type 1

router acls

- mgmt-only permits .44 and .45 to access line vty 0 2  
  - when admin sshs into .84 \> ssh into management ip in the linux shell (eg. .33 is r1)  
  - when the linux sends the packet, its source ip would be either tacacs1(.44) or tacacs2(.45)  
  - thats why we permit .44 or .45  
- WAN-IN is an ingress acl to filter outside traffic coming in  
  - denies port 22 and 23, permits icmp echo replies from .94 to .93 (ip sla), permits packets destined for the router's own public block (203.149.210.184/29 on R1, 129.126.142.184/29 on R2; checked before NAT), permit established tcp return traffic to .93, then deny all  
  - web server is .189, dns is .190  
  - open issue (P6): the tcp/22 and tcp/23 denies in the macro reference host 172.17.9.94 (the ISP). They should reference R1's own IP .93.

router PTSO

- PA bypass links: R1 G0/1/1 -> ASA E1/5 (172.16.0.108/30) and R2 G0/1/1 -> ASA E1/2 (172.16.0.112/30), both ip ospf cost 500. Used only when the PA is down; the router then NATs the ASA's traffic the same way.  
- ISP failover: R1/ISP1 down -> R1's default is withdrawn -> R2's backup default (metric 50) takes over -> traffic leaves as 129.126.142.185.  
- R2's link to the PA down while R1 is up: nothing changes, traffic stays on R1 (R2 only carries traffic when R1 is down).


## **NetFlow v9 configuration for Routers and Distribution Switches**

flow record bank-flow-record

- description packet flow record for monitoring  
- match ipv4 source address  
- match ipv4 destination address  
- match transport source-port  
- match transport destination-port  
- match ipv4 protocol  
- match ipv4 tos  
- match interface input  
- collect interface output  
- collect counter bytes long  
- collect counter packets long  
- collect timestamp absolute first  
- collect timestamp absolute last

flow exporter bank-flow-exporter-1

- description export flows to tacacs1 .44  
- destination 172.16.0.44 vrf OOB  
- source G0/1/0 for routers, source vlan 99 for DSW  
- transport udp 2055  
- export-protocol netflow-v9  
- template data timeout 60

flow exporter bank-flow-exporter-2

- description export flows to tacacs2 .45  
- destination 172.16.0.45 vrf OOB  
- source G0/1/0 for routers, source vlan 99 for DSW  
- transport udp 2055  
- export-protocol netflow-v9  
- template data timeout 60

flow monitor bank-flow-monitor

- exporter bank-flow-exporter-1  
- exporter bank-flow-exporter-2   
- record bank-flow-record  
- cache timeout active 60  
- cache timeout inactive 15

apply flow monitor to interfaces

- ip flow monitor bank-flow-monitor input  
- DSW interfaces: vlan 100, vlan 23, 46, 61, 67  
- router interfaces: g0/0/0, g0/0/1, g0/1/1 (R1 and R2)

lecture recommends NetFlow collector by mangeEngine which we can implement in tacacs

## **Firepower 1010**

- hostname ASA-1010, domain-name gruppesechs.sitict.net, clock timezone Singapore, banner motd  
- clear configure aaa, crypto key zeroize rsa, crypto key generate rsa modulus 2048 noconfirm  
- enable password 4Venison2  
- username bigDonkBonkAdmin password Venison2Beef privilege 15  
  - passwords are typed in cleartext, the ASA hashes them itself  
- clear configure dhcpd  
  - remove factory defaults for internal dhcp daemon  
- object network obj\_any  
  - no nat (any,outside) dynamic interface, removes default dynamic NAT. Without this the ASA PATs HR/IT to .106 and their internet traffic is blackholed. Only the routers (R1/R2) do NAT.  
- int vlan1, no nameif, no security-level, no ip, shutdown  
- int e1/1, connect to PA eth1/3 (routed)  
  - nameif outside, security-level 0  
  - ip add 172.16.0.106 255.255.255.252, speed 1000, duplex full, ospf cost 10  
- int e1/2, connect to R2 g0/1/1 (routed, PA bypass)  
  - nameif wan-backup, security-level 0  
  - ip add 172.16.0.114 255.255.255.252, speed 1000, duplex full, ospf cost 500  
- int e1/3, connect to DSW1 g1/0/11 (routed)  
  - nameif inside, security-level 100  
  - ip add 172.16.0.97 255.255.255.252, speed 1000, duplex full, ospf cost 10  
  - point-to-point link to DSW1 only (172.16.0.96/30)  
- int e1/4, connect to DSW2 g1/0/11 (routed, added 5 Nov)  
  - nameif inside2, security-level 100  
  - ip add 172.16.0.101 255.255.255.252, speed 1000, duplex full, ospf cost 10  
  - its own /30, so it doesn't clash with e1/3. Same INSIDE\_IN ACL as inside.  
- int e1/5, connect to R1 g0/1/1 (routed, PA bypass, added 5 Nov)  
  - nameif wan-backup2, security-level 0  
  - ip add 172.16.0.110 255.255.255.252, speed 1000, duplex full, ospf cost 500  
- int e1/7 and e1/8, unused, shutdown  
  - e1/7 used to carry the DMZ backup (Vlan92 dmz .86). Removed 5 Nov: the DMZ belongs to the PA only.  
- int e1/6, connect directly to Proxmox eno2 (vmbr1)  
  - switchport access vlan 69  
- int vlan 69  
  - nameif server, security-level 60  
  - ip add 172.16.0.73 255.255.255.248  
- int management1/1, port above console port (cabled to DSW2 g1/0/23)  
  - for oob management  
  - security-level 100, ip add 172.16.0.42 255.255.255.240, management-only, nameif management  
- route management 0.0.0.0 0.0.0.0 172.16.0.33 1 (REQUIRED, management-only interface is not in OSPF)  
- ospf  
  - router ospf 1  
    - router-id 5.5.5.5  
    - network 172.16.0.96 255.255.255.252 area 0, asa to dsw1  
    - network 172.16.0.100 255.255.255.252 area 0, asa to dsw2  
    - network 172.16.0.104 255.255.255.252 area 0, asa to pa  
    - network 172.16.0.112 255.255.255.252 area 0, asa to r2  
    - redistribute connected subnets  
      - OSPF is not run on the server SVI. On the Firepower 1010, OSPF on switchport-backed SVIs throws `ip_multicast_ctl failed to get channel` errors. 172.16.0.72/29 is advertised as an external route instead.  
    - log-adj-changes  
  - the data-plane default route (via PA .105) is learned from OSPF  
- object creation for ACL  
  - object network STAFF, subnet 172.16.0.0 255.255.255.224  
  - object network ATM, subnet 172.16.0.48 255.255.255.248  
  - object network HR, subnet 172.16.0.56 255.255.255.248  
  - object network IT, subnet 172.16.0.64 255.255.255.248  
  - object network SERVERS, subnet 172.16.0.72 255.255.255.248  
  - object network DMZ-SUBNET, subnet 172.16.0.80 255.255.255.248  
  - object network INTERNAL, subnet 172.16.0.0 255.255.255.128  
  - object network FILE-SVR host .74, ACCT-SVR host .75, TXN-SVR host .76, DHCP-SVR host .77  
  - object network WEB-SVR host .82, DNS-SVR host .83, JUMP-SVR host .84, JUMP2-SVR host .85  
  - object-group network STAFF-ATM (STAFF, ATM)  
  - object-group network HR-IT (HR, IT)  
  - object-group network DSWs (172.16.0.98, .102)  
  - object-group network DSW-SVIS (every DSW address that can source a DHCP relay: .2 .3 .50 .51 .58 .59 .66 .67 .98 .102)  
  - object-group network JUMP-SVRs (JUMP-SVR, JUMP2-SVR)  
- time-range (to be given on demo day)  
  - time-range TXN-WINDOW  
    - periodic weekdays 01:00 to 03:00  
- ACLs  
  - INSIDE\_IN, from dsw to asa  
    - permit ospf  
    - permit dhcp relay from DSW-SVIS to DHCP-SVR, udp port 67  
    - allow HR to access file server, tcp port 445 and icmp  
    - allow staff and atm to access accounts and txn servers, ip  
    - allow HR-IT to DNS-SVR udp/tcp 53, and WEB-SVR tcp 80/443  
    - allow IT to ssh to JUMP-SVRs  
    - deny ip any to SERVERS log, deny ip any to DMZ-SUBNET log  
    - deny STAFF-ATM to any, (internet), ip  
    - allow HR-IT to access any, (PA, routers, internet), ip  
    - default deny any log  
  - SERVER\_IN, from supermicro to asa  
    - permit dhcp replies from DHCP-SVR bootps to DSW-SVIS bootps (must stay above the INTERNAL deny)  
    - deny any servers access to INTERNAL, ip  
    - permit txn server tcp 80/443 to any, and udp 53 to DNS-SVR, with time-range TXN-WINDOW  
    - default deny any log  
  - OUTSIDE\_IN, PA/R2 to ASA  
    - permit ospf  
    - default deny any log  
    - DMZ\_IN and the OUTSIDE\_IN permits to WEB-SVR/DNS-SVR/JUMP-SVR were removed 5 Nov with the ASA's DMZ interface (they only served the standby DMZ path)  
- access-group INSIDE\_IN in interface inside  
- access-group INSIDE\_IN in interface inside2  
- access-group SERVER\_IN in interface server  
- access-group OUTSIDE\_IN in interface outside  
- access-group OUTSIDE\_IN in interface wan-backup  
- access-group OUTSIDE\_IN in interface wan-backup2  
- MPF config  
  - policy-map global\_policy  
    - class inspection\_default  
      - inspect icmp, inspect icmp error  
- aaa  
  - aaa-server TACACS protocol tacacs+  
    - initialize aaa server group name TACACS that use tacacs+  
  - aaa-server TACACS (management) host 172.16.0.44  
    - adds 172.16.0.44 to the aaa group, (management) parameter to ensure all tacacs+ packets(ssh oob management) will route there  
    - key bigBoy@ServerPassword1  
  - aaa-server TACACS (management) host 172.16.0.45  
    - key bigBoy@ServerPassword2  
  - aaa authentication ssh console TACACS LOCAL  
  - aaa authentication enable console TACACS LOCAL  
  - aaa accounting command TACACS  
  - aaa authorization command TACACS LOCAL is **disabled** (commented out in the macro). With TACACS not answering it falls back to LOCAL and locks you out. Re-enable only after tac\_plus is tested, with a console session open.  
- ssh  
  - ssh 172.16.0.44 255.255.255.255 management  
    - equivalent to vty ACLs, specifies which ips can ssh to int management  
  - ssh 172.16.0.45 255.255.255.255 management  
  - ssh timeout 10, disconnects idle ssh after 10 mins  
  - no http server enable  
- ntp server 172.16.0.44 source management prefer, 172.16.0.45 source management  
  - logging enable, logging timestamp, logging trap informational, logging host management 172.16.0.44, logging host management 172.16.0.45  
  - asa needs ntp here also for txn time range, timestamps for logs  
  - .44/.45 run chrony (VM102/VM106), which syncs from the internet pool through the PA  
- PTSO  
  - Ethernet1/2 (wan-backup) to R2 G0/1/1 (172.16.0.114/30) and Ethernet1/5 (wan-backup2) to R1 G0/1/1 (172.16.0.110/30), ospf cost 500: used only when the PA is down  
  - Vlan69 (server, security-level 60): Internal Servers Gateway (172.16.0.73/29) on Ethernet1/6 (cabled directly to Proxmox eno2), redistributed into OSPF  
  - Ethernet1/3 (inside) and Ethernet1/4 (inside2): one routed /30 to each DSW, both active  
  - No NAT on the ASA (5 Nov). The old standby PAT was removed: with the PA down, traffic reaches R1 directly un-NAT'd and R1 translates it as usual  
  - No DMZ interface (removed 5 Nov). If the PA fails, users and internal servers keep internet via R1 (wan-backup2), or R2 if R1 is also down; the DMZ is down by design.

## **PA-440**

- hostname PA-440, domain gruppesechs.sitict.net, login-banner  
- type static ip-address 172.16.0.41 netmask 255.255.255.240 default-gateway 172.16.0.33  
  - type static disables the DHCP mode on the management interface of the palto, to enable us to assign the static ip address for management  
  - this is the MGT port, cabled to DSW1 g1/0/23  
  - default-gateway is REQUIRED: the management plane is separate from the data-plane virtual router that runs OSPF  
  - all oob management originates from the jump server(tacacs1 or tacacs2).  
- permitted-ip 172.16.0.44/32, permitted-ip 172.16.0.45/32  
  - allow only tacacs1 and tacacs2  
- service disable-telnet yes, \-http yes, \-ssh no, \-https no  
- ntp-servers, primary 172.16.0.44, secondary 172.16.0.45 (chrony on VM102/VM106)  
- set mgt-config users bigDonkBonkAdmin permissions role-based superuser yes, password Venison2Beef  
- set mgt-config users donkBonkEng1 / donkBonkEng2 permissions role-based superreader yes  
- network profiles interface-management-profile PING-ONLY ping yes  
  - by default, palto eth1/1 to 1/8 drops traffic destined for the firewalls own ip address  
  - if r1 wants to ping 172.16.0.92 or 172.16.0.105, it will drop the ping.  
  - applied to eth1/3, eth1/6, eth1/7, vlan.10 and vlan.92  
  - this profile enables the ping, probably should remove to prevent inside attacker recon?  
- interfaces  
  - eth1/1: unused, link-state down (avoids the ZTP WAN port)  
  - eth1/2: layer2, lldp enable no, towards R2 g0/0/1  
  - eth1/3: layer3 ip 172.16.0.105/30, towards ASA e1/1  
    - runs layer 3 for routing and it only connects 1 cable towards asa  
  - eth1/4: layer2, lldp enable no, cabled directly to Proxmox eno5 (vmbr2, DMZ VLAN 92)  
  - eth1/5: layer2, lldp enable no, towards R1 g0/0/1  
    - disable lldp on untrusted outside ports, prevent recon  
  - eth1/6: layer3 ip 172.16.0.117/30, towards DSW1 g1/0/12 (ASA bypass, up)  
  - eth1/7: layer3 ip 172.16.0.121/30, towards DSW2 g1/0/12 (ASA bypass, up)  
    - changed 5 Nov from layer2 ports bridged into vlan.100. Bridged, they formed a Layer 2 loop with Po5 (the PA runs no STP), so they had to stay down. As separate routed /30s there is nothing to loop.  
  - eth1/8: unused, **link-state down** (was the internal-server backup, removed 5 Nov)  
- vlan interfaces  
  - vlan.10 ip 172.16.0.92/29 (network vlan OUTSIDE interface \[ ethernet1/2 ethernet1/5 \])  
    - similar for asa’s vlan 69, this is for the transit between routers and pa  
    - we have to place them in a vlan and layer 2, as if both eth1/5 and eth1/2 are given ip address in the .88/29 subnet, it will cause overlap  
  - vlan.92 ip 172.16.0.81/29 (network vlan VL-DMZ interface \[ ethernet1/4 \]), dmz’s gateway  
  - vlan.100 and vlan.69 removed 5 Nov (pa440\_cleanup.ttl deletes them on a live box)  
- set zone  
  - outside network layer3 vlan.10, towards routers  
  - outside-l2 network layer2 \[ eth1/2 eth1/5 \]  
    - outside requires layer 2 and 3 because eth1/5 and eth1/2 are layer 2 interfaces, so we put in a dedicated outside-l2 zone so layer 2 frames are inspected between routers and pa. vlan.10 also holds a layer 3 ip address so we need a layer 3 zone as well so traffic can be inspected and routed.  
  - inside network layer3 \[ eth1/3 eth1/6 eth1/7 \], towards asa and the two DSW bypass links  
  - dmz network layer3 vlan.92, towards supermicro  
  - dmz-l2 network layer2 eth1/4  
  - zones inside-l2, server and server-l2 removed 5 Nov  
- virtual router and ospf  
  - network virtual-router default interface \[ ethernet1/3 ethernet1/6 ethernet1/7 vlan.10 vlan.92 \]   
    - the PA has no interface in 172.16.0.72/29; it learns it from the ASA via OSPF, so traffic to the internal servers always goes through the ASA  
  - network virtual-router default protocol ospf enable yes   
  - network virtual-router default protocol ospf router-id 6.6.6.6   
  - area 0.0.0.0 interface ethernet1/3 enable yes passive no metric 10 (to ASA)  
  - area 0.0.0.0 interface vlan.92 enable yes passive no metric 10 (DMZ)  
  - area 0.0.0.0 interface vlan.10 enable yes passive no metric 10 (to R1/R2)  
  - area 0.0.0.0 interface ethernet1/6 and ethernet1/7 enable yes passive no metric 500 (ASA bypass to DSW1/DSW2)  
    - interface \<name\> enable yes, enables ospf on that interface, like ip ospf 1 area 0  
    - passive no/yes, just makes the interface not passive which will send ospf packets  
  - ospf is enabled here so that routing towards R1/R2 is achieved, and can switch in the event r1’s link fails.  
  - no static default (DEFAULT-OUT removed 5 Nov): OSPF supplies it from R1, or R2 when R1/ISP1 is down. An untracked static via .90 would blackhole traffic with R1 down  
- object creation  
  - address STAFF 172.16.0.0/27, ATM 172.16.0.48/29, HR-NET 172.16.0.56/29, IT-NET 172.16.0.64/29 (HR-NET/IT-NET named to avoid clashing with GeoIP country codes)  
  - address SERVERS 172.16.0.72/29, DMZ-SUBNET 172.16.0.80/29  
  - address FILE-SVR .74, ACCT-SVR .75, TXN-SVR .76, DHCP-SVR .77  
  - address WEB-SVR .82, DNS-SVR .83, JUMP-SVR .84, JUMP2-SVR .85  
  - PUB-PAT / PUB-WEB / PUB-DNS removed 5 Nov (no NAT on the PA)  
  - address DNS-FORWARDERS-1 8.8.8.8/32, DNS-FORWARDERS-2 1.1.1.1/32, ICT-DNS 172.18.4.32/32 (lab caching DNS, needed for other racks' names)  
  - address-group STAFF-ATM \[ STAFF ATM \], HR-IT \[ HR-NET IT-NET \], INTERNET-USERS \[ HR-NET IT-NET \]  
  - service TCP-2222 removed 5 Nov: the routers map public tcp/2222 to .84:22, so the PA only sees tcp/22. Remote admins still use ssh \-p 2222 bigDonkBonkAdmin@203.149.210.185  
  - DSW1-SVI, DSW2-SVI, DSWs and UDP-67 removed 5 Nov (only the deleted PTSO-DHCP-RELAY rule used them)  
- schedule TXN-WINDOW recurring weekly monday-friday 01:00-03:00 \[ final time given during demo \]  
- rules  
  - set rulebase security rules \<rule name\> from \<zone\> to \<zone\> source \<address\> destination \<address\> source-user any category any application \<app-id\> service \<port\> schedule \<name\> rule-type \<interzone\> action \<allow/deny\> log-end \<yes/no\>  
  - ALLOW-OSPF from \[ outside inside \] to \[ outside inside \] application ospf, allow, log-end no  
    - allow ospf between zones so routers, firewalls and dsw can maintain ospf neighbors  
  - INBOUND-WEB from outside to dmz, destination WEB-SVR, application \[ web-browsing ssl \], service application-default, allow  
    - allows outside traffic to access dmz. The router already translated .189 to 172.16.0.82, so the rule uses the real address  
    - strict filtering of application \[ web-browsing ssl \] which is HTTP and HTTPS respectively  
  - INBOUND-DNS from outside to dmz, destination DNS-SVR, application dns, allow  
  - JUMP-SSH-IN from outside to dmz, destination JUMP-SVR, application ssh, service application-default, allow  
    - allows external admins to ssh into the jump host. The router maps public tcp/2222 to .84:22, the PA checks that it really is ssh  
  - INTERNAL-TO-DNS from inside to dmz, source INTERNET-USERS, destination DNS-SVR, application dns, allow  
  - INTERNAL-TO-WEB from inside to dmz, source INTERNET-USERS, destination WEB-SVR, application \[ web-browsing ssl \], allow  
  - IT-TO-JUMP-SSH from inside to dmz, source IT-NET, destination \[ JUMP-SVR JUMP2-SVR \], application ssh, allow  
  - PTSO-HR-TO-FILE, PTSO-STAFF-ATM-SVR and PTSO-DHCP-RELAY (inside -> server) removed 5 Nov with the PA's server interface. Internal-server access is enforced by the ASA only.  
  - HR-IT-INTERNET from inside to outside, source INTERNET-USERS, application \[ web-browsing ssl dns ping \], allow  
    - also covers users arriving over the eth1/6/7 bypass (same zone inside) when the ASA is down  
  - TXN-INTERNET-TIMED from inside to \[ outside dmz \], source TXN-SVR, application \[ web-browsing ssl dns \], schedule TXN-WINDOW, allow  
  - DMZ-OUTBOUND-NTP from dmz to outside, source \[ JUMP-SVR JUMP2-SVR \], application \[ ntp dns \], allow  
  - DNS-SVR-FORWARD from dmz to outside, source DNS-SVR, destination \[ ICT-DNS DNS-FORWARDERS-1 DNS-FORWARDERS-2 \], application dns, allow  
    - lets the DNS server recurse for HR/IT through its forwarders  
  - DENY-ALL rule-type interzone from any to any, deny, log-end yes  
    - default deny all at the end  
- NAT: none on the PA (5 Nov). The routers do it: R1 for 203.149.210.184/29, R2 for 129.126.142.184/29. The PA sees real inside and outside addresses in both directions, so its rules use the real server objects. pa440\_cleanup.ttl deletes the old NAT-OUTBOUND, NAT-WEB, NAT-WEB-HTTPS, NAT-DNS and NAT-JUMP-SSH rules.  
- AAA  
  - set shared server-profile tacplus TACACS server JUMP address 172.16.0.44 secret bigBoy@ServerPassword1  
  - set shared server-profile tacplus TACACS server JUMP2 address 172.16.0.45 secret bigBoy@ServerPassword2  
  - set shared server-profile tacplus TACACS protocol PAP, timeout 3, retry 2  
  - the macro does not create or apply an authentication profile yet (planned: AUTH-TACACS + AUTH-LOCAL in sequence SEQ-ADMIN, applied with set deviceconfig system authentication-profile). Do this only after tac\_plus is tested.  
  - set shared log-settings syslog SYSLOG-JUMP server JUMP server 172.16.0.44 transport UDP port 514 format BSD facility LOG\_USER   
  - set shared log-settings syslog SYSLOG-JUMP server JUMP2 server 172.16.0.45 transport UDP port 514 format BSD facility LOG\_USER  
- rebuild: run pa440\_cleanup.ttl first, then pa440.ttl (set commands are additive)  
- live differences (see HANDOVER.md section 6): HR-IT-INTERNET is still any/any live, and the temporary TEMP-SVR-UPDATES / TEMP-SVR-NAT install rules are still present (after the 5 Nov NAT change, TEMP-SVR-NAT is replaced by router NAT-SOURCES lines: HANDOVER.md 8.4)  
- PTSO  
  - Shifted ethernet1/1 to ethernet1/5: Avoids the ZTP WAN port eth1/1 completely; vlan.10 (172.16.0.92/29, metric 10\) bridges ethernet1/5 (to R1 G0/0/1) and ethernet1/2 (to R2 G0/0/1)  
  - PTSO ASA bypass (5 Nov): eth1/6 (172.16.0.117/30 to DSW1 G1/0/12) and eth1/7 (172.16.0.121/30 to DSW2 G1/0/12), routed, zone inside, OSPF metric 500. Both up. Used automatically if the ASA is unreachable from both DSWs; the PA then filters users with the same inside-zone rules. Internal servers are unreachable during an ASA failure (by design).  
  - No internal-server interface (vlan.69/eth1/8 removed 5 Nov). The PA owns only the DMZ.

## **Users**

Our RBAC design relies on both Privilege Levels and Views given to the Network Engineers and Manager.

| User | Username | Cisco Devices | Firepower 1010 | PA-440 |
| :---- | :---- | :---- | :---- | :---- |
| Network Manager | bigDonkBonkAdmin  | Privilege 15 | Privilege 15 | superuser |
| Network Engineer 1 | donkBonkEng1 | eng-view | Privilege 5 | superreader |
| Network Engineer 2 | donkBonkEng2 | eng-view | Privilege 5 | superreader |

- We use the default Cisco/Firepower Privilege level 15 for the Network Manager to give them full access to commands of the cisco devices.  
- We created an eng-view(cisco)/privilege 5(firepower) for the Network Engineers to provide them with commands related to troubleshooting and basic interface configuration.

Cisco Devices configuration

- parser view eng-view  
  - secret engViewTriple3  
  - commands exec include all show  
  - commands exec include ping  
  - commands exec include traceroute  
  - commands exec include clear  
    - allows troubleshooting and monitoring  
  - commands configure include interface  
  - commands interface include shutdown  
  - commands interface include no shutdown  
  - commands interface include switchport (switches only, not on R1/R2)  
  - commands exec include configure terminal  
  - commands exec include configure  
    - commands to allow basic interface configuration

create accounts on cisco

- username bigDonkBonkAdmin privilege 15 algorithm-type scrypt secret Venison2Beef  
- username donkBonkEng1 view eng-view algorithm-type scrypt secret Beef2Venison  
- username donkBonkEng2 view eng-view algorithm-type scrypt secret Beef3Venison

Firepower

- username bigDonkBonkAdmin password Venison2Beef privilege 15  
- username donkBonkEng1 password Beef2Venison privilege 5  
- username donkBonkEng2 password Beef3Venison privilege 5  
- privilege show level 5 mode exec command running-config  
- privilege show level 5 mode exec command route  
- privilege show level 5 mode exec command ospf  
- privilege show level 5 mode exec command interface  
- privilege show level 5 mode exec command access-list  
- privilege cmd level 5 mode exec command ping  
- privilege cmd level 5 mode exec command traceroute

PA-440

- set mgt-config users bigDonkBonkAdmin permissions role-based superuser yes  
- set mgt-config users bigDonkBonkAdmin password  
  - Venison2Beef  
- set mgt-config users donkBonkEng1 permissions role-based superreader yes  
- set mgt-config users donkBonkEng1 password  
  - Beef2Venison  
- set mgt-config users donkBonkEng2 permissions role-based superreader yes  
- set mgt-config users donkBonkEng2 password  
  - Beef3Venison

TACACS config, edit authorization.xml in C:\\ProgramData\\[TACACS.net](http://TACACS.net)\\config or tac\_plus.conf in linux \- need to test  
\# (Use bigBoy@ServerPassword1 on .44, and bigBoy@ServerPassword2 on .45)  
key \= "bigBoy@ServerPassword1"

\# \--- GROUP 1: Network Manager (Full Admin) \---  
group \= net\_manager {  
    default service \= permit  
    service \= exec {  
        priv-lvl \= 15  
    }  
}

\# \--- GROUP 2: Network Engineers (Restricted RBAC & View) \---  
group \= net\_engineers {  
    default service \= deny  
    service \= exec {  
        priv-lvl \= 7  
        cli-view-name \= "eng-view"  
    }  
    \# Permitted CLI commands for Engineers  
    cmd \= show { permit .\* }  
    cmd \= ping { permit .\* }  
    cmd \= traceroute { permit .\* }  
    cmd \= configure { permit "terminal" }  
    cmd \= interface { permit .\* }  
    cmd \= description { permit .\* }  
    cmd \= shutdown { permit .\* }  
    cmd \= no { permit "shutdown" }  
    \# Everything else (reload, write erase, username, aaa, access-list) is implicitly DENIED and logged\!  
}

\# \--- USER ACCOUNTS \---  
user \= bigDonkBonkAdmin {  
    member \= net\_manager  
    login \= cleartext "Venison2Beef"  
    enable \= cleartext "4Venison2"  
}

user \= donkBonkEng1 {  
    member \= net\_engineers  
    login \= cleartext "Beef2Venison"  
}

user \= donkBonkEng2 {  
    member \= net\_engineers  
    login \= cleartext "Beef3Venison"  
}

**done (4 Nov, details in HANDOVER.md section 8):**

- all servers built on the supermicro: web (vm100, nginx), DNS (vm101, BIND, forwards to the ICT DNS 172.18.4.32 then 8.8.8.8), file (vm103, samba tcp/445 only), accounts (vm104, nginx https portal), transactions (vm105), DHCP (vm107, isc-dhcp-server)  
- chrony on vm102 and vm106, in /etc/chrony/chrony.conf  
  - pool [0.sg.pool.ntp.org](http://0.sg.pool.ntp.org) iburst maxsources 4  
    - uses singapore ntp pool by dmz interface  
  - server 172.16.0.45 iburst on vm102 (172.16.0.44 on vm106)  
  - allow 172.16.0.32/28  
  - local stratum 10 orphan  
  - sudo systemctl restart chrony, restart the service

**still need to do:**

- tac\_plus on vm102 and vm106 (test before relying on it, keep a console open)  
- NetFlow Analyzer by ManageEngine on tacacs1 and tacacs2  
- Wazuh(SIEM) on tacacs1 and tacacs2 (agents need ASA SERVER\_IN + PA inside -> dmz permits for tcp/1514-1515)  
- remove the temporary server internet rules after the last install  
- final hardening: remove the temporary ICMP permits on the DSWs, re-enable ASA aaa authorization command after TACACS works  
- set the final TXN-WINDOW on demo day (ASA time-range, PA schedule)

[image1]: <data:image/png;base64,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>