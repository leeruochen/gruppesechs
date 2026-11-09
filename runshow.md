WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW1 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username:
Username:
Username: bigDonkBonkAdmin
Password:

SW1>en
Password:
SW1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
SW1(config)#int range g1/0/5-16
SW1(config-if-range)#no switchport port-security mac-address sticky
SW1(config-if-range)#switchport port-security violation restrict
SW1(config-if-range)#switchport port-security aging time 5
SW1(config-if-range)#switchport port-security aging type inactivity
SW1(config-if-range)#
SW1(config-if-range)#clear port-security sticky
                      ^
% Invalid input detected at '^' marker.

SW1(config-if-range)#exit
SW1(config)#clear port-security sticky
              ^
% Invalid input detected at '^' marker.

SW1(config)#exit
SW1#clear port-s
SW1#clear port-security sticky
SW1#WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW3 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin
Password:

SW3>en
Password:
SW3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
SW3(config)#int range g1/0/5-6
SW3(config-if-range)#no switchport port-security mac-address sticky
SW3(config-if-range)#switchport port-security violation restrict
SW3(config-if-range)#switchport port-security aging time 5
SW3(config-if-range)#switchport port-security aging type inactivity
SW3(config-if-range)#
SW3(config-if-range)#exit
SW3(config)#exit
SW3#clear port
SW3#clear port-security sticky
SW3#
SW2>en
Password:
SW2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
SW2(config)#int range g1/0/5-7
SW2(config-if-range)#no switchport port-security mac-address sticky
SW2(config-if-range)#switchport port-security violation restrict
SW2(config-if-range)#switchport port-security aging time 5
SW2(config-if-range)#switchport port-security aging type inactivity
SW2(config-if-range)#
SW2(config-if-range)#exit
SW2(config)#exit
SW2#clear port
SW2#clear port-security sticky
SW2#WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW4 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin
Password:

SW4>en
Password:
SW4#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
SW4(config)#int range g1/0/5-7
SW4(config-if-range)#no switchport port-security mac-address sticky
SW4(config-if-range)#switchport port-security violation restrict
SW4(config-if-range)#switchport port-security aging time 5
SW4(config-if-range)#switchport port-security aging type inactivity
SW4(config-if-range)#
SW4(config-if-range)#exit
SW4(config)#exit
SW4#clear port
SW4#clear port-security sticky
SW4#
SW4#show running
SW4#show running-config

Building configuration...

Current configuration : 16703 bytes
!
! Last configuration change at 21:37:30 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname SW4
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
logging buffered 64000 informational
no logging console
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip tacacs source-interface Vlan99
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
!
!
!
clock timezone SGT 8 0
switch 1 provision c9200l-24t-4g
!
!
!
!
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
ip dhcp snooping vlan 67
no ip dhcp snooping information option
ip dhcp snooping
ip arp inspection vlan 67
ip arp inspection validate src-mac dst-mac ip
login delay 2
login on-failure log
login on-success log
vtp version 1
!
!
!
!
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-3845540381
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-3845540381
 revocation-check none
 rsakeypair TP-self-signed-3845540381
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-3845540381
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  31312F30 2D060355 04030C26 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 33383435 35343033 3831301E 170D3236 30373136 30343033
  33385A17 0D333630 37313530 34303333 385A3031 312F302D 06035504 030C2649
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D33 38343535
  34303338 31308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201
  0A028201 010098D8 877EF1D4 C7BBF568 EA25E6A7 A13B6248 2EE0006C C4C185F3
  8E350AB7 0F574482 47D50E82 B2D7F822 B88E8685 23D25D2C 91BD75DA A5F3C69E
  8EE4C671 1C50EC50 BABEFECA 6086DCE0 EBCC7F18 14A34813 F8DA6A86 24F7AE76
  914311EC C747F45C 027F236B 2EE16587 2F786D91 7EEC3B22 8370E5D4 E0BA9673
  2AD60714 26ECA967 07F356F5 A710092A 98801D16 922BAEBA 8C8F610E DA0DBEE4
  B591151D 0B4AF2DB 5652B491 41D643F8 990E5B25 F3500BA7 5F5E91EB 57CFA023
  5018D80F B91F9BEF 32093AA4 6D5ABB11 4B3923A9 0F9C842A 2DE91EB5 3F4A4A82
  B9AB9D84 F7C0CA36 763E53CD A0C35CF5 41942EE1 C62F9200 B2FB8820 42A09E5B
  CBE8C0A0 D87D0203 010001A3 53305130 1D060355 1D0E0416 0414E101 BA663555
  CF43CCA9 664DE5E8 A9FC0E18 F670301F 0603551D 23041830 168014E1 01BA6635
  55CF43CC A9664DE5 E8A9FC0E 18F67030 0F060355 1D130101 FF040530 030101FF
  300D0609 2A864886 F70D0101 0B050003 82010100 77F381D8 A4DF93D7 525F9781
  EE19BEE8 08F52E09 11EC327A 7DF76E8B 9E527501 279D366D 7B939BA4 BFE65C54
  45A0F6DB B4516AFE C4B46FA7 78D7BD98 8F233602 5D7D0616 308CAEE1 A590DD25
  98385493 268DEAA0 EF2DD6F1 7DDB558E 50CA6211 A60AE57C AACC7C75 3508EA11
  AB47CC2B F78904BB 14759A50 463EA592 422B0F05 C0060801 8FF8A4D7 B017FFCF
  6E611917 592DA951 C027B2A1 CB58878A A56E94F8 3E3119B8 3F37229E BB303410
  16E41504 FA06453E FAE547F8 4BAA3385 8D59DD46 75C3153C F6112B14 7B6CFB06
  10034446 04CE918C 22DDE91B 69C07002 AACA9FEC 58B45915 1B582519 E61B06FF
  42480973 09693A2E 2CDA65A7 F3AD5C27 4A29ED54
        quit
!
license boot level network-essentials addon dna-essentials
memory free low-watermark processor 8237
!
diagnostic bootup level minimal
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
!
errdisable recovery cause bpduguard
errdisable recovery cause psecure-violation
errdisable recovery cause dhcp-rate-limit
errdisable recovery cause arp-inspection
!
enable secret 9 $9$aPMCPr/GnkGMNE$eLNSfxU9bMJ9mIIHECwVN6m7A/pdDe9eVXsPA6KvrlE
!
username sitstudent secret 9 $9$mVEBvsufLHsbY.$vbXs3uwT7Z3i8e1Ycim.enlbBarp.ejhRaQQ/gbW8Gs
username sitadmin secret 9 $9$I3Vx8rq4wPUCrU$Nap54lDTZY699cYEbbdNTLyPPrUwJNunEnmBL9TJzus
username bigDonkBonkAdmin privilege 15 secret 9 $9$IElovRDDH0J5RU$d7SzG5dPw0Mcyd6OZC.nZErn.a2PaxKUP0JdcLWI6Dc
username donkBonkEng1 view eng-view secret 9 $9$cYuT0Aoi5LMt1E$O24f4LVALejwfDizV1L0OTdULvy3BcLryYcmW6HiW.M
username donkBonkEng2 view eng-view secret 9 $9$UScxL6qFE0Y9YU$3x5R0PlBIltJ7HDqCluHsxrGMvmwpbAdnO2wzLoJhiw
!
redundancy
 mode sso
crypto engine compliance shield disable
!
!
!
!
!
transceiver type all
 monitoring
!
!
class-map match-any system-cpp-police-ewlc-control
  description EWLC Control
class-map match-any system-cpp-police-topology-control
  description Topology control
class-map match-any system-cpp-police-sw-forward
  description Sw forwarding, L2 LVX data packets, LOGGING, Transit Traffic
class-map match-any system-cpp-default
  description EWLC data, Inter FED Traffic
class-map match-any system-cpp-police-sys-data
  description Openflow, Exception, EGR Exception, NFL Sampled Data, RPF Failed
class-map match-any system-cpp-police-punt-webauth
  description Punt Webauth
class-map match-any system-cpp-police-l2lvx-control
  description L2 LVX control packets
class-map match-any system-cpp-police-forus
  description Forus Address resolution and Forus traffic
class-map match-any system-cpp-police-multicast-end-station
  description MCAST END STATION
class-map match-any system-cpp-police-high-rate-app
  description High Rate Applications
class-map match-any system-cpp-police-multicast
  description MCAST Data
class-map match-any system-cpp-police-l2-control
  description L2 control
class-map match-any system-cpp-police-dot1x-auth
  description DOT1X Auth
class-map match-any system-cpp-police-data
  description ICMP redirect, ICMP_GEN and BROADCAST
class-map match-any system-cpp-police-stackwise-virt-control
  description Stackwise Virtual OOB
class-map match-any non-client-nrt-class
class-map match-any system-cpp-police-routing-control
  description Routing control and Low Latency
class-map match-any system-cpp-police-protocol-snooping
  description Protocol snooping
class-map match-any system-cpp-police-dhcp-snooping
  description DHCP snooping
class-map match-any system-cpp-police-ios-routing
  description L2 control, Topology control, Routing control, Low Latency
class-map match-any system-cpp-police-system-critical
  description System Critical and Gold Pkt
class-map match-any system-cpp-police-ios-feature
  description ICMPGEN,BROADCAST,ICMP,L2LVXCntrl,ProtoSnoop,PuntWebauth,MCASTData,Transit,DOT1XAuth,Swfwd,LOGGING,L2LVXData,ForusTraffic,ForusARP,McastEndStn,Openflow,Exception,EGRExcption,NflSampled,RpfFailed
!
policy-map system-cpp-policy
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 description Trunk_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface Port-channel2
 description Trunk_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip dhcp client client-id ascii FOC27510D5Y
 ip address 172.20.10.207 255.255.254.0
 shutdown
!
interface GigabitEthernet1/0/1
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/2
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/3
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/4
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/5
 description IT_Workstations_VLAN67
 switchport access vlan 67
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/6
 description IT_Workstations_VLAN67
 switchport access vlan 67
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/7
 description IT_Workstations_VLAN67
 switchport access vlan 67
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/8
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/9
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/10
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/11
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/12
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/13
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/14
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/16
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/17
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/18
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/19
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/20
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/21
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/22
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/23
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/24
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/1/1
!
interface GigabitEthernet1/1/2
!
interface GigabitEthernet1/1/3
!
interface GigabitEthernet1/1/4
!
interface Vlan1
 no ip address
!
interface Vlan99
 description OOB_Management_SVI
 ip address 172.16.0.40 255.255.255.240
!
ip default-gateway 172.16.0.33
no ip http server
ip http authentication local
no ip http secure-server
ip http client source-interface GigabitEthernet0/0
ip forward-protocol nd
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip ssh bulk-mode 131072
ip ssh time-out 60
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
!
logging host 172.16.0.44
logging host 172.16.0.45
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 05090F08034357292A0005040E1E342B38373F3C272642
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 121B0C1030041524192E363E362712121405140E445D0A
!
!
!
control-plane
 service-policy input system-cpp-policy
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW4 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$NPSV7MCo9JK.eU$ccc9FW.tBdmrR33cjLdwGN84djwL6yTKeGc6jW90DxA
 commands interface include shutdown
 commands interface include switchport
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line vty 0 2
 access-class MGMT-ONLY in
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 060406266E4110373127190E15 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server 172.16.0.44 key 1 prefer
ntp server 172.16.0.45 key 1
!
!
!
!
!
!
end

SW4#
SW3#show run
SW3#show running-config
Building configuration...

Current configuration : 16156 bytes
!
! Last configuration change at 21:36:03 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname SW3
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
logging buffered 64000 informational

SW3#show running-config
Building configuration...

Current configuration : 16156 bytes
!
! Last configuration change at 21:36:03 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname SW3
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
logging buffered 64000 informational
no logging console
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip tacacs source-interface Vlan99
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
!
!
!
clock timezone SGT 8 0
switch 1 provision c9200l-24t-4g
!
!
!
!
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
ip dhcp snooping vlan 61
no ip dhcp snooping information option
ip dhcp snooping
ip arp inspection vlan 61
ip arp inspection validate src-mac dst-mac ip
login delay 2
login on-failure log
login on-success log
vtp version 1
!
!
!
!
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-3786926272
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-3786926272
 revocation-check none
 rsakeypair TP-self-signed-3786926272
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-3786926272
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  31312F30 2D060355 04030C26 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 33373836 39323632 3732301E 170D3236 31313036 30363135
  31315A17 0D333631 31303530 36313531 315A3031 312F302D 06035504 030C2649
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D33 37383639
  32363237 32308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201
  0A028201 0100EB7F 49BFBF0B E577A887 2C463CA5 7F88870B FC4D58D3 D7D219CE
  07406441 53A971FE 0C412AF2 03ECD930 C88614E1 CD8E00E8 4F1A51B9 50EC4315
  61CBD8E3 FB0F7C99 BC810E80 34536A57 8E581C13 FB2DDBBE 7C1D0B42 CA468F3B
  18E0688B 24583729 42B6B5F8 0B3492B9 526A1D29 3E71CD3D 0AC16CB9 99CE4E42
  02A12222 02EB3F79 FD863696 51E341D6 4B7F9B3D FA3F5696 501057BB B21E6C10
  BCB9CB0B 4B06E7DA B20706B3 99095A9A 2A452AF5 D283FB47 728EF27B 9AC2E28E
  A13558F0 E3B15DCC 2DED89A2 3768E1B5 41B539D8 3A64DE31 BC828C79 0A4FD6F1
  1D78AE9E E6E38118 9F2985A6 276E1EA4 AC3C3996 F8366209 9C13E283 34B9B18A
  318267D0 8A6B0203 010001A3 53305130 1D060355 1D0E0416 04142226 36744921
  4AF0EB58 9F40E8BB B7403405 F0CA301F 0603551D 23041830 16801422 26367449
  214AF0EB 589F40E8 BBB74034 05F0CA30 0F060355 1D130101 FF040530 030101FF
  300D0609 2A864886 F70D0101 0B050003 82010100 2B0AE18F C3ED4961 1F51A11C
  3B969CA1 C6A0BB97 691022BA EE67BDE4 F9B7E857 0484B09E 9F38420A 47860B80
  E0A6C736 6D2FC8D0 488B6789 E1BFB389 C5C22D63 14718A89 25EF0654 59510CAD
  D7FABDF9 D10C7F13 8F077A73 F4D43A42 4A72D210 7E92C8C4 E28522F5 60BFB797
  564E623D D8DC2248 1424B0E4 E46D5B1C DB3A7E1A 28DDA17F FDBB41E8 AF9AA640
  A463AAE1 BD5891D1 9413B27B 9DC3281D 33948C19 1DB6D0D4 EE13BD55 E9766452
  7CAC180B 3EE052E2 B148A11D 8DDCA959 CE54B924 7449D0D7 5CBB856F 5BFC3F35
  3295BC83 E56EA909 3F85145F BD5BB6E2 509E6D52 22B11B92 E01CA68A 46C5A815
  176F6C41 03F98895 BE42AE52 F1860FC9 E30EF867
        quit
!
license boot level network-essentials addon dna-essentials
memory free low-watermark processor 8237
!
diagnostic bootup level minimal
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
!
errdisable recovery cause bpduguard
errdisable recovery cause psecure-violation
errdisable recovery cause dhcp-rate-limit
errdisable recovery cause arp-inspection
!
enable secret 9 $9$itO.rImRIvizFk$Ivb5JqIaPvlfukfsirOZ52DLRa89fv86LrwOiKVf.Kg
!
username sitstudent secret 9 $9$PcqmenO1ddUfRk$wlHSL7DKXMRhG3g0.C9.Mf9.eZSP1guocnJrNFHjm2k
username sitadmin secret 9 $9$m/zsa1OIHWidvk$aGCTA9ziUY24npKrO688t7uPqrncyDOGtArE3KNRCmU
username bigDonkBonkAdmin privilege 15 secret 9 $9$eA0JrLVU/.RbQU$CnuMMxZEqm3G3609sQNMXMXSd5mbbiP.jRukQjOMYuc
username donkBonkEng1 view eng-view secret 9 $9$3vwlzFGpp2XZI.$8plr2IwBdZ4LIzrXAW.dmr7yeN665O.HhiopwErk752
username donkBonkEng2 view eng-view secret 9 $9$OmHscNdFchjwl.$KP.2ZyTDY1GBRVeiX1oVQozTPeQQirPdMZG4dU/t9Nw
!
redundancy
 mode sso
crypto engine compliance shield disable
!
!
!
!
!
transceiver type all
 monitoring
!
!
class-map match-any system-cpp-police-ewlc-control
  description EWLC Control
class-map match-any system-cpp-police-topology-control
  description Topology control
class-map match-any system-cpp-police-sw-forward
  description Sw forwarding, L2 LVX data packets, LOGGING, Transit Traffic
class-map match-any system-cpp-default
  description EWLC data, Inter FED Traffic
class-map match-any system-cpp-police-sys-data
  description Openflow, Exception, EGR Exception, NFL Sampled Data, RPF Failed
class-map match-any system-cpp-police-punt-webauth
  description Punt Webauth
class-map match-any system-cpp-police-l2lvx-control
  description L2 LVX control packets
class-map match-any system-cpp-police-forus
  description Forus Address resolution and Forus traffic
class-map match-any system-cpp-police-multicast-end-station
  description MCAST END STATION
class-map match-any system-cpp-police-high-rate-app
  description High Rate Applications
class-map match-any system-cpp-police-multicast
  description MCAST Data
class-map match-any system-cpp-police-l2-control
  description L2 control
class-map match-any system-cpp-police-dot1x-auth
  description DOT1X Auth
class-map match-any system-cpp-police-data
  description ICMP redirect, ICMP_GEN and BROADCAST
class-map match-any system-cpp-police-stackwise-virt-control
  description Stackwise Virtual OOB
class-map match-any non-client-nrt-class
class-map match-any system-cpp-police-routing-control
  description Routing control and Low Latency
class-map match-any system-cpp-police-protocol-snooping
  description Protocol snooping
class-map match-any system-cpp-police-dhcp-snooping
  description DHCP snooping
class-map match-any system-cpp-police-ios-routing
  description L2 control, Topology control, Routing control, Low Latency
class-map match-any system-cpp-police-system-critical
  description System Critical and Gold Pkt
class-map match-any system-cpp-police-ios-feature
  description ICMPGEN,BROADCAST,ICMP,L2LVXCntrl,ProtoSnoop,PuntWebauth,MCASTData,Transit,DOT1XAuth,Swfwd,LOGGING,L2LVXData,ForusTraffic,ForusARP,McastEndStn,Openflow,Exception,EGRExcption,NflSampled,RpfFailed
!
policy-map system-cpp-policy
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 description Trunk_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface Port-channel2
 description Trunk_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip address 172.20.10.206 255.255.254.0
 shutdown
!
interface GigabitEthernet1/0/1
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/2
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/3
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/4
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/5
 description HR_Workstations_VLAN61
 switchport access vlan 61
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/6
 description HR_Workstations_VLAN61
 switchport access vlan 61
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/7
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/8
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/9
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/10
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
!
interface GigabitEthernet1/0/11
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/12
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/13
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/14
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/16
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/17
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/18
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/19
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/20
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/21
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/22
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/23
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/24
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/1/1
!
interface GigabitEthernet1/1/2
!
interface GigabitEthernet1/1/3
!
interface GigabitEthernet1/1/4
!
interface Vlan1
 no ip address
!
interface Vlan99
 description OOB_Management_SVI
 ip address 172.16.0.39 255.255.255.240
!
ip default-gateway 172.16.0.33
no ip http server
no ip http secure-server
ip forward-protocol nd
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip ssh bulk-mode 131072
ip ssh time-out 60
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
!
logging host 172.16.0.44
logging host 172.16.0.45
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 05090F08034357292A0005040E1E342B38373F3C272642
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 1510020B262532041B3627341615260212454E5741505C
!
!
!
control-plane
 service-policy input system-cpp-policy
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW3 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$pLlBJmV0wIEsA.$N1QZKTslpicZLiyGoDYiNhNhlEGdp7s/I5e8h/XYycI
 commands interface include shutdown
 commands interface include switchport
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line vty 0 2
 access-class MGMT-ONLY in
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 082345492B161C39263B070133 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server 172.16.0.44 key 1 prefer
ntp server 172.16.0.45 key 1
!
!
!
!
!
!
end

SW3#
SW2#show run
SW2#show running-config
Building configuration...

Current configuration : 16565 bytes
!
! Last configuration change at 21:36:48 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname SW2
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
logging buffered 64000 informational
no logging console
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip tacacs source-interface Vlan99
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
!
!
!
clock timezone SGT 8 0
switch 1 provision c9200l-24t-4g
!
!
!
!
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
ip dhcp snooping vlan 46
no ip dhcp snooping information option
ip dhcp snooping
ip arp inspection vlan 46
ip arp inspection validate src-mac dst-mac ip
login delay 2
login on-failure log
login on-success log
vtp version 1
!
!
!
!
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-423246041
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-423246041
 revocation-check none
 rsakeypair TP-self-signed-423246041
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-423246041
 certificate self-signed 01
  3082032E 30820216 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  30312E30 2C060355 04030C25 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 34323332 34363034 31301E17 0D323631 31303630 36313434
  365A170D 33363131 30353036 31343436 5A303031 2E302C06 03550403 0C25494F
  532D5365 6C662D53 69676E65 642D4365 72746966 69636174 652D3432 33323436
  30343130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030 82010A02
  82010100 96048B8C 2D9CC8E6 BB05856D 2AA9FFC0 051865C8 31F32A05 EADDDC88
  2CC07CC0 08828631 3DAC6D66 A183FBA4 4B73A79E FC830B4C 1AA098D1 FE727FA5
  72BC3525 88E64CE2 9897B4F1 2A6F8945 D08A166F B5230858 80F76FF1 A9BA2CF0
  FD76D8D1 14E705D7 24433852 35FA6EA7 FBEEDF90 F5EF6347 E80C7D7F CE417E52
  1FF52E2B 9CDC447B D3951615 86BA844A D94E0CBC 8484D9AD EAF2E5E4 4BB3A0F3
  98A2A54E 2A4C52F1 2C86A0AF C682449A A0FB8800 45354D62 185EC6EB 3935DC0F
  3AC599AD B4C0793E BAA79A69 9DE9EC94 C18451DA 3753DB6A 3CC6A0AB 2511E060
  3D562242 D8329925 046471C8 B1802932 29134DE6 F5D00D32 F00F360F BD6934ED
  1EC331B3 02030100 01A35330 51301D06 03551D0E 04160414 663514AE 338392D2
  7721DB81 34C9E6AB 58034154 301F0603 551D2304 18301680 14663514 AE338392
  D27721DB 8134C9E6 AB580341 54300F06 03551D13 0101FF04 05300301 01FF300D
  06092A86 4886F70D 01010B05 00038201 0100441D F0E33D36 5577CED7 9A1001FC
  73C87900 5A18CF71 F729D6E7 ACE6E087 741C8BAD 41244CA7 7F50D38E 3C4813AB
  2EF3A3CA 31BDB7A1 0A877102 589DAF68 B8E08ABF 5715C8FC EFE60DB2 F0334E6A
  4A3F4790 5951A5CD C8EAED26 080E6EDB 83345E78 B72E78E2 5F3B6DA8 5DDD47B5
  95487B5D 44A6E276 7AD6570C B25D8874 CE7EBD2D 860D33D3 1C9715DE 872B3936
  5679C357 4BA5F778 40DBE91B 5EC00B13 C00A6265 DBF9C14B 012CABC7 A25F440E
  A7D498B9 5C70735E 0617BA24 E2DDC80C 9EACDCBF A2ACCA3C 98AFED95 4CC21095
  0B2A0EB9 FA0B5E2B A9B7AB1E 50EA106A 7790A2A6 A43F4872 97B89A39 0892EACE
  B4A33541 F16A70BC D964BE26 818CD081 1594
        quit
!
license boot level network-essentials addon dna-essentials
memory free low-watermark processor 8237
!
diagnostic bootup level minimal
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
!
errdisable recovery cause bpduguard
errdisable recovery cause psecure-violation
errdisable recovery cause dhcp-rate-limit
errdisable recovery cause arp-inspection
!
enable secret 9 $9$HLA6BRe7d/ERaE$3G4Xcm7vOiuITP.pnU7/SNsamtyxUYOFGsk/XjEEQyU
!
username sitstudent secret 9 $9$OtoW7EiPIfUVh.$zejMAgDqyY7IY.GK7d.etr0Q8riPUOugUUo.uZwLD7k
username sitadmin secret 9 $9$0pMFxuP7r78VU.$2Plmmj9KW0SWKQAKc.kAjy6/fu4zfYZEY.GfLbopeM2
username bigDonkBonkAdmin privilege 15 secret 9 $9$7/sxrwqqOZm8WE$qP54frmYaidYkB3g3Sf72Geqpfq7K9EdEIeGeRwybnI
username donkBonkEng1 view eng-view secret 9 $9$tS2g7Noo83.85E$Cuni/pb76N1JzaIkrzjtTSQKIxmuK1AAaZT7PUDibZo
username donkBonkEng2 view eng-view secret 9 $9$BsyIB9vP2i2CS.$t93eoz7XbaztiKVVvFkDvo.jZtg4ZlThrFoKZ6BlGsc
!
redundancy
 mode sso
crypto engine compliance shield disable
!
!
!
!
!
transceiver type all
 monitoring
!
!
class-map match-any system-cpp-police-ewlc-control
  description EWLC Control
class-map match-any system-cpp-police-topology-control
  description Topology control
class-map match-any system-cpp-police-sw-forward
  description Sw forwarding, L2 LVX data packets, LOGGING, Transit Traffic
class-map match-any system-cpp-default
  description EWLC data, Inter FED Traffic
class-map match-any system-cpp-police-sys-data
  description Openflow, Exception, EGR Exception, NFL Sampled Data, RPF Failed
class-map match-any system-cpp-police-punt-webauth
  description Punt Webauth
class-map match-any system-cpp-police-l2lvx-control
  description L2 LVX control packets
class-map match-any system-cpp-police-forus
  description Forus Address resolution and Forus traffic
class-map match-any system-cpp-police-multicast-end-station
  description MCAST END STATION
class-map match-any system-cpp-police-high-rate-app
  description High Rate Applications
class-map match-any system-cpp-police-multicast
  description MCAST Data
class-map match-any system-cpp-police-l2-control
  description L2 control
class-map match-any system-cpp-police-dot1x-auth
  description DOT1X Auth
class-map match-any system-cpp-police-data
  description ICMP redirect, ICMP_GEN and BROADCAST
class-map match-any system-cpp-police-stackwise-virt-control
  description Stackwise Virtual OOB
class-map match-any non-client-nrt-class
class-map match-any system-cpp-police-routing-control
  description Routing control and Low Latency
class-map match-any system-cpp-police-protocol-snooping
  description Protocol snooping
class-map match-any system-cpp-police-dhcp-snooping
  description DHCP snooping
class-map match-any system-cpp-police-ios-routing
  description L2 control, Topology control, Routing control, Low Latency
class-map match-any system-cpp-police-system-critical
  description System Critical and Gold Pkt
class-map match-any system-cpp-police-ios-feature
  description ICMPGEN,BROADCAST,ICMP,L2LVXCntrl,ProtoSnoop,PuntWebauth,MCASTData,Transit,DOT1XAuth,Swfwd,LOGGING,L2LVXData,ForusTraffic,ForusARP,McastEndStn,Openflow,Exception,EGRExcption,NflSampled,RpfFailed
!
policy-map system-cpp-policy
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 description Trunk_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface Port-channel2
 description Trunk_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip address 172.20.10.205 255.255.254.0
 shutdown
!
interface GigabitEthernet1/0/1
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/2
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/3
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/4
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/5
 description ATM_Terminals_VLAN46
 switchport access vlan 46
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/6
 description ATM_Terminals_VLAN46
 switchport access vlan 46
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/7
 description ATM_Terminals_VLAN46
 switchport access vlan 46
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/8
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/9
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/10
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/11
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/12
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/13
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/14
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/16
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/17
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/18
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/19
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/20
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/21
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/22
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/23
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/24
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/1/1
!
interface GigabitEthernet1/1/2
!
interface GigabitEthernet1/1/3
!
interface GigabitEthernet1/1/4
!
interface Vlan1
 no ip address
!
interface Vlan99
 description OOB_Management_SVI
 ip address 172.16.0.38 255.255.255.240
!
ip default-gateway 172.16.0.33
no ip http server
no ip http secure-server
ip forward-protocol nd
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip ssh bulk-mode 131072
ip ssh time-out 60
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
!
logging host 172.16.0.44
logging host 172.16.0.45
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 082345492B161C37210E1E122F3914292026351C151252
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 13071E1529031D0A18213A2530302306051016594B5C01
!
!
!
control-plane
 service-policy input system-cpp-policy
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW2 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$m/CzfQYnuAHdR.$yRpC36AwlCcK9XHZ54sr5x7mNu10MaalFM25HYfZ8AM
 commands interface include shutdown
 commands interface include switchport
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line vty 0 2
 access-class MGMT-ONLY in
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 14151B0C2E0B3305101838303B 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server 172.16.0.44 key 1 prefer
ntp server 172.16.0.45 key 1
!
!
!
!
!
!
end

SW2#
SW1#show run
SW1#show running-config
Building configuration...

Current configuration : 20324 bytes
!
! Last configuration change at 21:35:42 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname SW1
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
logging buffered 64000 informational
no logging console
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip tacacs source-interface Vlan99
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
!
!
!
clock timezone SGT 8 0
switch 1 provision c9200l-24t-4g
!
!
!
!
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
ip dhcp snooping vlan 23
no ip dhcp snooping information option
ip dhcp snooping
ip arp inspection vlan 23
ip arp inspection validate src-mac dst-mac ip
login delay 2
login on-failure log
login on-success log
vtp version 1
!
!
!
!
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-482099133
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-482099133
 revocation-check none
 rsakeypair TP-self-signed-482099133
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-482099133
 certificate self-signed 01
  3082032E 30820216 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  30312E30 2C060355 04030C25 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 34383230 39393133 33301E17 0D323631 31303630 36313431
  385A170D 33363131 30353036 31343138 5A303031 2E302C06 03550403 0C25494F
  532D5365 6C662D53 69676E65 642D4365 72746966 69636174 652D3438 32303939
  31333330 82012230 0D06092A 864886F7 0D010101 05000382 010F0030 82010A02
  82010100 E376905D 527D8C02 3E5B89A3 9E276B2B B988B232 65049EA0 50C0E8B0
  65650198 A1F15A40 55965EBD D3C0CC7F 444B23D3 8D5FA233 44C825BC 16858A37
  34B4974A E0A4B241 233065CA 179ABFD6 FC86944B 27BFC3D2 06392831 4143F755
  692E4848 97CF1682 8E80BE4B 13809AA0 D6A1245E FF568CC3 7A56560C C1E8383F
  C62C0998 54E8E964 124A13CF 59ECFE45 05E7B13E 80F345C9 3AB9B8F6 B62512DE
  C95E748C 433FDCFF A7A12ABD 155C3EEF 51209D23 E9754AE5 C3DB3187 D4FCF11D
  581FDD97 546349C0 0FD7F382 C12A03E1 E6F4A3D0 5B531C1C 7F6D8165 3F24CBEA
  4CA32B5A BE223AE3 FF55596A C086DB59 427D112E 299E3E51 A7A6ED06 D6C541F1
  7B1C51DF 02030100 01A35330 51301D06 03551D0E 04160414 DEB4042A 74A4DFCF
  75192A32 1F7B3BBA B0F20389 301F0603 551D2304 18301680 14DEB404 2A74A4DF
  CF75192A 321F7B3B BAB0F203 89300F06 03551D13 0101FF04 05300301 01FF300D
  06092A86 4886F70D 01010B05 00038201 010019BB D4A68814 42AA3866 974D7C9D
  1FD6D573 1D26DA9A 525A1D77 42BC3052 71DE9A6F EE8FAC19 35F1F2E0 6D8F7B84
  438B7866 6205F21A 816C5093 010A7030 6AD8BEF2 0286605B C4E9CD18 91F187DC
  C5154E45 49BF44BF 5EFF654B CBBDD9C5 4A30B906 89B9616D AFAAE5AE 33ED8976
  63F0AC6D A18F22C6 DD052CE1 D6505409 EB784953 F9C0BB5B CC669A81 71AAFB6E
  AEEF62A3 8FD9CE1F 05C4C798 EF996832 3BAB4FB3 85418317 81374CC1 63B73154
  06A6C025 352C5239 6BB51714 6AEA9A83 343CE69B 0F27624A 94B9F096 C31749FB
  4ED7689E 02CC6648 ECBD5168 B673FDA5 25F4B10E B187DDAD B4F0AA62 1F28ACF6
  20F273D9 9808C6D1 1515CC9E 189D6E47 836B
        quit
!
license boot level network-essentials addon dna-essentials
memory free low-watermark processor 8237
!
diagnostic bootup level minimal
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
!
!
errdisable recovery cause bpduguard
errdisable recovery cause psecure-violation
errdisable recovery cause dhcp-rate-limit
errdisable recovery cause arp-inspection
!
enable secret 9 $9$.i5msLodkxSWy.$Vt7oQJEAO8dC4iTqXbehUN3c3KC5rcvOjoj.O5Z3zyQ
!
username sitstudent secret 9 $9$LIQtPtVqBruBkE$VWHBcO7n.8aJ06/.MlL3fXpWB6efjmiB7l/v8EJHLKw
username sitadmin secret 9 $9$ykZFDu47nYo53.$GPw81fITMk5rNUff.cMj.lbLZMcJWQ.Je.q5nh6SgP2
username bigDonkBonkAdmin privilege 15 secret 9 $9$vicW4TNKt9Cfwk$Z3iYe3Bq4SB8yffPQq0cFhFwBlJsRKuacooE4D07WYU
username donkBonkEng1 view eng-view secret 9 $9$yfPE3zMypJdvEk$iDaxjuTxDyvjZkRdtvvI6DxAgXRl3UYWFTt3oKrTkBg
username donkBonkEng2 view eng-view secret 9 $9$l0nF9W9LQYTQnU$FFKvcq3ldX5.Hvk9m7UNvtXs1EdhnClzxKEvjGkUcMU
!
redundancy
 mode sso
crypto engine compliance shield disable
!
!
!
!
!
transceiver type all
 monitoring
!
!
class-map match-any system-cpp-police-ewlc-control
  description EWLC Control
class-map match-any system-cpp-police-topology-control
  description Topology control
class-map match-any system-cpp-police-sw-forward
  description Sw forwarding, L2 LVX data packets, LOGGING, Transit Traffic
class-map match-any system-cpp-default
  description EWLC data, Inter FED Traffic
class-map match-any system-cpp-police-sys-data
  description Openflow, Exception, EGR Exception, NFL Sampled Data, RPF Failed
class-map match-any system-cpp-police-punt-webauth
  description Punt Webauth
class-map match-any system-cpp-police-l2lvx-control
  description L2 LVX control packets
class-map match-any system-cpp-police-forus
  description Forus Address resolution and Forus traffic
class-map match-any system-cpp-police-multicast-end-station
  description MCAST END STATION
class-map match-any system-cpp-police-high-rate-app
  description High Rate Applications
class-map match-any system-cpp-police-multicast
  description MCAST Data
class-map match-any system-cpp-police-l2-control
  description L2 control
class-map match-any system-cpp-police-dot1x-auth
  description DOT1X Auth
class-map match-any system-cpp-police-data
  description ICMP redirect, ICMP_GEN and BROADCAST
class-map match-any system-cpp-police-stackwise-virt-control
  description Stackwise Virtual OOB
class-map match-any non-client-nrt-class
class-map match-any system-cpp-police-routing-control
  description Routing control and Low Latency
class-map match-any system-cpp-police-protocol-snooping
  description Protocol snooping
class-map match-any system-cpp-police-dhcp-snooping
  description DHCP snooping
class-map match-any system-cpp-police-ios-routing
  description L2 control, Topology control, Routing control, Low Latency
class-map match-any system-cpp-police-system-critical
  description System Critical and Gold Pkt
class-map match-any system-cpp-police-ios-feature
  description ICMPGEN,BROADCAST,ICMP,L2LVXCntrl,ProtoSnoop,PuntWebauth,MCASTData,Transit,DOT1XAuth,Swfwd,LOGGING,L2LVXData,ForusTraffic,ForusARP,McastEndStn,Openflow,Exception,EGRExcption,NflSampled,RpfFailed
!
policy-map system-cpp-policy
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 description Trunk_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface Port-channel2
 description Trunk_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 ip dhcp snooping trust
!
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip address 172.20.10.204 255.255.254.0
 shutdown
!
interface GigabitEthernet1/0/1
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/2
 description Uplink_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 1 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/3
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/4
 description Uplink_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 channel-group 2 mode active
 ip dhcp snooping trust
!
interface GigabitEthernet1/0/5
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/6
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/7
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/8
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/9
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/10
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/11
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/12
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/13
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/14
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/15
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/16
 description Staff_Workstations_VLAN23
 switchport access vlan 23
 switchport mode access
 switchport nonegotiate
 switchport port-security maximum 5
 switchport port-security violation restrict
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
 no cdp enable
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 ip verify source
 ip dhcp snooping limit rate 10
!
interface GigabitEthernet1/0/17
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/18
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/19
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/20
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/21
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/22
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/23
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/24
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/1/1
!
interface GigabitEthernet1/1/2
!
interface GigabitEthernet1/1/3
!
interface GigabitEthernet1/1/4
!
interface Vlan1
 no ip address
!
interface Vlan99
 description OOB_Management_SVI
 ip address 172.16.0.37 255.255.255.240
!
ip default-gateway 172.16.0.33
no ip http server
no ip http secure-server
ip forward-protocol nd
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip ssh bulk-mode 131072
ip ssh time-out 60
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
!
logging host 172.16.0.44
logging host 172.16.0.45
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 104C001E27180B2B3F01383D213A033431001019110507
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 1510020B262532041B3627341615260212454E5741505C
!
!
!
control-plane
 service-policy input system-cpp-policy
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS SW1 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$ZWnHFl1A/2Q5jE$zElz7PVtkeV3TnuCdtUXO5fx8ottC26QaCiMfVET5dE
 commands interface include shutdown
 commands interface include switchport
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line vty 0 2
 access-class MGMT-ONLY in
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 070D284B6C06002B232200091D 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server 172.16.0.44 key 1 prefer
ntp server 172.16.0.45 key 1
!
!
!
!
!
!
end

SW1#$                                                                     WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW2 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin
Password:

DSW2>en
Password:
DSW2#show run
Building configuration...

Current configuration : 22543 bytes
!
! Last configuration change at 21:16:34 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname DSW2
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
vrf definition OOB
 !
 address-family ipv4
 exit-address-family
!
logging buffered 64000 informational
no logging console
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip vrf forwarding OOB
 ip tacacs source-interface Vlan99
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
clock timezone SGT 8 0
switch 1 provision c9300l-24t-4g
!
!
!
!
ip routing
!
!
!
!
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
login delay 2
login on-failure log
login on-success log
vtp version 1
!
!
!
!
!
!
!
!
flow record bank-flow-record
 description packet flow record for monitoring
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match ipv4 protocol
 match ipv4 tos
 match interface input
 collect interface output
 collect counter bytes long
 collect counter packets long
 collect timestamp absolute first
 collect timestamp absolute last
!
!
flow exporter bank-flow-exporter-1
 description export flows to tacacs1 .44
 destination 172.16.0.44 vrf OOB
 source Vlan99
 transport udp 2055
 template data timeout 60
!
!
flow exporter bank-flow-exporter-2
 description export flows to tacacs2 .45
 destination 172.16.0.45 vrf OOB
 source Vlan99
 transport udp 2055
 template data timeout 60
!
!
flow monitor bank-flow-monitor
 description Redundant Dual-Collector Traffic Visibility
 exporter bank-flow-exporter-1
 exporter bank-flow-exporter-2
 cache timeout active 60
 record bank-flow-record
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-1171067940
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-1171067940
 revocation-check none
 rsakeypair TP-self-signed-1171067940
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-1171067940
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  31312F30 2D060355 04030C26 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 31313731 30363739 3430301E 170D3236 31303136 30323539
  33375A17 0D333631 30313530 32353933 375A3031 312F302D 06035504 030C2649
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D31 31373130
  36373934 30308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201
  0A028201 0100DA00 2190C5AC DE21BB60 5C774132 F8F34396 BF14FB2C 6C4B345B
  94AF705B E5E9E1B1 B1B3D4E0 0F0357A5 A04BE079 A6E9EB29 59BBEBB2 4306D3C0
  F4D1D700 89509C92 E85EDE98 89923BCB B2141447 6579229C 4982E0A2 D28069C5
  4A5DEC11 2C436211 727EE9E1 914DD5EE 0D6F074C 05F3C6A8 294BBFFB 31EDFB25
  D0027D79 7BCCEEDC 9683E7E9 8BA8587C 3833345B 55A12B44 436DE7F7 375A72F5
  9FB6949F 0560C659 258D798C A88DE386 C26F218B 5A530F8A 91953823 A499B781
  2FDFE327 FC268ACB F2F8E3B1 E841F62D 3D6097B4 048FF6C9 BDE7AB6C A93294D8
  27AD0508 42821CB8 22C0F4A8 F053B952 B636ABCC 09A531BC 2E944F3E F8B20A71
  16282B51 CC550203 010001A3 53305130 1D060355 1D0E0416 04149978 468241FB
  B216C3EE 2A987A1B 25B35305 6F83301F 0603551D 23041830 16801499 78468241
  FBB216C3 EE2A987A 1B25B353 056F8330 0F060355 1D130101 FF040530 030101FF
  300D0609 2A864886 F70D0101 0B050003 82010100 CC16A469 66523F4F 1249D480
  304AF408 ED161D1A 2273E3D6 D9FB1AD5 510A40CF 1A222D4E 5634E36D 854FC180
  0BCAFB7F 849FB610 4089EF01 132F5444 39CB2DB0 5CF94F73 E1C0E105 60213017
  4069FF8F 4ABB58D6 1F99F15D AB32175F 9E2239CA 3B1B5002 C2CB782E A355A2B8
  1CCBBA02 E93EC985 82613C31 212579DD 5B4C1D95 44839E8A 522C798E 801F61F8
  AD3527BB 3BD2969C B82D4D76 3D4D2BAA C48CB398 D8EDCAE3 648BC315 96938CF0
  DF6A8577 FDB9214E F4ADB439 4E8496A0 564F4518 3CFD7816 A1E71F73 AE700F6C
  ED36245D 0614748B 8156900D 88FB6F45 E0A3190A 3A57B57F 0F0A0534 03304F63
  322761A7 8A67C315 FA6CAD41 3C50403A 6093E925
        quit
!
!
license boot level network-advantage addon dna-advantage
memory free low-watermark processor 130180
!
diagnostic bootup level minimal
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
spanning-tree vlan 23,46,99-100 priority 28672
spanning-tree vlan 61,67 priority 24576
!
!
!
enable secret 9 $9$GExzh4/imSNILk$s3nB7wRC4S5NqyfZbtwAFJVGTT4W3IuC1NP7tPzAqEU
!
username sitstudent secret 9 $9$zf4pFjL6ptS1b.$hJosjVwTFzCZfBS8me4q3MK4hhz5L9xAzomD4WeFIKU
username sitadmin secret 9 $9$bBFADZPSjRbfFE$HaH3FwJ/aqiLixuJ9zSqukF9qVs5Cf9xEQzVM7NE3hg
username bigDonkBonkAdmin privilege 15 secret 9 $9$tlhvYp9qC.Sg4E$.OMZOgtxoU4Wd/JWGLE.5.ABPagKCRPDZocC8KQMqT.
username donkBonkEng1 view eng-view secret 9 $9$INu04/ejSSUxdU$1s3nx4ipENuz2kM0DfItGxavglFi9XUXIrXIxm3xmN2
username donkBonkEng2 view eng-view secret 9 $9$EOALwmyx/bdVpE$CjQrzfIH1yEBmNzroZSf3fBLp1jRqPTYpybHpAU60hU
!
redundancy
 mode sso
crypto engine compliance shield disable
!
!
!
!
!
transceiver type all
 monitoring
!
track 10 ip sla 10 reachability
!
!
class-map match-any system-cpp-police-ewlc-control
  description EWLC Control
class-map match-any system-cpp-police-topology-control
  description Topology control
class-map match-any system-cpp-police-sw-forward
  description Sw forwarding, L2 LVX data packets, LOGGING, Transit Traffic
class-map match-any system-cpp-default
  description EWLC Data, Inter FED Traffic
class-map match-any system-cpp-police-sys-data
  description Openflow, Exception, EGR Exception, NFL Sampled Data, RPF Failed
class-map match-any system-cpp-police-punt-webauth
  description Punt Webauth
class-map match-any system-cpp-police-l2lvx-control
  description L2 LVX control packets
class-map match-any system-cpp-police-forus
  description Forus Address resolution and Forus traffic
class-map match-any system-cpp-police-multicast-end-station
  description MCAST END STATION
class-map match-any system-cpp-police-high-rate-app
  description High Rate Applications
class-map match-any system-cpp-police-multicast
  description MCAST Data
class-map match-any system-cpp-police-l2-control
  description L2 control
class-map match-any system-cpp-police-dot1x-auth
  description DOT1X Auth
class-map match-any system-cpp-police-data
  description ICMP redirect, ICMP_GEN and BROADCAST
class-map match-any system-cpp-police-stackwise-virt-control
  description Stackwise Virtual OOB
class-map match-any non-client-nrt-class
class-map match-any system-cpp-police-routing-control
  description Routing control and Low Latency
class-map match-any system-cpp-police-protocol-snooping
  description Protocol snooping
class-map match-any system-cpp-police-dhcp-snooping
  description DHCP snooping
class-map match-any system-cpp-police-ios-routing
  description L2 control, Topology control, Routing control, Low Latency
class-map match-any system-cpp-police-system-critical
  description System Critical and Gold Pkt
class-map match-any system-cpp-police-ios-feature
  description ICMPGEN,BROADCAST,ICMP,L2LVXCntrl,ProtoSnoop,PuntWebauth,MCASTData,Transit,DOT1XAuth,Swfwd,LOGGING,L2LVXData,ForusTraffic,ForusARP,McastEndStn,Openflow,Exception,EGRExcption,NflSampled,RpfFailed
!
policy-map system-cpp-policy
!
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 description Trunk_to_SW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel2
 description Trunk_to_SW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel3
 description Trunk_to_SW3
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel4
 description Trunk_to_SW4
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel5
 description Trunk_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,46,61,67,99,100
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip dhcp client client-id ascii cisco-70da.48fb.2500-Gi0/0
 ip address 172.20.10.203 255.255.254.0
 shutdown
 negotiation auto
!
interface GigabitEthernet1/0/1
 description Uplink_to_SW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode active
!
interface GigabitEthernet1/0/2
 description Uplink_to_SW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode active
!
interface GigabitEthernet1/0/3
 description Uplink_to_SW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 2 mode active
!
interface GigabitEthernet1/0/4
 description Uplink_to_SW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 2 mode active
!
interface GigabitEthernet1/0/5
 description Uplink_to_SW3
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 3 mode active
!
interface GigabitEthernet1/0/6
 description Uplink_to_SW3
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 3 mode active
!
interface GigabitEthernet1/0/7
 description Uplink_to_SW4
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 4 mode active
!
interface GigabitEthernet1/0/8
 description Uplink_to_SW4
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 4 mode active
!
interface GigabitEthernet1/0/9
 description InterSwitch_Link_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,46,61,67,99,100
 switchport mode trunk
 switchport nonegotiate
 channel-group 5 mode active
!
interface GigabitEthernet1/0/10
 description InterSwitch_Link_to_DSW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,46,61,67,99,100
 switchport mode trunk
 switchport nonegotiate
 channel-group 5 mode active
!
interface GigabitEthernet1/0/11
 description Routed_Transit_to_ASA_E1_4
 no switchport
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.102 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 03234824151F271E1E5B4F
 ip ospf cost 10
!
interface GigabitEthernet1/0/12
 description Routed_Bypass_to_PA_eth1_7
 no switchport
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.122 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 03234824151F271E1E5B4F
 ip ospf cost 500
!
interface GigabitEthernet1/0/13
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/14
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/16
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/17
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/18
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/19
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/20
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/21
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/22
 description OOB_MGMT_DEVICES_R2_ASA_PROXMOX
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/0/23
 description OOB_MGMT_DEVICES_R2_ASA_PROXMOX
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/0/24
 description OOB_MGMT_DEVICES_R2_ASA_PROXMOX
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/1/1
!
interface GigabitEthernet1/1/2
!
interface GigabitEthernet1/1/3
!
interface GigabitEthernet1/1/4
!
interface AppGigabitEthernet1/0/1
!
interface Vlan1
 no ip address
!
interface Vlan23
 description Staff_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.3 255.255.255.224
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_STAFF_IN in
 ip ospf cost 20
 standby version 2
 standby 23 ip 172.16.0.1
 standby 23 priority 90
 standby 23 preempt
 standby 23 authentication md5 key-string 7 123E163F01191C567A7972
 standby 23 track 10 decrement 30
!
interface Vlan46
 description ATM_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.51 255.255.255.248
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_ATM_IN in
 ip ospf cost 20
 standby version 2
 standby 46 ip 172.16.0.49
 standby 46 priority 90
 standby 46 preempt
 standby 46 authentication md5 key-string 7 096B5D260A1511405B5E52
 standby 46 track 10 decrement 30
!
interface Vlan61
 description HR_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.59 255.255.255.248
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_HR_IN in
 ip ospf cost 10
 standby version 2
 standby 61 ip 172.16.0.57
 standby 61 priority 110
 standby 61 preempt
 standby 61 authentication md5 key-string 7 1535182417383B76786163
 standby 61 track 10 decrement 30
!
interface Vlan67
 description IT_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.67 255.255.255.248
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_IT_IN in
 ip ospf cost 10
 standby version 2
 standby 67 ip 172.16.0.65
 standby 67 priority 110
 standby 67 preempt
 standby 67 authentication md5 key-string 7 112E0A2D04001B5E54787D
 standby 67 track 10 decrement 30
!
interface Vlan99
 description OOB_Management_SVI
 vrf forwarding OOB
 ip address 172.16.0.36 255.255.255.240
 no ip redirects
 no ip proxy-arp
!
interface Vlan100
 description DSW1_DSW2_Transit_over_Po5
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.126 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 052C1520325C485B495741
 ip ospf cost 10
!
router ospf 1
 router-id 4.4.4.4
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface Vlan100
 no passive-interface GigabitEthernet1/0/11
 no passive-interface GigabitEthernet1/0/12
 network 172.16.0.0 0.0.0.31 area 0
 network 172.16.0.48 0.0.0.7 area 0
 network 172.16.0.56 0.0.0.7 area 0
 network 172.16.0.64 0.0.0.7 area 0
 network 172.16.0.100 0.0.0.3 area 0
 network 172.16.0.120 0.0.0.3 area 0
 network 172.16.0.124 0.0.0.3 area 0
!
ip forward-protocol nd
no ip http server
ip http authentication local
no ip http secure-server
ip http client source-interface GigabitEthernet0/0
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip route vrf OOB 0.0.0.0 0.0.0.0 172.16.0.33
ip ssh bulk-mode 131072
ip ssh time-out 60
!
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
ip access-list extended ACL_ATM_IN
 10 permit udp 172.16.0.48 0.0.0.7 host 224.0.0.102 eq 1985
 20 permit udp host 172.16.0.77 eq bootps any eq bootps
 30 permit udp any host 255.255.255.255 eq bootps
 40 permit udp 172.16.0.48 0.0.0.7 host 172.16.0.77 eq bootps
 50 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.75
 60 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.76
 70 deny ip any any log
ip access-list extended ACL_HR_IN
 10 permit udp host 172.16.0.77 eq bootps any eq bootps
 20 permit udp any host 255.255.255.255 eq bootps
 30 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.77 eq bootps
 40 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.74 eq 445
 50 permit icmp 172.16.0.56 0.0.0.7 host 172.16.0.74
 60 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain
 70 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain
 80 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq www
 90 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq 443
 110 deny ip 172.16.0.56 0.0.0.7 172.16.0.0 0.0.0.127 log
 120 permit ip 172.16.0.56 0.0.0.7 any
ip access-list extended ACL_IT_IN
 10 permit udp host 172.16.0.77 eq bootps any eq bootps
 20 permit udp any host 255.255.255.255 eq bootps
 30 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.77 eq bootps
 40 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain
 50 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain
 60 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq www
 70 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq 443
 80 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.84 eq 22
 90 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.85 eq 22
 110 deny ip 172.16.0.64 0.0.0.7 172.16.0.0 0.0.0.127 log
 120 permit ip 172.16.0.64 0.0.0.7 any
ip access-list extended ACL_STAFF_IN
 10 permit udp 172.16.0.0 0.0.0.31 host 224.0.0.102 eq 1985
 20 permit udp host 172.16.0.77 eq bootps any eq bootps
 30 permit udp any host 255.255.255.255 eq bootps
 40 permit udp 172.16.0.0 0.0.0.31 host 172.16.0.77 eq bootps
 50 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.75
 60 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.76
 70 deny ip any any log
!
ip sla 10
 icmp-echo 172.16.0.101 source-interface GigabitEthernet1/0/11
  frequency 5
ip sla schedule 10 life forever start-time now
logging host 172.16.0.44 vrf OOB
logging host 172.16.0.45 vrf OOB
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 094E470E3B0A0E323809163C2E36183226310408040750
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 05090F08034357292A0005040E1E342B38373F3C272641
!
!
!
!
control-plane
 service-policy input system-cpp-policy
!
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW2 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$OuK6r1TislrqhU$AT4XPG96Uus2/zuSc3szGyBabB5MKlrv2ZvzUpgzG5U
 commands interface include shutdown
 commands interface include switchport
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line vty 0 2
 access-class MGMT-ONLY in vrf-also
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 082345492B161C39263B070133 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server vrf OOB 172.16.0.44 key 1 prefer
ntp server vrf OOB 172.16.0.45 key 1
!
!
!
!
!
!
end

DSW2#                                                                WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW1 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin
Password:

DSW1>en
Password:
DSW1#show run
Building configuration...

Current configuration : 22486 bytes
!
! Last configuration change at 16:16:30 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service tcp-keepalives-in
service tcp-keepalives-out
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform punt-keepalive disable-kernel-core
!
hostname DSW1
!
!
vrf definition Mgmt-vrf
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family
!
vrf definition OOB
 !
 address-family ipv4
 exit-address-family
!
logging buffered 64000 informational
no logging console
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip vrf forwarding OOB
 ip tacacs source-interface Vlan99
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
clock timezone SGT 8 0
switch 1 provision c9300l-24t-4g
!
!
!
!
ip routing
!
!
!
!
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
login delay 2
login on-failure log
login on-success log
vtp version 1
!
!
!
!
!
!
!
!
flow record bank-flow-record
 description packet flow record for monitoring
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match ipv4 protocol
 match ipv4 tos
 match interface input
 collect interface output
 collect counter bytes long
 collect counter packets long
 collect timestamp absolute first
 collect timestamp absolute last
!
!
flow exporter bank-flow-exporter-1
 description export flows to tacacs1 .44
 destination 172.16.0.44 vrf OOB
 source Vlan99
 transport udp 2055
 template data timeout 60
!
!
flow exporter bank-flow-exporter-2
 description export flows to tacacs2 .45
 destination 172.16.0.45 vrf OOB
 source Vlan99
 transport udp 2055
 template data timeout 60
!
!
flow monitor bank-flow-monitor
 description Redundant Dual-Collector Traffic Visibility
 exporter bank-flow-exporter-1
 exporter bank-flow-exporter-2
 cache timeout active 60
 record bank-flow-record
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-19850445
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-19850445
 revocation-check none
 rsakeypair TP-self-signed-19850445
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-19850445
 certificate self-signed 01
  3082032C 30820214 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  2F312D30 2B060355 04030C24 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 31393835 30343435 301E170D 32363130 31363033 30303537
  5A170D33 36313031 35303330 3035375A 302F312D 302B0603 5504030C 24494F53
  2D53656C 662D5369 676E6564 2D436572 74696669 63617465 2D313938 35303434
  35308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201 0A028201
  0100B869 66B8AD3B 43BBAAF0 FDE773DC 099E2924 2FF671D7 6A3C3E22 C82FFDD9
  882EC49D 8DE8FEDC 91B9399B 41437077 96641136 DBBD1F02 42F5496D 5158FE83
  82D4D7CB 1FBAB55A 63307C5E 68AAB828 5AA739A0 930379EB DCA74245 34704FAA
  51C00AA8 53DB2460 CF0A478E 0FCDF394 F60BF6CA C18703B0 320A293E 41C89B84
  75DB7EFF ED9AE6A9 9CF455E8 AC6CBB5E B575CFCE DCABE6CE 196D2A36 8272842B
  9AC17E8F 8F67607C 751D0D44 BE1D8869 DAC846A3 A38FD6F5 AED81558 637B4632
  BAB93A2E AF406C70 A39690F7 32A2F123 1E574C49 B42A6409 897584FA D55781AE
  75401B7E 57972A7E 1FE54C14 36590EBE 6F6F4CC0 A559722F 08A863A2 87F7A319
  3A010203 010001A3 53305130 1D060355 1D0E0416 04145E59 AEB0ADEE A788FF67
  31157C9B FD93E39E BE6E301F 0603551D 23041830 1680145E 59AEB0AD EEA788FF
  6731157C 9BFD93E3 9EBE6E30 0F060355 1D130101 FF040530 030101FF 300D0609
  2A864886 F70D0101 0B050003 82010100 32088C75 49292438 84C03DBD B71AE64F
  4F15E0B5 8544ADC3 193140A5 B4C4982A F5572D39 99050C97 4A2480D5 966823A0
  1BCFB75F 78271DDD EEBD2AAC 3C5AF07C 9C0A3D4E 4567176D 270DB94E F2E5D831
  370BC815 41FF370E 6975F337 68350EF6 8C349EA8 186DC96D EAFA6860 29F77EB2
  404DB1AE 1E805184 F4A583DE 4B6C14A8 CAC333EC 1480B2C7 416EBA71 4F9C9C0E
  C277D937 688556D9 70807FEE 02D4972A 1999BF3B 740925A2 6CF697C0 A40FAB7E
  182FC539 76804E92 86EB4E98 F63985DF 45363BD6 94E0DD15 08DA17A0 70BEB895
  E3020D69 ECC5D3B5 82712E3E FE4F8F8A FA7A29EF 3AD8F5E6 AC70B822 26B5CC93
  973AC856 EA7871F9 D751E819 F5922E85
        quit
!
!
license boot level network-advantage addon dna-advantage
memory free low-watermark processor 130180
!
diagnostic bootup level minimal
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
spanning-tree vlan 23,46,99-100 priority 24576
spanning-tree vlan 61,67 priority 28672
!
!
!
enable secret 9 $9$NXa6BKRQVJsFiE$kaa516i.tF4PnJvFdaPo1GBpEanMzBwkkP5gZFE4W7E
!
username sitstudent secret 9 $9$itdWOl/ggzlSdk$16Xx3KB4bI6jZsQy.EwFHwG3R0YA3QC6Uu2E8Dfq1M.
username sitadmin secret 9 $9$pXXNvZxM9TmasE$vUBprVQGGNc9MBPR.1LWet0sg9KgfDwGwe2h0Zm2Ynk
username bigDonkBonkAdmin privilege 15 secret 9 $9$rhkqB8XbIy9kA.$GJonbz2568ovr.rjvRBRqCGtcMhDUI0uAlNvBh7dppU
username donkBonkEng1 view eng-view secret 9 $9$aDIvu/PsfFQ0HU$lIiqgXXi24O/j1KhCvctGMg.f5/Slr5nPflGzL3QKYE
username donkBonkEng2 view eng-view secret 9 $9$LpPbWhHLbv7glU$aDDUvmEIakoeS6pcMvPyKpurl55w7daGgP2de20dV7Y
!
redundancy
 mode sso
crypto engine compliance shield disable
!
!
!
!
!
transceiver type all
 monitoring
!
track 10 ip sla 10 reachability
!
!
class-map match-any system-cpp-police-ewlc-control
  description EWLC Control
class-map match-any system-cpp-police-topology-control
  description Topology control
class-map match-any system-cpp-police-sw-forward
  description Sw forwarding, L2 LVX data packets, LOGGING, Transit Traffic
class-map match-any system-cpp-default
  description EWLC Data, Inter FED Traffic
class-map match-any system-cpp-police-sys-data
  description Openflow, Exception, EGR Exception, NFL Sampled Data, RPF Failed
class-map match-any system-cpp-police-punt-webauth
  description Punt Webauth
class-map match-any system-cpp-police-l2lvx-control
  description L2 LVX control packets
class-map match-any system-cpp-police-forus
  description Forus Address resolution and Forus traffic
class-map match-any system-cpp-police-multicast-end-station
  description MCAST END STATION
class-map match-any system-cpp-police-high-rate-app
  description High Rate Applications
class-map match-any system-cpp-police-multicast
  description MCAST Data
class-map match-any system-cpp-police-l2-control
  description L2 control
class-map match-any system-cpp-police-dot1x-auth
  description DOT1X Auth
class-map match-any system-cpp-police-data
  description ICMP redirect, ICMP_GEN and BROADCAST
class-map match-any system-cpp-police-stackwise-virt-control
  description Stackwise Virtual OOB
class-map match-any non-client-nrt-class
class-map match-any system-cpp-police-routing-control
  description Routing control and Low Latency
class-map match-any system-cpp-police-protocol-snooping
  description Protocol snooping
class-map match-any system-cpp-police-dhcp-snooping
  description DHCP snooping
class-map match-any system-cpp-police-ios-routing
  description L2 control, Topology control, Routing control, Low Latency
class-map match-any system-cpp-police-system-critical
  description System Critical and Gold Pkt
class-map match-any system-cpp-police-ios-feature
  description ICMPGEN,BROADCAST,ICMP,L2LVXCntrl,ProtoSnoop,PuntWebauth,MCASTData,Transit,DOT1XAuth,Swfwd,LOGGING,L2LVXData,ForusTraffic,ForusARP,McastEndStn,Openflow,Exception,EGRExcption,NflSampled,RpfFailed
!
policy-map system-cpp-policy
!
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 description Trunk_to_SW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel2
 description Trunk_to_SW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel3
 description Trunk_to_SW3
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel4
 description Trunk_to_SW4
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 spanning-tree guard root
!
interface Port-channel5
 description Trunk_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,46,61,67,99,100
 switchport mode trunk
 switchport nonegotiate
!
interface GigabitEthernet0/0
 vrf forwarding Mgmt-vrf
 ip address 172.20.10.202 255.255.254.0
 shutdown
 negotiation auto
!
interface GigabitEthernet1/0/1
 description Uplink_to_SW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode active
!
interface GigabitEthernet1/0/2
 description Uplink_to_SW1
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode active
!
interface GigabitEthernet1/0/3
 description Uplink_to_SW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 2 mode active
!
interface GigabitEthernet1/0/4
 description Uplink_to_SW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 46,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 2 mode active
!
interface GigabitEthernet1/0/5
 description Uplink_to_SW3
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 3 mode active
!
interface GigabitEthernet1/0/6
 description Uplink_to_SW3
 switchport trunk native vlan 821
 switchport trunk allowed vlan 61,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 3 mode active
!
interface GigabitEthernet1/0/7
 description Uplink_to_SW4
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 4 mode active
!
interface GigabitEthernet1/0/8
 description Uplink_to_SW4
 switchport trunk native vlan 821
 switchport trunk allowed vlan 67,99
 switchport mode trunk
 switchport nonegotiate
 channel-group 4 mode active
!
interface GigabitEthernet1/0/9
 description InterSwitch_Link_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,46,61,67,99,100
 switchport mode trunk
 switchport nonegotiate
 channel-group 5 mode active
!
interface GigabitEthernet1/0/10
 description InterSwitch_Link_to_DSW2
 switchport trunk native vlan 821
 switchport trunk allowed vlan 23,46,61,67,99,100
 switchport mode trunk
 switchport nonegotiate
 channel-group 5 mode active
!
interface GigabitEthernet1/0/11
 description Routed_Transit_to_ASA_E1_3
 no switchport
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.98 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 08065F611A09034542595A
 ip ospf cost 10
!
interface GigabitEthernet1/0/12
 description Routed_Bypass_to_PA_eth1_6
 no switchport
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.118 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 15351823173A2D76786163
 ip ospf cost 500
!
interface GigabitEthernet1/0/13
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/14
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/15
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/16
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/17
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/18
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/19
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/20
 description UNUSED_PORTS
 switchport access vlan 727
 switchport mode access
 switchport nonegotiate
 shutdown
!
interface GigabitEthernet1/0/21
 description OOB_Mgmt_Workstation
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/0/22
 description OOB_MGMT_DEVICES_R1_PA_PROXMOX
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/0/23
 description OOB_MGMT_DEVICES_R1_PA_PROXMOX
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/0/24
 description OOB_MGMT_DEVICES_R1_PA_PROXMOX
 switchport access vlan 99
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/1/1
!
interface GigabitEthernet1/1/2
!
interface GigabitEthernet1/1/3
!
interface GigabitEthernet1/1/4
!
interface AppGigabitEthernet1/0/1
!
interface Vlan1
 no ip address
!
interface Vlan23
 description Staff_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.2 255.255.255.224
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_STAFF_IN in
 ip ospf cost 10
 standby version 2
 standby 23 ip 172.16.0.1
 standby 23 priority 110
 standby 23 preempt
 standby 23 authentication md5 key-string 7 0023002E17491B545F731A
 standby 23 track 10 decrement 30
!
interface Vlan46
 description ATM_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.50 255.255.255.248
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_ATM_IN in
 ip ospf cost 10
 standby version 2
 standby 46 ip 172.16.0.49
 standby 46 priority 110
 standby 46 preempt
 standby 46 authentication md5 key-string 7 072832645D1B095747405D
 standby 46 track 10 decrement 30
!
interface Vlan61
 description HR_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.58 255.255.255.248
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_HR_IN in
 ip ospf cost 20
 standby version 2
 standby 61 ip 172.16.0.57
 standby 61 priority 90
 standby 61 preempt
 standby 61 authentication md5 key-string 7 1322043A181E14787B767E
 standby 61 track 10 decrement 30
!
interface Vlan67
 description IT_VLAN_Gateway
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.66 255.255.255.248
 ip helper-address 172.16.0.77
 no ip redirects
 no ip proxy-arp
 ip access-group ACL_IT_IN in
 ip ospf cost 20
 standby version 2
 standby 67 ip 172.16.0.65
 standby 67 priority 90
 standby 67 preempt
 standby 67 authentication md5 key-string 7 06211C095F5C194B554544
 standby 67 track 10 decrement 30
!
interface Vlan99
 description OOB_Management_SVI
 vrf forwarding OOB
 ip address 172.16.0.35 255.255.255.240
 no ip redirects
 no ip proxy-arp
!
interface Vlan100
 description DSW1_DSW2_Transit_over_Po5
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.125 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 0134152B481B005D711E18
 ip ospf cost 10
!
router ospf 1
 router-id 3.3.3.3
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface Vlan100
 no passive-interface GigabitEthernet1/0/11
 no passive-interface GigabitEthernet1/0/12
 network 172.16.0.0 0.0.0.31 area 0
 network 172.16.0.48 0.0.0.7 area 0
 network 172.16.0.56 0.0.0.7 area 0
 network 172.16.0.64 0.0.0.7 area 0
 network 172.16.0.96 0.0.0.3 area 0
 network 172.16.0.116 0.0.0.3 area 0
 network 172.16.0.124 0.0.0.3 area 0
!
ip forward-protocol nd
no ip http server
ip http authentication local
no ip http secure-server
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip route vrf OOB 0.0.0.0 0.0.0.0 172.16.0.33
ip ssh bulk-mode 131072
ip ssh time-out 60
!
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
ip access-list extended ACL_ATM_IN
 10 permit udp 172.16.0.48 0.0.0.7 host 224.0.0.102 eq 1985
 20 permit udp host 172.16.0.77 eq bootps any eq bootps
 30 permit udp any host 255.255.255.255 eq bootps
 40 permit udp 172.16.0.48 0.0.0.7 host 172.16.0.77 eq bootps
 50 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.75
 60 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.76
 70 deny ip any any log
ip access-list extended ACL_HR_IN
 10 permit udp host 172.16.0.77 eq bootps any eq bootps
 20 permit udp any host 255.255.255.255 eq bootps
 30 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.77 eq bootps
 40 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.74 eq 445
 50 permit icmp 172.16.0.56 0.0.0.7 host 172.16.0.74
 60 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain
 70 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain
 80 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq www
 90 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq 443
 110 deny ip 172.16.0.56 0.0.0.7 172.16.0.0 0.0.0.127 log
 120 permit ip 172.16.0.56 0.0.0.7 any
ip access-list extended ACL_IT_IN
 10 permit udp host 172.16.0.77 eq bootps any eq bootps
 20 permit udp any host 255.255.255.255 eq bootps
 30 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.77 eq bootps
 40 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain
 50 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain
 60 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq www
 70 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq 443
 80 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.84 eq 22
 90 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.85 eq 22
 110 deny ip 172.16.0.64 0.0.0.7 172.16.0.0 0.0.0.127 log
 120 permit ip 172.16.0.64 0.0.0.7 any
ip access-list extended ACL_STAFF_IN
 10 permit udp 172.16.0.0 0.0.0.31 host 224.0.0.102 eq 1985
 20 permit udp host 172.16.0.77 eq bootps any eq bootps
 30 permit udp any host 255.255.255.255 eq bootps
 40 permit udp 172.16.0.0 0.0.0.31 host 172.16.0.77 eq bootps
 50 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.75
 60 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.76
 70 deny ip any any log
!
ip sla 10
 icmp-echo 172.16.0.97 source-interface GigabitEthernet1/0/11
  frequency 5
ip sla schedule 10 life forever start-time now
logging host 172.16.0.44 vrf OOB
logging host 172.16.0.45 vrf OOB
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 00061A01265412263C245E580C0B351601181B0B382F75
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 060406266E4110393612001D09161A2A373B243A301755
!
!
!
!
control-plane
 service-policy input system-cpp-policy
!
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW1 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$W9O2/ZCaLFjI7E$D9vClxp7ASPBax83uVO2f4qTeDcqgj2O.qmjIYf7r9M
 commands interface include shutdown
 commands interface include switchport
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line vty 0 2
 access-class MGMT-ONLY in vrf-also
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 00061A01265412283B11474B10 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server vrf OOB 172.16.0.44 key 1 prefer
ntp server vrf OOB 172.16.0.45 key 1
!
!
!
!
!
!
end

DSW1#$                                                                       WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R2 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin
Password:

R2>en
Password:
R2#show run
Building configuration...

Current configuration : 12556 bytes
!
! Last configuration change at 17:14:19 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service tcp-keepalives-in
service tcp-keepalives-out
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform qfp utilization monitor load 80
platform punt-keepalive disable-kernel-core
!
hostname R2
!
boot-start-marker
boot-end-marker
!
!
vrf definition Mgmt-vrf
 rd 1:1
 !
 address-family ipv4
  route-target export 1:1
  route-target import 1:1
 exit-address-family
!
vrf definition OOB
 !
 address-family ipv4
 exit-address-family
!
logging buffered 64000 informational
no logging console
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip vrf forwarding OOB
 ip tacacs source-interface GigabitEthernet0/1/0
!
aaa authentication login default group tac-group local
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
clock timezone SGT 8 0
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
!
!
!
!
!
!
login delay 2
login on-failure log
login on-success log
!
!
!
!
!
!
!
subscriber templating
vtp version 1
multilink bundle-name authenticated
!
flow record bank-flow-record
 description packet flow record for monitoring
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match ipv4 protocol
 match ipv4 tos
 match interface input
 collect interface output
 collect counter bytes long
 collect counter packets long
 collect timestamp absolute first
 collect timestamp absolute last
!
!
flow exporter bank-flow-exporter-1
 description export flows to tacacs1 .44
 destination 172.16.0.44 vrf OOB
 source GigabitEthernet0/1/0
 transport udp 2055
 template data timeout 60
!
!
flow exporter bank-flow-exporter-2
 description export flows to tacacs2 .45
 destination 172.16.0.45 vrf OOB
 source GigabitEthernet0/1/0
 transport udp 2055
 template data timeout 60
!
!
flow monitor bank-flow-monitor
 description Redundant Dual-Collector Traffic Visibility
 exporter bank-flow-exporter-1
 exporter bank-flow-exporter-2
 cache timeout active 60
 record bank-flow-record
!
!
pae
!
!
crypto pki trustpoint TP-self-signed-3634272404
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-3634272404
 revocation-check none
 rsakeypair TP-self-signed-3634272404
 hash sha256
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
!
crypto pki certificate chain TP-self-signed-3634272404
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  31312F30 2D060355 04030C26 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 33363334 32373234 3034301E 170D3236 30373136 30333330
  35375A17 0D333630 37313530 33333035 375A3031 312F302D 06035504 030C2649
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D33 36333432
  37323430 34308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201
  0A028201 0100EDE2 66AC2573 C6C24181 300C37D9 4CDB1955 008C3B0E 824D570A
  A14340C6 FBB8D33E E20D65CC 35CCA41B 9822D13C CB9D0937 21AE2506 484A6375
  CAFDD948 979A2165 E7EECAEA B6EB78FF 4C5998CE D957C359 13B3E28A 2776D756
  E68C6AB2 708540EE 61170EB7 862A8A25 44411F1B 16B81062 3DE40A83 4D503BB4
  5D5FE1CE 371AF38E E9C3A81D 5DF02F74 6108E83C 431A6A97 975DABC1 661CE104
  4029BBC9 CEB94805 3F7ED479 BDAB34E6 F012E120 294F1D6C 1903181A 1171A401
  F1DAED4D 1E4739BC 2286A02E 8A83CAC2 535B91CE F26B381B 384BA406 1FCD209D
  C8539D0E 1119D330 79CEFFDC 9D57230C 452514E2 5105AD92 143319D7 63A93D88
  9B8160CF 7DD90203 010001A3 53305130 1D060355 1D0E0416 0414723E 88A7AE6F
  1E14BC52 79DD8CE6 FC4E2913 6C44301F 0603551D 23041830 16801472 3E88A7AE
  6F1E14BC 5279DD8C E6FC4E29 136C4430 0F060355 1D130101 FF040530 030101FF
  300D0609 2A864886 F70D0101 0B050003 82010100 A6ABA9C2 02E52C7E 7534C326
  5B64DF4F F56265BE 6B17C9F7 DE9B03BF 1BDA7AEF 44FC84B4 F27B9F54 6E451F14
  8F34E234 2BC4489E B98664C6 EABA7D11 3E4AB00F B190D706 4C4839BC 98362258
  B003C9CA 7D55C80A 4F8215CA C909E0AE 4299982F 305B5FFA 4492004D CC037425
  CE6A08A9 38E18A92 740353D0 F7B4E57E DA7E2631 137BA4A7 DCAFE332 F229BB9C
  3AAAE321 5139BCCE 2571F176 93266389 8EC743C4 55134FC3 D647B4ED AD570A4F
  253FF75B D0B62CCF BDCBC224 B709505D 8DB4E04A 4047E7C7 BF0130DD B6E807B2
  B3ED86B1 6CA8FEB9 CA37E761 D94349E8 B09BC051 17372FD7 A9EBF6BF D7490F9E
  A7F31E6C 5CABAFC5 DC21D976 7A04E289 058B4069
        quit
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
!
!
diagnostic bootup level minimal
!
license udi pid C8200L-1N-4T sn FGL2803L1LW
memory free low-watermark processor 61453
!
spanning-tree extend system-id
!
!
enable secret 9 $9$1X35PoxBTgt8sk$xZ56a3DDp5Rf6/FxRoSSJEKWlIyw.mzJz/fBEi07.H2
!
username sitstudent secret 9 $9$cgiRaZ0wKiAIt.$nT5uu.c96AFFmFEyYEFWNtQBWO0p1wQM9s59.IX/lzY
username sitadmin secret 9 $9$MwzS1uEPwNtB5U$Bek6yHDXhs2fEk4afqwwdehbRtuqs9pmOqTzweL4tEg
username bigDonkBonkAdmin privilege 15 secret 9 $9$L076vI5XzZ77ok$mvleISwFPBtbHI99OjunSYxK9gjfbDHuJMKg/XkM2lE
username donkBonkEng1 view eng-view secret 9 $9$zpLVvYIRxfVNbU$9S8Yh3U12GSOF/Gz32qL4tNfBgBJ0yl1BaXDy/cgkI.
username donkBonkEng2 view eng-view secret 9 $9$aO.YhfIc6Paw4.$1Zm5lXgm86XcoIhBRxv/SWoAmEPaIPmsO.XBJJhIArE
!
redundancy
 mode none
!
!
!
!
track 1 ip sla 1 reachability
!
!
!
!
!
interface Null0
 no ip unreachables
!
interface GigabitEthernet0/0/0
 description WAN_Link_to_ISP2
 ip flow monitor bank-flow-monitor input
 ip address 172.17.10.93 255.255.255.252
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 ip verify unicast source reachable-via rx allow-default
 ip nat outside
 ip access-group WAN-IN in
 negotiation auto
 no cdp enable
!
interface GigabitEthernet0/0/1
 description Secondary_Transit_to_PA440_eth1_2
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.91 255.255.255.248
 no ip redirects
 no ip proxy-arp
 ip nat inside
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 096B5D260A1511405B5E52
 ip ospf cost 20
 negotiation auto
!
interface GigabitEthernet0/0/2
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/3
 no ip address
 negotiation auto
!
interface GigabitEthernet0/1/0
 description OOB_Management_Interface_VLAN99
 vrf forwarding OOB
 ip address 172.16.0.34 255.255.255.240
 no ip redirects
 no ip proxy-arp
 negotiation auto
!
interface GigabitEthernet0/1/1
 description PA_Bypass_Link_to_ASA_E1_2
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.113 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip nat inside
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 112E0A2A04020D5E54787D
 ip ospf cost 500
 negotiation auto
!
router ospf 1
 router-id 2.2.2.2
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface GigabitEthernet0/0/1
 no passive-interface GigabitEthernet0/1/1
 network 172.16.0.88 0.0.0.7 area 0
 network 172.16.0.112 0.0.0.3 area 0
 default-information originate metric 50 metric-type 1
!
ip forward-protocol nd
no ip http server
ip http authentication local
no ip http secure-server
ip http client source-interface GigabitEthernet0/1/1
!
ip nat pool PUB-PAT 129.126.142.185 129.126.142.185 prefix-length 29
ip nat inside source static tcp 172.16.0.84 22 129.126.142.185 2222 extendable
ip nat inside source static 172.16.0.82 129.126.142.189
ip nat inside source static tcp 172.16.0.83 53 129.126.142.190 53 extendable
ip nat inside source static udp 172.16.0.83 53 129.126.142.190 53 extendable
ip nat inside source list NAT-SOURCES pool PUB-PAT overload
ip route 0.0.0.0 0.0.0.0 172.17.10.94 track 1
ip route 129.126.142.184 255.255.255.248 Null0
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip ssh bulk-mode 131072
ip ssh time-out 60
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
ip access-list extended NAT-SOURCES
 10 permit ip 172.16.0.56 0.0.0.7 any
 20 permit ip 172.16.0.64 0.0.0.7 any
 30 permit ip host 172.16.0.76 any
 40 permit ip host 172.16.0.83 any
 50 permit ip host 172.16.0.84 any
 60 permit ip host 172.16.0.85 any
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
 10 deny tcp any host 172.17.10.93 eq 22 log
 20 deny tcp any host 172.17.10.93 eq telnet log
 30 permit icmp host 172.17.10.94 host 172.17.10.93 echo-reply
 40 permit ip any 129.126.142.184 0.0.0.7
 50 permit tcp any host 172.17.10.93 established
 60 deny ip any any log
!
ip sla 1
 icmp-echo 172.17.10.94 source-interface GigabitEthernet0/0/0
  frequency 5
ip sla schedule 1 life forever start-time now
logging host 172.16.0.44 vrf OOB
logging host 172.16.0.45 vrf OOB
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 13071E1529031D0A18213A2530302306051016594B5C02
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 01110F0379041F2F12495C1F1C172713181F132539207A
!
!
!
!
control-plane
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R2 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$GiBP2FrNJBHQAU$XHW7kSx9XgBCMGSqD.EzVpy4uca5SS/KQXb43AE5.bY
 commands interface include shutdown
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line aux 0
line vty 0 2
 access-class MGMT-ONLY in vrf-also
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 045902012D2E55603D290E120B 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server vrf OOB 172.16.0.44 key 1 prefer
ntp server vrf OOB 172.16.0.45 key 1
!
!
!
!
!
!
end

R2#$                                                                       WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R1 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin
Password:

R1>en
Password:
Nov  9 21:47:00.211: %SEC_LOGIN-5-LOGIN_SUCCESS: Login Success [user: bigDonkBonkAdmin] [Source: LOCAL] [localport: 0] at 21:47:00 SGT Mon Nov 9 2026
R1#show run
Building configuration...

Current configuration : 12516 bytes
!
! Last configuration change at 21:33:06 SGT Mon Nov 9 2026 by bigDonkBonkAdmin
!
version 17.12
service tcp-keepalives-in
service tcp-keepalives-out
service timestamps debug datetime msec
service timestamps log datetime msec localtime
service password-encryption
platform qfp utilization monitor load 80
platform punt-keepalive disable-kernel-core
!
hostname R1
!
boot-start-marker
boot-end-marker
!
!
vrf definition Mgmt-vrf
 rd 1:1
 !
 address-family ipv4
  route-target export 1:1
  route-target import 1:1
 exit-address-family
!
vrf definition OOB
 !
 address-family ipv4
 exit-address-family
!
logging buffered 64000 informational
logging console informational
logging monitor informational
aaa new-model
!
!
aaa group server tacacs+ tac-group
 server name TACACS1
 server name TACACS2
 ip vrf forwarding OOB
 ip tacacs source-interface GigabitEthernet0/1/0
!
aaa authentication login default group tac-group local
 --More--
Nov  9 21:47:03.228: %FMANFP-6-IPACCESSLOGDP: F0/0: fman_fp_image: list WAN-IN denied icmp f445.8884.2772 172.17.9.94 -> 172.17.9.94 (2048/0), 216 packets
aaa authentication login CONSOLE group tac-group local
aaa authorization config-commands
aaa authorization exec default group tac-group local
aaa authorization commands 15 default group tac-group local
aaa accounting exec default start-stop group tac-group
aaa accounting commands 15 default stop-only group tac-group
!
!
aaa session-id common
clock timezone SGT 8 0
!
no ip domain lookup
ip domain name gruppesechs.sitict.net
!
!
!
!
!
!
!
!
!
login delay 2
login on-failure log
login on-success log
!
!
!
!
!
!
!
subscriber templating
vtp version 1
multilink bundle-name authenticated
!
flow record bank-flow-record
 description packet flow record for monitoring
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match ipv4 protocol
 match ipv4 tos
 match interface input
 collect interface output
 collect counter bytes long
 collect counter packets long
 collect timestamp absolute first
 collect timestamp absolute last
!
!
flow exporter bank-flow-exporter-1
 description export flows to tacacs1 .44
 destination 172.16.0.44 vrf OOB
 source GigabitEthernet0/1/0
 transport udp 2055
 template data timeout 60
!
!
flow exporter bank-flow-exporter-2
 description export flows to tacacs2 .45
 destination 172.16.0.45 vrf OOB
 source GigabitEthernet0/1/0
 transport udp 2055
 template data timeout 60
!
!
flow monitor bank-flow-monitor
 description Redundant Dual-Collector Traffic Visibility
 exporter bank-flow-exporter-1
 exporter bank-flow-exporter-2
 cache timeout active 60
 record bank-flow-record
!
!
pae
!
!
crypto pki trustpoint SLA-TrustPoint
 enrollment pkcs12
 revocation-check crl
 hash sha256
!
crypto pki trustpoint TP-self-signed-4263099988
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-4263099988
 revocation-check none
 rsakeypair TP-self-signed-4263099988
 hash sha256
!
!
crypto pki certificate chain SLA-TrustPoint
 certificate ca 01
  30820321 30820209 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  32310E30 0C060355 040A1305 43697363 6F312030 1E060355 04031317 43697363
  6F204C69 63656E73 696E6720 526F6F74 20434130 1E170D31 33303533 30313934
  3834375A 170D3338 30353330 31393438 34375A30 32310E30 0C060355 040A1305
  43697363 6F312030 1E060355 04031317 43697363 6F204C69 63656E73 696E6720
  526F6F74 20434130 82012230 0D06092A 864886F7 0D010101 05000382 010F0030
  82010A02 82010100 A6BCBD96 131E05F7 145EA72C 2CD686E6 17222EA1 F1EFF64D
  CBB4C798 212AA147 C655D8D7 9471380D 8711441E 1AAF071A 9CAE6388 8A38E520
  1C394D78 462EF239 C659F715 B98C0A59 5BBB5CBD 0CFEBEA3 700A8BF7 D8F256EE
  4AA4E80D DB6FD1C9 60B1FD18 FFC69C96 6FA68957 A2617DE7 104FDC5F EA2956AC
  7390A3EB 2B5436AD C847A2C5 DAB553EB 69A9A535 58E9F3E3 C0BD23CF 58BD7188
  68E69491 20F320E7 948E71D7 AE3BCC84 F10684C7 4BC8E00F 539BA42B 42C68BB7
  C7479096 B4CB2D62 EA2F505D C7B062A4 6811D95B E8250FC4 5D5D5FB8 8F27D191
  C55F0D76 61F9A4CD 3D992327 A8BB03BD 4E6D7069 7CBADF8B DF5F4368 95135E44
  DFC7C6CF 04DD7FD1 02030100 01A34230 40300E06 03551D0F 0101FF04 04030201
  06300F06 03551D13 0101FF04 05300301 01FF301D 0603551D 0E041604 1449DC85
  4B3D31E5 1B3E6A17 606AF333 3D3B4C73 E8300D06 092A8648 86F70D01 010B0500
  03820101 00507F24 D3932A66 86025D9F E838AE5C 6D4DF6B0 49631C78 240DA905
  604EDCDE FF4FED2B 77FC460E CD636FDB DD44681E 3A5673AB 9093D3B1 6C9E3D8B
  D98987BF E40CBD9E 1AECA0C2 2189BB5C 8FA85686 CD98B646 5575B146 8DFC66A8
  467A3DF4 4D565700 6ADF0F0D CF835015 3C04FF7C 21E878AC 11BA9CD2 55A9232C
  7CA7B7E6 C1AF74F6 152E99B7 B1FCF9BB E973DE7F 5BDDEB86 C71E3B49 1765308B
  5FB0DA06 B92AFE7F 494E8A9E 07B85737 F3A58BE1 1A48A229 C37C1E69 39F08678
  80DDCD16 D6BACECA EEBC7CF9 8428787B 35202CDC 60E4616A B623CDBD 230E3AFB
  418616A9 4093E049 4D10AB75 27E86F73 932E35B5 8862FDAE 0275156F 719BB2F0
  D697DF7F 28
        quit
crypto pki certificate chain TP-self-signed-4263099988
 certificate self-signed 01
  30820330 30820218 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030
  31312F30 2D060355 04030C26 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 34323633 30393939 3838301E 170D3236 31313036 30363131
  35395A17 0D333631 31303530 36313135 395A3031 312F302D 06035504 030C2649
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D34 32363330
  39393938 38308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201
  0A028201 01009B2D 70820D47 6B7EB8C9 D7F08322 3950D0E6 241B303F F7818611
  B4D3290C BAB00647 8C05EF57 36ACC294 26696108 0933335D 51A053C9 C7A65252
  CADF31D6 C30590F0 C6DEAAD1 E2CCF0EA 9BFF9D24 53D62DF7 50B44453 DAE6F81E
  F0FEDAAB 1CDD291A CC01FB05 DEE21E31 4EBB4689 E26F4826 735DB467 72B0531F
  DD5DB529 47FC0ED3 E6F6072C 4BC322F4 38FEBDB2 BC43784E 5C724995 41C0C8FE
  3B84F936 27C9E621 E71801BE 770E21B6 502E6F16 36686562 F0153532 EC11E1D1
  552B2519 3CB194F2 4E2EC75F DDA860C0 A35449EA CB8940EF E69AD5AE 5744F65B
  A47C2F07 B4C3DACD 433B19D1 A5C77CFC 40DA561F 500309BA 7DAC5A1E 5CBE86D7
  B553407F D2B50203 010001A3 53305130 1D060355 1D0E0416 04148B27 2D330314
  068514A1 37B5D8FA 29E7BDA9 4258301F 0603551D 23041830 1680148B 272D3303
  14068514 A137B5D8 FA29E7BD A9425830 0F060355 1D130101 FF040530 030101FF
  300D0609 2A864886 F70D0101 0B050003 82010100 020B1DE1 7B0BC624 8BE425AE
  E54698FA 970EFE83 7DC05040 483A9067 DD42C2D8 B0BA757E 8D2010DC 368BC006
  6B368509 D532EE53 5A219666 E2E230E1 95195D77 7B19E0F7 3C6FF532 350F3C39
  CD86BAFF A06D49FA 26FA833D 9633C725 D1C7AFE2 432E280A 572E4CF1 8677515D
  3D59B714 CBB2116D 16D2385B 108E2FAE 5E503AA5 DC5AB03E 3876F837 D013FB6A
  DDD3EC87 C5F9516A 476C8809 100CED65 0C4AE3F7 11377521 855B0D59 4826A4B9
  0A7A031C 9913604E CF92A523 0FD35B56 0E8587BE 4EF85215 ACB8125D F0E78885
  D98C1EE4 EF2B9698 6241052B 5227387C 62C470A5 0C1EBA1A 74037934 769A3CA1
  5475C251 054D957B 62EC9AE4 7A92FCF7 93650F8B
        quit
!
!
diagnostic bootup level minimal
!
license udi pid C8200L-1N-4T sn FGL2803L1L4
memory free low-watermark processor 61453
!
spanning-tree extend system-id
!
!
enable secret 9 $9$3zh.ojLFZMZRtU$LAWSJk6aG.47ZhfuYbS6bgBAs.VcyHY44Jl/Nxzhp0w
!
username sitstudent secret 9 $9$tsQR6TpaVC7s5U$6el8g9/3KuQy7K5IWWWTZDIsHb5G/IVch1QuzDCLV1M
username sitadmin secret 9 $9$4k9PaKIZWKXEr.$0cL5lKDmWrG1nZVovVi6MDQMu6BwgWAgqXvOXvXxYmc
username bigDonkBonkAdmin privilege 15 secret 9 $9$02nImIdIQmR8Ok$4TfR7UQuFrxR.8wO7AjMbDeZmdQ68.LZe25pee4TzDM
username donkBonkEng1 view eng-view secret 9 $9$qmdw50u3UE873k$bBRFjMo2zcb4CHcE.qS3j5b.DCRxHkJLNSQekXAgKKM
username donkBonkEng2 view eng-view secret 9 $9$0f4I/cdanO8iWk$4jFdjWgwRn3wPcUQC3AZKnIA9PTl89VTckWJnpO03Ek
!
redundancy
 mode none
!
!
!
!
track 1 ip sla 1 reachability
!
!
!
!
!
interface Null0
 no ip unreachables
!
interface GigabitEthernet0/0/0
 description WAN_Link_to_ISP1
 ip flow monitor bank-flow-monitor input
 ip address 172.17.9.93 255.255.255.252
 no ip redirects
 no ip unreachables
 no ip proxy-arp
 ip verify unicast source reachable-via rx allow-default
 ip nat outside
 ip access-group WAN-IN in
 negotiation auto
 no cdp enable
!
interface GigabitEthernet0/0/1
 description Primary_Transit_to_PA440_eth1_5
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.90 255.255.255.248
 no ip redirects
 no ip proxy-arp
 ip nat inside
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 047C18291C314A1C594B53
 ip ospf cost 10
 negotiation auto
!
interface GigabitEthernet0/0/2
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/3
 no ip address
 negotiation auto
!
interface GigabitEthernet0/1/0
 description OOB_Management_Gateway_VLAN99
 vrf forwarding OOB
 ip address 172.16.0.33 255.255.255.240
 no ip redirects
 no ip proxy-arp
 negotiation auto
!
interface GigabitEthernet0/1/1
 description PA_Bypass_Link_to_ASA_E1_5
 ip flow monitor bank-flow-monitor input
 ip address 172.16.0.109 255.255.255.252
 no ip redirects
 no ip proxy-arp
 ip nat inside
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 7 1322043D181C02787B767E
 ip ospf cost 500
 negotiation auto
!
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 10000
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
 no passive-interface GigabitEthernet0/0/1
 no passive-interface GigabitEthernet0/1/1
 network 172.16.0.88 0.0.0.7 area 0
 network 172.16.0.108 0.0.0.3 area 0
 default-information originate metric 10 metric-type 1
!
ip forward-protocol nd
no ip http server
no ip http secure-server
!
ip nat pool PUB-PAT 203.149.210.185 203.149.210.185 prefix-length 29
ip nat inside source static tcp 172.16.0.84 22 203.149.210.185 2222 extendable
ip nat inside source static 172.16.0.82 203.149.210.189
ip nat inside source static tcp 172.16.0.83 53 203.149.210.190 53 extendable
ip nat inside source static udp 172.16.0.83 53 203.149.210.190 53 extendable
ip nat inside source list NAT-SOURCES pool PUB-PAT overload
ip route 0.0.0.0 0.0.0.0 172.17.9.94 track 1
ip route 203.149.210.184 255.255.255.248 Null0
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO_NOT_DELETE_THIS_ROUTE
ip ssh bulk-mode 131072
ip ssh time-out 60
!
ip access-list standard MGMT-ONLY
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any log
ip access-list standard NTP-SERVERS
 10 permit 172.16.0.44
 20 permit 172.16.0.45
 30 deny   any
!
ip access-list extended NAT-SOURCES
 10 permit ip 172.16.0.56 0.0.0.7 any
 20 permit ip 172.16.0.64 0.0.0.7 any
 30 permit ip host 172.16.0.76 any
 40 permit ip host 172.16.0.83 any
 50 permit ip host 172.16.0.84 any
 60 permit ip host 172.16.0.85 any
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
 10 deny tcp any host 172.17.9.93 eq 22 log
 20 deny tcp any host 172.17.9.93 eq telnet log
 30 permit icmp host 172.17.9.94 host 172.17.9.93 echo-reply
 50 permit ip any 203.149.210.184 0.0.0.7
 60 permit tcp any host 172.17.9.93 established
 70 deny ip any any log
!
ip sla 1
 icmp-echo 172.17.9.94 source-interface GigabitEthernet0/0/0
  frequency 5
ip sla schedule 1 life forever start-time now
logging host 172.16.0.44 vrf OOB
logging host 172.16.0.45 vrf OOB
!
tacacs server TACACS1
 address ipv4 172.16.0.44
 key 7 05090F08034357292A0005040E1E342B38373F3C272642
tacacs server TACACS2
 address ipv4 172.16.0.45
 key 7 05090F08034357292A0005040E1E342B38373F3C272641
!
!
!
!
control-plane
!
banner motd ^CWARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R1 IS PROHIBITED.^C
parser view eng-view
 secret 9 $9$PtP8grcjTCuciU$AHCLulMa0H0DMKuYE/9U5f8nAn8.ZmXFityg6zQOa66
 commands interface include shutdown
 commands configure include interface
 commands exec include traceroute
 commands exec include ping
 commands exec include configure terminal
 commands exec include configure
 commands exec include all show
 commands exec include clear
!
!
line con 0
 logging synchronous
 login authentication CONSOLE
 stopbits 1
line aux 0
line vty 0 2
 access-class MGMT-ONLY in vrf-also
 logging synchronous
 transport input ssh
line vty 3 4
 transport input none
line vty 5 31
 transport input none
!
ntp logging
ntp authentication-key 1 md5 060406266E4110373127190E15 7
ntp authenticate
ntp trusted-key 1
ntp access-group peer NTP-SERVERS
ntp server vrf OOB 172.16.0.44 key 1 prefer
ntp server vrf OOB 172.16.0.45 key 1
!
!
!
!
!
!
end

R1#$

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS PA-440 IS PROHIBITED.
PA-440 login: bigDonkBonkAdmin
Password:
Last login: Mon Nov  9 21:28:26 on ttyS0



Number of failed attempts since last successful login: 0



bigDonkBonkAdmin@PA-440> show config running

config {
  mgt-config {
    users {
      admin {
        phash $5$fqknpuqq$2b1Qq0KiHI/lppBjjtzwE7hIHHl52xfICtuQCLxwa5.;
        permissions {
          role-based {
            superuser yes;
          }
        }
      }
      sitstudent {
        permissions {
          role-based {
            superuser yes;
          }
        }
        description admin;
        phash $5$yuukakqc$mEfHLjFbt1YQWbIriisTGRed/ChCRO57CpldUabCNB7;
      }
      bigDonkBonkAdmin {
        permissions {
          role-based {
            superuser yes;
          }
        }
        phash $5$qkoxiewk$a.ND7JLzwgHRkJsBu20JaZQxu7Gyt1z69znR1enOpdA;
      }
      donkBonkEng1 {
        phash $5$wejqjqvg$5dIHxy3hIivGYmxgwvjzrIzNSTdxDBpKS.cYub6.7O0;
        permissions {
          role-based {
            superreader yes;
          }
        }
      }
      donkBonkEng2 {
        phash $5$vaiwdhhd$OdGx.nI6DD/e3PfOvlPyynl552Gb6i.FirpVVYeaSc7;
        permissions {
          role-based {
            superreader yes;
          }
        }
      }
    }
    password-complexity {
      enabled yes;
      minimum-length 8;
    }
  }
  shared {
    application;
    application-group;
    service;
    service-group;
    botnet {
      configuration {
        http {
          dynamic-dns {
            enabled yes;
            threshold 5;
          }
          malware-sites {
            enabled yes;
            threshold 5;
          }
          recent-domains {
            enabled yes;
            threshold 5;
          }
          ip-domains {
            enabled yes;
            threshold 10;
          }
          executables-from-unknown-sites {
            enabled yes;
            threshold 5;
          }
        }
        other-applications {
          irc yes;
        }
        unknown-applications {
          unknown-tcp {
            destinations-per-hour 10;
            sessions-per-hour 10;
            session-length {
              maximum-bytes 100;
              minimum-bytes 50;
            }
          }
          unknown-udp {
            destinations-per-hour 10;
            sessions-per-hour 10;
            session-length {
              maximum-bytes 100;
              minimum-bytes 50;
            }
          }
        }
      }
      report {
        topn 100;
        scheduled yes;
      }
    }
    server-profile {
      tacplus {
        TACACS {
          server {
            JUMP {
              address 172.16.0.44;
              secret -AQ==VUmGNWPdWuJt2J9MFpDMqf7If6U=puGOOXAA9pi+zTF76jUsHHEctcNmZGGxiXuI8n2UGZE=;
            }
            JUMP2 {
              address 172.16.0.45;
              secret -AQ==qmQnvaHTF5quvUkoQXHi8EiZZvQ=puGOOXAA9pi+zTF76jUsHOVRSn255prHAE5++8r8X6c=;
            }
          }
          timeout 3;
          protocol PAP;
        }
      }
    }
    authentication-profile {
      AUTH-TACACS {
        method {
          tacplus {
            server-profile TACACS;
          }
        }
        allow-list all;
      }
      AUTH-LOCAL {
        method {
          local-database;
        }
        allow-list all;
      }
    }
    authentication-sequence {
      SEQ-ADMIN {
        authentication-profiles [ AUTH-TACACS AUTH-LOCAL];
      }
    }
    log-settings {
      syslog {
        SYSLOG-JUMP {
          server {
            JUMP {
              server 172.16.0.44;
              transport UDP;
              port 514;
              format BSD;
              facility LOG_USER;
            }
            JUMP2 {
              server 172.16.0.45;
              transport UDP;
              port 514;
              format BSD;
              facility LOG_USER;
            }
          }
        }
      }
      profiles {
        LFP-WAZUH {
          match-list {
            ML-TRAFFIC {
              log-type traffic;
              filter "All Logs";
              send-syslog SYSLOG-JUMP;
            }
            ML-THREAT {
              log-type threat;
              filter "All Logs";
              send-syslog SYSLOG-JUMP;
            }
          }
        }
      }
      system {
        match-list {
          ML-SYS {
            filter "All Logs";
            send-syslog SYSLOG-JUMP;
          }
        }
      }
      config {
        match-list {
          ML-CFG {
            filter "All Logs";
            send-syslog SYSLOG-JUMP;
          }
        }
      }
    }
  }
  devices {
    localhost.localdomain {
      network {
        interface {
          ethernet {
            ethernet1/1 {
              link-state down;
            }
            ethernet1/2 {
              layer2 {
                lldp {
                  enable no;
                }
              }
            }
            ethernet1/5 {
              layer2 {
                lldp {
                  enable no;
                }
              }
            }
            ethernet1/3 {
              layer3 {
                ip {
                  172.16.0.105/30;
                }
                interface-management-profile PING-ONLY;
              }
            }
            ethernet1/4 {
              layer2 {
                lldp {
                  enable no;
                }
              }
            }
            ethernet1/6 {
              link-state auto;
              layer3 {
                ip {
                  172.16.0.117/30;
                }
                interface-management-profile PING-ONLY;
              }
            }
            ethernet1/7 {
              link-state auto;
              layer3 {
                ip {
                  172.16.0.121/30;
                }
                interface-management-profile PING-ONLY;
              }
            }
            ethernet1/8 {
              link-state down;
            }
          }
          loopback {
            units;
          }
          vlan {
            units {
              vlan.10 {
                ip {
                  172.16.0.92/29;
                }
                interface-management-profile PING-ONLY;
              }
              vlan.92 {
                ip {
                  172.16.0.81/29;
                }
                interface-management-profile PING-ONLY;
              }
            }
          }
          tunnel {
            units;
          }
        }
        vlan {
          OUTSIDE {
            virtual-interface {
              interface vlan.10;
            }
            interface [ ethernet1/2 ethernet1/5];
          }
          VL-DMZ {
            interface ethernet1/4;
            virtual-interface {
              interface vlan.92;
            }
          }
        }
        virtual-wire;
        profiles {
          monitor-profile {
            default {
              interval 3;
              threshold 5;
              action wait-recover;
            }
          }
          interface-management-profile {
            PING-ONLY {
              ping yes;
            }
          }
          zone-protection-profile {
            ZP-OUTSIDE {
              flood {
                tcp-syn {
                  enable yes;
                  syn-cookies {
                    alarm-rate 100;
                    activate-rate 200;
                    maximal-rate 2000;
                  }
                }
                udp {
                  enable yes;
                  red {
                    alarm-rate 500;
                    activate-rate 1000;
                    maximal-rate 5000;
                  }
                }
                icmp {
                  enable yes;
                  red {
                    alarm-rate 50;
                    activate-rate 100;
                    maximal-rate 500;
                  }
                }
                icmpv6 {
                  red {
                    alarm-rate 10000;
                    activate-rate 10000;
                    maximal-rate 40000;
                  }
                  enable no;
                }
                other-ip {
                  red {
                    alarm-rate 10000;
                    activate-rate 10000;
                    maximal-rate 40000;
                  }
                  enable no;
                }
              }
              scan {
                8001 {
                  action {
                    block-ip {
                      track-by source;
                      duration 300;
                    }
                  }
                  interval 2;
                  threshold 100;
                }
                8002 {
                  action {
                    block-ip {
                      track-by source;
                      duration 300;
                    }
                  }
                  interval 10;
                  threshold 100;
                }
                8003 {
                  action {
                    block-ip {
                      track-by source;
                      duration 300;
                    }
                  }
                  interval 2;
                  threshold 100;
                }
              }
              discard-ip-spoof yes;
              net-inspection {
                rule;
              }
            }
            ZP-DMZ {
              discard-ip-spoof yes;
              scan {
                8001 {
                  action {
                    block-ip {
                      track-by source;
                      duration 300;
                    }
                  }
                  interval 2;
                  threshold 100;
                }
                8002 {
                  action {
                    block-ip {
                      track-by source;
                      duration 300;
                    }
                  }
                  interval 10;
                  threshold 100;
                }
              }
            }
          }
        }
        ike {
          crypto-profiles {
            ike-crypto-profiles {
              default {
                encryption [ aes-128-cbc 3des];
                hash sha1;
                dh-group group2;
                lifetime {
                  hours 8;
                }
              }
              Suite-B-GCM-128 {
                encryption aes-128-cbc;
                hash sha256;
                dh-group group19;
                lifetime {
                  hours 8;
                }
              }
              Suite-B-GCM-256 {
                encryption aes-256-cbc;
                hash sha384;
                dh-group group20;
                lifetime {
                  hours 8;
                }
              }
            }
            ipsec-crypto-profiles {
              default {
                esp {
                  encryption [ aes-128-cbc 3des];
                  authentication sha1;
                }
                dh-group group2;
                lifetime {
                  hours 1;
                }
              }
              Suite-B-GCM-128 {
                esp {
                  encryption aes-128-gcm;
                  authentication none;
                }
                dh-group group19;
                lifetime {
                  hours 1;
                }
              }
              Suite-B-GCM-256 {
                esp {
                  encryption aes-256-gcm;
                  authentication none;
                }
                dh-group group20;
                lifetime {
                  hours 1;
                }
              }
            }
            global-protect-app-crypto-profiles {
              default {
                encryption aes-128-cbc;
                authentication sha1;
              }
            }
          }
        }
        qos {
          profile {
            default {
              class-bandwidth-type {
                mbps {
                  class {
                    class1 {
                      priority real-time;
                    }
                    class2 {
                      priority high;
                    }
                    class3 {
                      priority high;
                    }
                    class4 {
                      priority medium;
                    }
                    class5 {
                      priority medium;
                    }
                    class6 {
                      priority low;
                    }
                    class7 {
                      priority low;
                    }
                    class8 {
                      priority low;
                    }
                  }
                }
              }
            }
          }
        }
        virtual-router {
          default {
            protocol {
              bgp {
                enable no;
                dampening-profile {
                  default {
                    cutoff 1.25;
                    reuse 0.5;
                    max-hold-time 900;
                    decay-half-life-reachable 300;
                    decay-half-life-unreachable 900;
                    enable yes;
                  }
                }
              }
              ospf {
                router-id 6.6.6.6;
                area {
                  0.0.0.0 {
                    type {
                      normal;
                    }
                    interface {
                      vlan.10 {
                        enable yes;
                        passive no;
                        metric 10;
                        link-type {
                          broadcast;
                        }
                        authentication OSPF-MD5;
                      }
                      ethernet1/3 {
                        enable yes;
                        passive no;
                        metric 10;
                        link-type {
                          broadcast;
                        }
                        authentication OSPF-MD5;
                      }
                      vlan.92 {
                        enable yes;
                        passive yes;
                        metric 10;
                      }
                      ethernet1/6 {
                        enable yes;
                        passive no;
                        metric 500;
                        authentication OSPF-MD5;
                      }
                      ethernet1/7 {
                        enable yes;
                        passive no;
                        metric 500;
                        authentication OSPF-MD5;
                      }
                    }
                  }
                }
                enable yes;
                reject-default-route no;
                auth-profile {
                  OSPF-MD5 {
                    md5 {
                      1 {
                        key -AQ==pRYy08r2QTz+1h+htj7ReUL++ik=gG6fEbkrPh4lDYGZUpo8lg==;
                        preferred yes;
                      }
                    }
                  }
                }
              }
            }
            interface [ vlan.10 ethernet1/3 vlan.92 ethernet1/6 ethernet1/7];
            routing-table {
              ip {
                static-route;
              }
            }
          }
        }
      }
      deviceconfig {
        system {
          ip-address 172.16.0.41;
          netmask 255.255.255.240;
          update-server updates.paloaltonetworks.com;
          update-schedule {
            threats {
              recurring {
                weekly {
                  day-of-week wednesday;
                  at 01:02;
                  action download-only;
                }
              }
            }
          }
          timezone Asia/Singapore;
          service {
            disable-telnet yes;
            disable-http yes;
            disable-ssh no;
            disable-https no;
          }
          hostname PA-440;
          device-telemetry {
            device-health-performance yes;
            product-usage yes;
            threat-prevention yes;
            region Americas;
          }
          domain gruppesechs.sitict.net;
          login-banner "WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS PA-440 IS PROHIBITED.";
          type {
            static;
          }
          default-gateway 172.16.0.33;
          ntp-servers {
            primary-ntp-server {
              ntp-server-address 172.16.0.44;
              authentication-type {
                symmetric-key {
                  key-id 1;
                  algorithm {
                    md5 {
                      authentication-key -AQ==d1HcITRQi08k/OiZWh5fdR/Td/M=4jUAsqaapG0rM70S1CEyig==;
                    }
                  }
                }
              }
            }
            secondary-ntp-server {
              ntp-server-address 172.16.0.45;
              authentication-type {
                symmetric-key {
                  key-id 1;
                  algorithm {
                    md5 {
                      authentication-key -AQ==d1HcITRQi08k/OiZWh5fdR/Td/M=4jUAsqaapG0rM70S1CEyig==;
                    }
                  }
                }
              }
            }
          }
          authentication-profile SEQ-ADMIN;
          permitted-ip {
            172.16.0.44/32;
            172.16.0.45/32;
          }
        }
        setting {
          config {
            rematch yes;
          }
          management {
            hostname-type-in-syslog FQDN;
            admin-lockout {
              failed-attempts 5;
              lockout-time 5;
            }
            idle-timeout 10;
          }
        }
      }
      vsys {
        vsys1 {
          application;
          application-group;
          zone {
            outside {
              network {
                layer3 vlan.10;
                zone-protection-profile ZP-OUTSIDE;
                log-setting LFP-WAZUH;
              }
            }
            outside-l2 {
              network {
                layer2 [ ethernet1/5 ethernet1/2];
              }
            }
            inside {
              network {
                layer3 [ ethernet1/3 ethernet1/6 ethernet1/7];
              }
            }
            dmz {
              network {
                layer3 vlan.92;
                zone-protection-profile ZP-DMZ;
                log-setting LFP-WAZUH;
              }
            }
            dmz-l2 {
              network {
                layer2 ethernet1/4;
              }
            }
          }
          service {
            WAZUH-AGENT {
              protocol {
                tcp {
                  port 1514-1515;
                }
              }
            }
          }
          service-group;
          schedule {
            TXN-WINDOW {
              schedule-type {
                recurring {
                  weekly {
                    monday 01:00-03:00;
                    tuesday 01:00-03:00;
                    wednesday 01:00-03:00;
                    thursday 01:00-03:00;
                    friday 01:00-03:00;
                  }
                }
              }
            }
          }
          rulebase {
            security {
              rules {
                ALLOW-OSPF 796ccf63-6546-4717-8139-286e0dce7c01 {
                  from [ outside inside];
                  to [ outside inside];
                  source any;
                  destination any;
                  source-user any;
                  category any;
                  application ospf;
                  service any;
                  action allow;
                  log-end no;
                }
                INBOUND-WEB 65923444-3f19-4aac-9f3f-6532931f72aa {
                  from outside;
                  to dmz;
                  source any;
                  destination WEB-SVR;
                  source-user any;
                  category any;
                  application [ web-browsing ssl];
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                INBOUND-DNS 6c64b41d-b3e2-4d99-8c72-5bf10a7c9fb0 {
                  from outside;
                  to dmz;
                  source any;
                  destination DNS-SVR;
                  source-user any;
                  category any;
                  application dns;
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                JUMP-SSH-IN 568360ad-f4e1-4e60-bcf6-04b199c82faf {
                  from outside;
                  to dmz;
                  source any;
                  destination JUMP-SVR;
                  source-user any;
                  category any;
                  application ssh;
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                INTERNAL-TO-DNS baef6b8a-34d8-4c5a-8162-f87cbc5eec04 {
                  from inside;
                  to dmz;
                  source INTERNET-USERS;
                  destination DNS-SVR;
                  source-user any;
                  category any;
                  application dns;
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                INTERNAL-TO-WEB a6838ff7-ea8d-4679-a819-7074f3ca9b6d {
                  from inside;
                  to dmz;
                  source INTERNET-USERS;
                  destination WEB-SVR;
                  source-user any;
                  category any;
                  application [ web-browsing ssl];
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                IT-TO-JUMP-SSH bf1c53c4-2075-4a2c-88e8-e64752a845c0 {
                  from inside;
                  to dmz;
                  source IT-NET;
                  destination [ JUMP-SVR JUMP2-SVR];
                  source-user any;
                  category any;
                  application ssh;
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                HR-IT-INTERNET f524d3ae-2356-4950-8a5b-d7c6a0b706c0 {
                  from inside;
                  to outside;
                  source INTERNET-USERS;
                  destination any;
                  source-user any;
                  category any;
                  action allow;
                  log-end yes;
                  application any;
                  service any;
                  log-setting LFP-WAZUH;
                }
                TXN-INTERNET-TIMED 28b12428-79d2-4bd1-9688-e3452f8a833f {
                  from inside;
                  to [ outside dmz];
                  source TXN-SVR;
                  destination any;
                  source-user any;
                  category any;
                  application [ web-browsing ssl dns];
                  service application-default;
                  schedule TXN-WINDOW;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                DMZ-OUTBOUND-NTP 5dff2fdc-5128-49a3-946a-981847fb9596 {
                  from dmz;
                  to outside;
                  source [ JUMP-SVR JUMP2-SVR];
                  destination any;
                  source-user any;
                  category any;
                  application [ ntp dns];
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                DNS-SVR-FORWARD 981ffd66-c6bb-43c2-bcdc-228e9d8c1e5f {
                  from dmz;
                  to outside;
                  source DNS-SVR;
                  destination [ ICT-DNS DNS-FORWARDERS-1 DNS-FORWARDERS-2];
                  source-user any;
                  category any;
                  application dns;
                  service application-default;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                WAZUH-AGENTS 3c59c77a-d82c-4bec-86a5-5ce8f0d552e6 {
                  from inside;
                  to dmz;
                  source SERVERS;
                  destination JUMP-SVR;
                  source-user any;
                  category any;
                  application any;
                  service WAZUH-AGENT;
                  action allow;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
                DENY-ALL 2a84bdde-7a09-4106-b62b-06f2fbb2bd34 {
                  rule-type interzone;
                  from any;
                  to any;
                  source any;
                  destination any;
                  source-user any;
                  category any;
                  application any;
                  service any;
                  action deny;
                  log-end yes;
                  log-setting LFP-WAZUH;
                }
              }
            }
            nat {
              rules;
            }
            dos {
              rules {
                DOS-DMZ-SERVERS d09f38a2-089f-44db-9da5-e1b96be49638 {
                  from {
                    zone outside;
                  }
                  to {
                    zone dmz;
                  }
                  source any;
                  destination [ WEB-SVR DNS-SVR JUMP-SVR];
                  source-user any;
                  service any;
                  action {
                    protect;
                  }
                  protection {
                    classified {
                      profile DOS-PER-CLIENT;
                      classification-criteria {
                        address source-ip-only;
                      }
                    }
                  }
                  log-setting LFP-WAZUH;
                }
              }
            }
          }
          address {
            STAFF {
              ip-netmask 172.16.0.0/27;
            }
            ATM {
              ip-netmask 172.16.0.48/29;
            }
            HR-NET {
              ip-netmask 172.16.0.56/29;
            }
            IT-NET {
              ip-netmask 172.16.0.64/29;
            }
            SERVERS {
              ip-netmask 172.16.0.72/29;
            }
            DMZ-SUBNET {
              ip-netmask 172.16.0.80/29;
            }
            FILE-SVR {
              ip-netmask 172.16.0.74/32;
            }
            ACCT-SVR {
              ip-netmask 172.16.0.75/32;
            }
            TXN-SVR {
              ip-netmask 172.16.0.76/32;
            }
            DHCP-SVR {
              ip-netmask 172.16.0.77/32;
            }
            WEB-SVR {
              ip-netmask 172.16.0.82/32;
            }
            DNS-SVR {
              ip-netmask 172.16.0.83/32;
            }
            JUMP-SVR {
              ip-netmask 172.16.0.84/32;
            }
            JUMP2-SVR {
              ip-netmask 172.16.0.85/32;
            }
            DNS-FORWARDERS-1 {
              ip-netmask 8.8.8.8/32;
            }
            DNS-FORWARDERS-2 {
              ip-netmask 1.1.1.1/32;
            }
            ICT-DNS {
              ip-netmask 172.18.4.32/32;
            }
          }
          address-group {
            STAFF-ATM {
              static [ STAFF ATM];
            }
            HR-IT {
              static [ HR-NET IT-NET];
            }
            INTERNET-USERS {
              static [ HR-NET IT-NET];
            }
          }
          profiles {
            dos-protection {
              DOS-PER-CLIENT {
                type classified;
                resource {
                  sessions {
                    enabled yes;
                    max-concurrent-limit 50;
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}

�
ASA-1010# show run
ASA-1010# show running-config
: Saved

:
: Serial Number: JAD2801087C
: Hardware:   FPR-1010, 7197 MB RAM, CPU Atom C3000 series 2200 MHz, 1 CPU (4 cores)
:
ASA Version 9.20(2)2
!
hostname ASA-1010
domain-name gruppesechs.sitict.net
enable password ***** pbkdf2
service-module 1 keepalive-timeout 4
service-module 1 keepalive-counter 6
names
no mac-address auto

!
interface Vlan1
 shutdown
 no nameif
 no security-level
 no ip address
!
interface Vlan69
 description Primary_Internal_Servers_Gateway_SVI
 nameif server
 security-level 60
 ip address 172.16.0.73 255.255.255.248
!
interface Ethernet1/1
 description Primary_Serial_Link_to_PA440_eth1_3
 no switchport
 speed 1000
 duplex full
 nameif outside
 security-level 0
 ip address 172.16.0.106 255.255.255.252
 ospf cost 10
 ospf message-digest-key 1 md5 *****
 ospf authentication message-digest
!
interface Ethernet1/2
 description PA_Bypass_Link_to_R2_G0_1_1
 no switchport
 speed 1000
 duplex full
 nameif wan-backup
 security-level 0
 ip address 172.16.0.114 255.255.255.252
 ospf cost 500
 ospf message-digest-key 1 md5 *****
 ospf authentication message-digest
!
interface Ethernet1/3
 description Inside_Transit_to_DSW1_G1_0_11
 no switchport
 speed 1000
 duplex full
 nameif inside
 security-level 100
 ip address 172.16.0.97 255.255.255.252
 ospf cost 10
 ospf message-digest-key 1 md5 *****
 ospf authentication message-digest
!
interface Ethernet1/4
 description Inside_Transit_to_DSW2_G1_0_11
 no switchport
 speed 1000
 duplex full
 nameif inside2
 security-level 100
 ip address 172.16.0.101 255.255.255.252
 ospf cost 10
 ospf message-digest-key 1 md5 *****
 ospf authentication message-digest
!
interface Ethernet1/5
 description PA_Bypass_Link_to_R1_G0_1_1
 no switchport
 speed 1000
 duplex full
 nameif wan-backup2
 security-level 0
 ip address 172.16.0.110 255.255.255.252
 ospf cost 500
 ospf message-digest-key 1 md5 *****
 ospf authentication message-digest
!
interface Ethernet1/6
 description Direct_to_Supermicro_eno2_vmbr1_VLAN69
 switchport
 switchport access vlan 69
 no security-level
!
interface Ethernet1/7
 description UNUSED_PORT
 switchport
 power inline auto
 shutdown
 no security-level
!
interface Ethernet1/8
 description UNUSED_PORT
 switchport
 power inline auto
 shutdown
 no security-level
!
interface Management1/1
 description OOB_Management_VLAN99
 management-only
 nameif management
 security-level 100
 ip address 172.16.0.42 255.255.255.240
!
!
time-range TXN-WINDOW
 periodic weekdays 1:00 to 3:00
!
banner motd WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS ASA IS PROHIBITED.
ftp mode passive
clock timezone Singapore
dns domain-lookup outside
dns server-group DefaultDNS
 name-server 208.67.222.222 outside
 name-server 208.67.220.220 outside
 domain-name gruppesechs.sitict.net
no object-group-search access-control
object network obj_any
 subnet 0.0.0.0 0.0.0.0
object network STAFF
 subnet 172.16.0.0 255.255.255.224
object network ATM
 subnet 172.16.0.48 255.255.255.248
object network HR
 subnet 172.16.0.56 255.255.255.248
object network IT
 subnet 172.16.0.64 255.255.255.248
object network SERVERS
 subnet 172.16.0.72 255.255.255.248
object network DMZ-SUBNET
 subnet 172.16.0.80 255.255.255.248
object network INTERNAL
 subnet 172.16.0.0 255.255.255.128
object network FILE-SVR
 host 172.16.0.74
object network ACCT-SVR
 host 172.16.0.75
object network TXN-SVR
 host 172.16.0.76
object network DHCP-SVR
 host 172.16.0.77
object network WEB-SVR
 host 172.16.0.82
object network DNS-SVR
 host 172.16.0.83
object network JUMP-SVR
 host 172.16.0.84
object network JUMP2-SVR
 host 172.16.0.85
object-group network STAFF-ATM
 network-object object STAFF
 network-object object ATM
object-group network HR-IT
 network-object object HR
 network-object object IT
object-group network DSWs
 network-object host 172.16.0.98
 network-object host 172.16.0.102
object-group network DSW-SVIS
 network-object host 172.16.0.2
 network-object host 172.16.0.3
 network-object host 172.16.0.50
 network-object host 172.16.0.51
 network-object host 172.16.0.58
 network-object host 172.16.0.59
 network-object host 172.16.0.66
 network-object host 172.16.0.67
 network-object host 172.16.0.98
 network-object host 172.16.0.102
object-group network JUMP-SVRs
 network-object object JUMP-SVR
 network-object object JUMP2-SVR
access-group OUTSIDE_IN in interface wan-backup
access-group OUTSIDE_IN in interface outside
access-group INSIDE_IN in interface inside
access-group INSIDE_IN in interface inside2
access-group OUTSIDE_IN in interface wan-backup2
access-group SERVER_IN in interface server
access-list INSIDE_IN extended permit ospf any any
access-list INSIDE_IN extended permit udp object-group DSW-SVIS object DHCP-SVR eq bootps
access-list INSIDE_IN extended permit tcp object HR object FILE-SVR eq 445
access-list INSIDE_IN extended permit icmp object HR object FILE-SVR
access-list INSIDE_IN extended permit ip object-group STAFF-ATM object ACCT-SVR
access-list INSIDE_IN extended permit ip object-group STAFF-ATM object TXN-SVR
access-list INSIDE_IN extended permit udp object-group HR-IT object DNS-SVR eq domain
access-list INSIDE_IN extended permit tcp object-group HR-IT object DNS-SVR eq domain
access-list INSIDE_IN extended permit tcp object-group HR-IT object WEB-SVR eq https
access-list INSIDE_IN extended permit tcp object IT object-group JUMP-SVRs eq ssh
access-list INSIDE_IN extended deny ip any object SERVERS log
access-list INSIDE_IN extended deny ip any object DMZ-SUBNET log
access-list INSIDE_IN extended deny ip object-group STAFF-ATM any log
access-list INSIDE_IN extended permit ip object-group HR-IT any
access-list INSIDE_IN extended deny ip any any log
access-list INSIDE_IN extended permit tcp object-group HR-IT object WEB-SVR eq www
access-list SERVER_IN extended permit tcp object SERVERS object JUMP-SVR range 1514 1515
access-list SERVER_IN extended permit udp object DHCP-SVR eq bootps object-group DSW-SVIS eq bootps
access-list SERVER_IN extended deny ip object SERVERS object INTERNAL log
access-list SERVER_IN extended permit tcp object TXN-SVR any eq www time-range TXN-WINDOW
access-list SERVER_IN extended permit tcp object TXN-SVR any eq https time-range TXN-WINDOW
access-list SERVER_IN extended permit udp object TXN-SVR object DNS-SVR eq domain time-range TXN-WINDOW
access-list SERVER_IN extended deny ip any any log
access-list OUTSIDE_IN extended permit ospf any any
access-list OUTSIDE_IN extended deny ip any any log
access-list MPF_INTERNAL_SVRS extended permit tcp any object SERVERS
pager lines 24
logging enable
logging timestamp
logging monitor informational
logging buffered informational
logging trap informational
logging asdm informational
logging host management 172.16.0.44
logging host management 172.16.0.45
mtu server 1500
mtu outside 1500
mtu wan-backup 1500
mtu inside 1500
mtu inside2 1500
mtu wan-backup2 1500
mtu management 1500
icmp unreachable rate-limit 1 burst-size 1
no asdm history enable
arp timeout 14400
no arp permit-nonconnected
arp rate-limit 16384
router ospf 1
 router-id 5.5.5.5
 network 172.16.0.96 255.255.255.252 area 0
 network 172.16.0.100 255.255.255.252 area 0
 network 172.16.0.104 255.255.255.252 area 0
 network 172.16.0.108 255.255.255.252 area 0
 network 172.16.0.112 255.255.255.252 area 0
 log-adj-changes
 redistribute connected subnets
!
route management 0.0.0.0 0.0.0.0 172.20.11.254 1
route management 0.0.0.0 0.0.0.0 172.16.0.33 1
timeout xlate 3:00:00
timeout pat-xlate 0:00:30
timeout conn 1:00:00 half-closed 0:10:00 udp 0:02:00 sctp 0:02:00 icmp 0:00:02
timeout sunrpc 0:10:00 h323 0:05:00 h225 1:00:00 mgcp 0:05:00 mgcp-pat 0:05:00
timeout sip 0:30:00 sip_media 0:02:00 sip-invite 0:03:00 sip-disconnect 0:02:00
timeout sip-provisional-media 0:02:00 uauth 0:05:00 absolute
timeout tcp-proxy-reassembly 0:01:00
timeout floating-conn 0:00:00
timeout conn-holddown 0:00:15
timeout igp stale-route 0:01:10
aaa-server TACACS protocol tacacs+
aaa-server TACACS (management) host 172.16.0.44
 key *****
aaa-server TACACS (management) host 172.16.0.45
 key *****
user-identity default-domain LOCAL
aaa authentication ssh console TACACS LOCAL
aaa authentication enable console TACACS LOCAL
aaa authentication serial console TACACS LOCAL
aaa authorization command TACACS LOCAL
aaa accounting command TACACS
aaa local authentication attempts max-fail 5
aaa authorization exec authentication-server auto-enable
aaa authentication login-history
http 192.168.45.0 255.255.255.0 management
no snmp-server location
no snmp-server contact
crypto ipsec security-association pmtu-aging infinite
crypto ca trustpoint _SmartCallHome_ServerCA
 no validation-usage
 crl configure
crypto ca trustpoint _SmartCallHome_ServerCA2
 no validation-usage
 crl configure
crypto ca trustpool policy
 auto-import
crypto ca certificate chain _SmartCallHome_ServerCA
 certificate ca 0a0142800000014523c844b500000002
    30820560 30820348 a0030201 0202100a 01428000 00014523 c844b500 00000230
    0d06092a 864886f7 0d01010b 0500304a 310b3009 06035504 06130255 53311230
    10060355 040a1309 4964656e 54727573 74312730 25060355 0403131e 4964656e
    54727573 7420436f 6d6d6572 6369616c 20526f6f 74204341 2031301e 170d3134
    30313136 31383132 32335a17 0d333430 31313631 38313232 335a304a 310b3009
    06035504 06130255 53311230 10060355 040a1309 4964656e 54727573 74312730
    25060355 0403131e 4964656e 54727573 7420436f 6d6d6572 6369616c 20526f6f
    74204341 20313082 0222300d 06092a86 4886f70d 01010105 00038202 0f003082
    020a0282 020100a7 5019de3f 993dd433 46f16f51 6182b2a9 4f8f6789 5d84d953
    dd0c28d9 d7f0ffae 95437299 f9b55d7c 8ac142e1 315074d1 810d7ccd 9b21ab43
    e2acad5e 866ef309 8a1f5a32 bda2eb94 f9e85c0a ecff98d2 af71b3b4 539f4e87
    ef92bcbd ec4f3230 884b175e 57c453c2 f602978d d9622bbf 241f628d dfc3b829
    4b49783c 93608822 fc99da36 c8c2a2d4 2c540067 356e73bf 0258f0a4 dde5b0a2
    267acae0 36a51916 f5fdb7ef ae3f40f5 6d5a04fd ce34ca24 dc74231b 5d331312
    5dc40125 f630dd02 5d9fe0d5 47bdb4eb 1ba1bb49 49d89f5b 02f38ae4 2490e462
    4f4fc1af 8b0e7417 a8d17288 6a7a0149 ccb44679 c617b1da 981e0759 fa752185
    65dd9056 cefbaba5 609dc49d f952b08b bd87f98f 2b230a23 763bf733 e1c900f3
    69f94ba2 e04ebc7e 93398407 f744707e fe075ae5 b1acd118 ccf235e5 494908ca
    56c93dfb 0f187d8b 3bc113c2 4d8fc94f 0e37e91f a10e6adf 622ecb35 0651792c
    c82538f4 fa4ba789 5c9cd2e3 0d39864a 747cd559 87c23f4e 0c5c52f4 3df75282
    f1eaa3ac fd49341a 28f34188 3a13eee8 deff991d 5fbacbe8 1ef2b950 60c031d3
    73e5efbe a0ed330b 74be2020 c4676cf0 08037a55 807f464e 96a7f41e 3ee1f6d8
    09e13364 2b63d732 5e9ff9c0 7b0f786f 97bc939a f99c1290 787a8087 15d77274
    9c557478 b1bae16e 7004ba4f a0ba68c3 7bff31f0 733d3d94 2ab10b41 0ea0fe4d
    88656b79 33b4d702 03010001 a3423040 300e0603 551d0f01 01ff0404 03020106
    300f0603 551d1301 01ff0405 30030101 ff301d06 03551d0e 04160414 ed4419c0
    d3f0068b eea47bbe 42e72654 c88e3676 300d0609 2a864886 f70d0101 0b050003
    82020100 0dae9032 f6a64b7c 44761961 1e2728cd 5e54ef25 bce30890 f929d7ae
    6808e194 0058ef2e 2e7e5352 8cb65c07 ea88ba99 8b5094d7 8280df61 090093ad
    0d14e6ce c1f23794 78b05f9c b3a273b8 8f059338 cd8d3eb0 b8fbc0cf b1f2ec2d
    2d1bccec aa9ab3aa 60821b2d 3bc3843d 578a961e 9c75b8d3 30cd6008 8390d38e
    54f14d66 c05d7403 40a3ee85 7ec21f77 9c06e8c1 a7185d52 95edc9dd 259e6dfa
    a9eda33a 34d0597b daed50f3 35bfedeb 144d31c7 60f4daf1 879ce248 e2c6c537
    fb0610fa 75596631 4729da76 9a1ce982 aeef9ab9 51f78823 9a699562 3ce55580
    36d75402 fff1b95d ced4236f d845844a 5b65ef89 0cdd14a7 20cb18a5 25b40df9
    01f0a2d2 f400c874 8ea12a48 8e65db13 c4e22517 7debbe87 5b172054 51934a53
    030bec5d ca33ed62 fd45c72f 5bdc58a0 8039e6fa d7fe1314 a6ed3d94 4a4274d4
    c3775973 cd8f46be 5538effa e89132ea 97580422 de38c3cc bc6dc933 3a6a0a69
    3fa0c8ea 728f8c63 8623bd6d 3c969e95 e0494caa a2b92a1b 9c368178 edc3e846
    e2265944 751ed975 8951cd10 849d6160 cb5df997 224d8e98 e6e37ff6 5bbbaecd
    ca4a816b 5e0bf351 e1742be9 7e27a7d9 99494ef8 a580db25 0f1c6362 8ac93367
    6b3c1083 c6addea8 cd168e8d f0073771 9ff2abfc 41f5c18b ec00375d 09e54e80
    effab15c 3806a51b 4ae1dc38 2d3cdcab 1f901ad5 4a9ceed1 706cccee f457f818
    ba846e87
  quit
crypto ca certificate chain _SmartCallHome_ServerCA2
 certificate ca 0509
    308205b7 3082039f a0030201 02020205 09300d06 092a8648 86f70d01 01050500
    3045310b 30090603 55040613 02424d31 19301706 0355040a 13105175 6f566164
    6973204c 696d6974 6564311b 30190603 55040313 1251756f 56616469 7320526f
    6f742043 41203230 1e170d30 36313132 34313832 3730305a 170d3331 31313234
    31383233 33335a30 45310b30 09060355 04061302 424d3119 30170603 55040a13
    1051756f 56616469 73204c69 6d697465 64311b30 19060355 04031312 51756f56
    61646973 20526f6f 74204341 20323082 0222300d 06092a86 4886f70d 01010105
    00038202 0f003082 020a0282 0201009a 18ca4b94 0d002daf 03298af0 0f81c8ae
    4c19851d 089fab29 4485f32f 81ad321e 9046bfa3 86261a1e fe7e1c18 3a5c9c60
    172a3a74 8333307d 615411cb edabe0e6 d2a27ef5 6b6f18b7 0a0b2dfd e93eef0a
    c6b310e9 dcc24617 f85dfda4 daff9e49 5a9ce633 e62496f7 3fba5b2b 1c7a35c2
    d667feab 66508b6d 28602bef d760c3c7 93bc8d36 91f37ff8 db1113c4 9c7776c1
    aeb7026a 817aa945 83e205e6 b956c194 378f4871 6322ec17 6507958a 4bdf8fc6
    5a0ae5b0 e35f5e6b 11ab0cf9 85eb44e9 f80473f2 e9fe5c98 8cf573af 6bb47ecd
    d45c022b 4c39e1b2 95952d42 87d7d5b3 9043b76c 13f1dedd f6c4f889 3fd175f5
    92c391d5 8a88d090 ecdc6dde 89c26571 968b0d03 fd9cbf5b 16ac92db eafe797c
    adebaff7 16cbdbcd 252be51f fb9a9fe2 51cc3a53 0c48e60e bdc9b476 0652e611
    13857263 0304e004 362b2019 02e874a7 1fb6c956 66f07525 dc67c10e 616088b3
    3ed1a8fc a3da1db0 d1b12354 df44766d ed41d8c1 b222b653 1cdf351d dca1772a
    31e42df5 e5e5dbc8 e0ffe580 d70b63a0 ff33a10f ba2c1515 ea97b3d2 a2b5bef2
    8c961e1a 8f1d6ca4 6137b986 7333d797 969e237d 82a44c81 e2a1d1ba 675f9507
    a32711ee 16107bbc 454a4cb2 04d2abef d5fd0c51 ce506a08 31f991da 0c8f645c
    03c33a8b 203f6e8d 673d3ad6 fe7d5b88 c95efbcc 61dc8b33 77d34432 35096204
    921610d8 9e2747fb 3b21e3f8 eb1d5b02 03010001 a381b030 81ad300f 0603551d
    130101ff 04053003 0101ff30 0b060355 1d0f0404 03020106 301d0603 551d0e04
    1604141a 8462bc48 4c332504 d4eed0f6 03c41946 d1946b30 6e060355 1d230467
    30658014 1a8462bc 484c3325 04d4eed0 f603c419 46d1946b a149a447 3045310b
    30090603 55040613 02424d31 19301706 0355040a 13105175 6f566164 6973204c
    696d6974 6564311b 30190603 55040313 1251756f 56616469 7320526f 6f742043
    41203282 02050930 0d06092a 864886f7 0d010105 05000382 0201003e 0a164d9f
    065ba8ae 715d2f05 2f67e613 4583c436 f6f3c026 0c0db547 645df8b4 72c946a5
    03182755 89787d76 ea963480 1720dce7 83f88dfc 07b8da5f 4d2e67b2 84fdd944
    fc775081 e67cb4c9 0d0b7253 f8760707 4147960c fbe08226 93558cfe 221f6065
    7c5fe726 b3f73290 9850d437 7155f692 2178f795 79faf82d 26876656 3077a637
    78335210 58ae3f61 8ef26ab1 ef187e4a 5963ca8d a256d5a7 2fbc561f cf39c1e2
    fb0aa815 2c7d4d7a 63c66c97 443cd26f c34a170a f890d257 a21951a5 2d9741da
    074fa950 da908d94 46e13ef0 94fd1000 38f53be8 40e1b46e 561a20cc 6f588ded
    2e458fd6 e9933fe7 b12cdf3a d6228cdc 84bb226f d0f8e4c6 39e90488 3cc3baeb
    557a6d80 9924f56c 01fbf897 b0945beb fdd26ff1 77680d35 6423acb8 55a103d1
    4d4219dc f8755956 a3f9a849 79f8af0e b911a07c b76aed34 d0b62662 381a870c
    f8e8fd2e d3907f07 912a1dd6 7e5c8583 99b03808 3fe95ef9 3507e4c9 626e577f
    a75095f7 bac89be6 8ea201c5 d666bf79 61f33c1c e1b9825c 5da0c3e9 d848bd19
    a2111419 6eb2861b 683e4837 1a88b75d 965e9cc7 ef276208 e291195c d2f121dd
    ba174282 97718153 31a99ff6 7d62bf72 e1a3931d cc8a265a 0938d0ce d70d8016
    b478a53a 874c8d8a a5d54697 f22c10b9 bc5422c0 01506943 9ef4b2ef 6df8ecda
    f1e3b1ef df918f54 2a0b25c1 2619c452 100565d5 8210eac2 31cd2e
  quit
telnet timeout 10
ssh stack ciscossh
ssh stricthostkeycheck
ssh timeout 10
ssh key-exchange group dh-group14-sha256
ssh 0.0.0.0 0.0.0.0 management
ssh 172.16.0.44 255.255.255.255 management
ssh 172.16.0.45 255.255.255.255 management
console timeout 0
threat-detection basic-threat
threat-detection statistics access-list
threat-detection statistics tcp-intercept rate-interval 30 burst-rate 400 average-rate 200
ntp authentication-key 1 md5 *****
ntp authenticate
ntp server 172.16.0.44 key 1 source management prefer
ntp server 172.16.0.45 key 1 source management
dynamic-access-policy-record DfltAccessPolicy
username sitstudent password ***** pbkdf2 privilege 15
username sitadmin password ***** pbkdf2 privilege 15
username bigDonkBonkAdmin password ***** pbkdf2 privilege 15
username donkBonkEng2 password ***** pbkdf2 privilege 5
username donkBonkEng1 password ***** pbkdf2 privilege 5
!
class-map CM_SVR_PROTECT
 match access-list MPF_INTERNAL_SVRS
class-map inspection_default
 match default-inspection-traffic
!
!
policy-map type inspect dns preset_dns_map
 parameters
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
policy-map PM_INSIDE_PROTECT
 class CM_SVR_PROTECT
  set connection conn-max 500 embryonic-conn-max 50 per-client-max 50 per-client-embryonic-max 10
  set connection timeout embryonic 0:00:15
policy-map global_policy
 class inspection_default
  inspect ftp
  inspect h323 h225
  inspect h323 ras
  inspect ip-options
  inspect netbios
  inspect rsh
  inspect rtsp
  inspect skinny
  inspect esmtp
  inspect sqlnet
  inspect sunrpc
  inspect tftp
  inspect sip
  inspect dns preset_dns_map
  inspect icmp
  inspect icmp error
policy-map type inspect dns migrated_dns_map_2
 parameters
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
policy-map type inspect dns migrated_dns_map_1
 parameters
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
!
service-policy global_policy global
service-policy PM_INSIDE_PROTECT interface inside
service-policy PM_INSIDE_PROTECT interface inside2
privilege cmd level 5 mode exec command ping
privilege cmd level 5 mode exec command traceroute
privilege show level 5 mode exec command running-config
privilege show level 5 mode exec command interface
privilege show level 5 mode exec command access-list
privilege show level 5 mode exec command route
privilege show level 5 mode exec command ospf
prompt hostname context
no call-home reporting anonymous
call-home
 profile License
  destination address http https://tools.cisco.com/its/service/oddce/services/DDCEService
  destination transport-method http
 profile CiscoTAC-1
  no active
  destination address http https://tools.cisco.com/its/service/oddce/services/DDCEService
  destination address email callhome@cisco.com
  destination transport-method http
  subscribe-to-alert-group diagnostic
  subscribe-to-alert-group environment
  subscribe-to-alert-group inventory periodic monthly
  subscribe-to-alert-group configuration periodic monthly
  subscribe-to-alert-group telemetry periodic daily
Cryptochecksum:23bc2eb95547a74d299dd61d584b489a
: end
ASA-1010#
