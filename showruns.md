C  
WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW1 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin  
Password:

DSW1\>  
Nov  6 13:21:47.637: %SEC\_LOGIN-5-LOGIN\_SUCCESS: Login Success \[user: bigDonkBon                                     kAdmin\] \[Source: LOCAL\] \[localport: 0\] at 13:21:47 SGT Fri Nov 6 2026  
DSW1\>en  
Password:  
DSW1\#terminal length 0  
DSW1\#show running-config  
Building configuration...

Current configuration : 21341 bytes  
\!  
\! Last configuration change at 17:03:07 SGT Wed Nov 4 2026 by bigDonkBonkAdmin  
\!  
version 17.12  
service timestamps debug datetime msec  
service timestamps log datetime msec localtime  
service password-encryption  
platform punt-keepalive disable-kernel-core  
\!  
hostname DSW1  
\!  
\!  
vrf definition Mgmt-vrf  
 \!  
 address-family ipv4  
 exit-address-family  
 \!  
 address-family ipv6  
 exit-address-family  
\!  
vrf definition OOB  
 \!  
 address-family ipv4  
 exit-address-family  
\!  
aaa new-model  
\!  
\!  
aaa group server tacacs+ tac-group  
 server name TACACS1  
 server name TACACS2  
 ip vrf forwarding OOB  
 ip tacacs source-interface Vlan99  
\!  
aaa authentication login default group tac-group local  
aaa authentication login CONSOLE group tac-group local  
aaa authorization config-commands  
aaa authorization exec default group tac-group local  
aaa authorization commands 15 default group tac-group local  
aaa accounting exec default start-stop group tac-group  
aaa accounting commands 15 default stop-only group tac-group  
\!  
\!  
aaa session-id common  
clock timezone SGT 8 0  
switch 1 provision c9300l-24t-4g  
\!  
\!  
\!  
\!  
ip routing  
\!  
\!  
\!  
\!  
\!  
no ip domain lookup  
ip domain name gruppesechs.sitict.net  
\!  
\!  
\!  
login on-success log  
vtp version 1  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
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
\!  
\!  
flow exporter bank-flow-exporter-1  
 description export flows to tacacs1 .44  
 destination 172.16.0.44 vrf OOB  
 source Vlan99  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow exporter bank-flow-exporter-2  
 description export flows to tacacs2 .45  
 destination 172.16.0.45 vrf OOB  
 source Vlan99  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow monitor bank-flow-monitor  
 description Redundant Dual-Collector Traffic Visibility  
 exporter bank-flow-exporter-1  
 exporter bank-flow-exporter-2  
 cache timeout active 60  
 record bank-flow-record  
\!  
\!  
crypto pki trustpoint SLA-TrustPoint  
 enrollment pkcs12  
 revocation-check crl  
 hash sha256  
\!  
crypto pki trustpoint TP-self-signed-19850445  
 enrollment selfsigned  
 subject-name cn=IOS-Self-Signed-Certificate-19850445  
 revocation-check none  
 rsakeypair TP-self-signed-19850445  
 hash sha256  
\!  
\!  
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
\!  
\!  
license boot level network-advantage addon dna-advantage  
memory free low-watermark processor 130180  
\!  
diagnostic bootup level minimal  
\!  
spanning-tree mode rapid-pvst  
spanning-tree extend system-id  
spanning-tree vlan 23,46,69,99-100 priority 24576  
spanning-tree vlan 61,67,92 priority 28672  
\!  
\!  
\!  
enable secret 9 $9$I5KoJsCNAGL2PE\$7aF1doTVfTEx9SpaUVmmdpKz74d3JGsEzglx3Z9MssU  
\!  
username sitstudent secret 9 $9$itdWOl/ggzlSdk\$16Xx3KB4bI6jZsQy.EwFHwG3R0YA3QC6Uu2E8Dfq1M.  
username sitadmin secret 9 $9$pXXNvZxM9TmasE\$vUBprVQGGNc9MBPR.1LWet0sg9KgfDwGwe2h0Zm2Ynk  
username bigDonkBonkAdmin privilege 15 secret 9 $9$mEd2Tr8NqwU63.\$.h8geaiBpyw7MdLwtpwIANCCOzh8SZssis4cH//bp/U  
username donkBonkEng1 view eng-view secret 9 $9$mdJrLJn7no1jTU\$N3YGHXkawZ/wr1IDpaGt5NyplpPBBJk//US/50K3Mds  
username donkBonkEng2 view eng-view secret 9 $9$I.tgfeYQQfPVaE\$aUm5Yi3TxQSnG7TKzMvN5k7oO00739T9MA1PE518bjY  
\!  
redundancy  
 mode sso  
crypto engine compliance shield disable  
\!  
\!  
\!  
\!  
\!  
transceiver type all  
 monitoring  
\!  
track 1 interface Vlan100 line-protocol  
\!  
\!  
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
  description ICMP redirect, ICMP\_GEN and BROADCAST  
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
\!  
policy-map system-cpp-policy  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
interface Port-channel1  
 description Trunk\_to\_SW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel2  
 description Trunk\_to\_SW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 46,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel3  
 description Trunk\_to\_SW3  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 61,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel4  
 description Trunk\_to\_SW4  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 67,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel5  
 description Trunk\_to\_DSW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,46,61,67,69,92,99,100  
 switchport mode trunk  
 switchport nonegotiate  
\!  
interface GigabitEthernet0/0  
 vrf forwarding Mgmt-vrf  
 ip address 172.20.10.202 255.255.254.0  
 shutdown  
 negotiation auto  
\!  
interface GigabitEthernet1/0/1  
 description Uplink\_to\_SW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 1 mode active  
\!  
interface GigabitEthernet1/0/2  
 description Uplink\_to\_SW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 1 mode active  
\!  
interface GigabitEthernet1/0/3  
 description Uplink\_to\_SW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 46,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 2 mode active  
\!  
interface GigabitEthernet1/0/4  
 description Uplink\_to\_SW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 46,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 2 mode active  
\!  
interface GigabitEthernet1/0/5  
 description Uplink\_to\_SW3  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 61,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 3 mode active  
\!  
interface GigabitEthernet1/0/6  
 description Uplink\_to\_SW3  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 61,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 3 mode active  
\!  
interface GigabitEthernet1/0/7  
 description Uplink\_to\_SW4  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 67,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 4 mode active  
\!  
interface GigabitEthernet1/0/8  
 description Uplink\_to\_SW4  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 67,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 4 mode active  
\!  
interface GigabitEthernet1/0/9  
 description InterSwitch\_Link\_to\_DSW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,46,61,67,69,92,99,100  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 5 mode active  
\!  
interface GigabitEthernet1/0/10  
 description InterSwitch\_Link\_to\_DSW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,46,61,67,69,92,99,100  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 5 mode active  
\!  
interface GigabitEthernet1/0/11  
 description Inside\_Transit\_to\_ASA\_E1\_3\_and\_PA\_eth1\_6  
 switchport access vlan 100  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/12  
 description Inside\_Transit\_to\_ASA\_E1\_3\_and\_PA\_eth1\_6  
 switchport access vlan 100  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/13  
 description L2\_Bridge\_VLAN69\_Supermicro\_ASA\_PA  
 switchport access vlan 69  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/14  
 description L2\_Bridge\_VLAN69\_Supermicro\_ASA\_PA  
 switchport access vlan 69  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/15  
 description L2\_Bridge\_VLAN69\_Supermicro\_ASA\_PA  
 switchport access vlan 69  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/16  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/17  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/18  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/19  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/20  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/21  
 description OOB\_Mgmt\_Workstation  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/0/22  
 description OOB\_MGMT\_DEVICES\_R1\_PA\_PROXMOX  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/0/23  
 description OOB\_MGMT\_DEVICES\_R1\_PA\_PROXMOX  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/0/24  
 description OOB\_MGMT\_DEVICES\_R1\_PA\_PROXMOX  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/1/1  
\!  
interface GigabitEthernet1/1/2  
\!  
interface GigabitEthernet1/1/3  
\!  
interface GigabitEthernet1/1/4  
\!  
interface AppGigabitEthernet1/0/1  
\!  
interface Vlan1  
 no ip address  
\!  
interface Vlan23  
 description Staff\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.2 255.255.255.224  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_STAFF\_IN in  
 standby version 2  
 standby 23 ip 172.16.0.1  
 standby 23 priority 110  
 standby 23 preempt  
 standby 23 track 1 decrement 20  
\!  
interface Vlan46  
 description ATM\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.50 255.255.255.248  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_ATM\_IN in  
 standby version 2  
 standby 46 ip 172.16.0.49  
 standby 46 priority 110  
 standby 46 preempt  
 standby 46 track 1 decrement 20  
\!  
interface Vlan61  
 description HR\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.58 255.255.255.248  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_HR\_IN in  
 standby version 2  
 standby 61 ip 172.16.0.57  
 standby 61 priority 90  
 standby 61 preempt  
 standby 61 track 1 decrement 20  
\!  
interface Vlan67  
 description IT\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.66 255.255.255.248  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_IT\_IN in  
 standby version 2  
 standby 67 ip 172.16.0.65  
 standby 67 priority 90  
 standby 67 preempt  
 standby 67 track 1 decrement 20  
\!  
interface Vlan99  
 description OOB\_Management\_SVI  
 vrf forwarding OOB  
 ip address 172.16.0.35 255.255.255.240  
 no ip redirects  
 no ip proxy-arp  
\!  
interface Vlan100  
 description Inside\_Transit\_SVI\_to\_ASA\_and\_PA  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.98 255.255.255.248  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 10  
\!  
router ospf 1  
 router-id 3.3.3.3  
 auto-cost reference-bandwidth 10000  
 passive-interface default  
 no passive-interface Vlan100  
 network 172.16.0.0 0.0.0.31 area 0  
 network 172.16.0.48 0.0.0.7 area 0  
 network 172.16.0.56 0.0.0.7 area 0  
 network 172.16.0.64 0.0.0.7 area 0  
 network 172.16.0.96 0.0.0.7 area 0  
\!  
ip forward-protocol nd  
ip http server  
ip http authentication local  
ip http secure-server  
ip route 0.0.0.0 0.0.0.0 172.16.0.97  
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO\_NOT\_DELETE\_THIS\_ROUTE  
ip route vrf OOB 0.0.0.0 0.0.0.0 172.16.0.33  
ip ssh bulk-mode 131072  
ip ssh time-out 60  
\!  
\!  
ip access-list standard MGMT-ONLY  
 10 permit 172.16.0.44  
 20 permit 172.16.0.45  
 30 deny   any log  
\!  
ip access-list extended ACL\_ATM\_IN  
 5 permit udp 172.16.0.48 0.0.0.7 host 224.0.0.102 eq 1985  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.48 0.0.0.7 host 172.16.0.77 eq bootps  
 30 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.75  
 40 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.76  
 50 deny ip any any log  
ip access-list extended ACL\_HR\_IN  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.77 eq bootps  
 30 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.74 eq 445  
 40 permit icmp 172.16.0.56 0.0.0.7 host 172.16.0.74  
 50 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain  
 60 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain  
 70 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq www  
 80 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq 443  
 90 permit icmp 172.16.0.56 0.0.0.7 any  
 100 deny ip 172.16.0.56 0.0.0.7 172.16.0.0 0.0.0.127 log  
 110 permit ip 172.16.0.56 0.0.0.7 any  
ip access-list extended ACL\_IT\_IN  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.77 eq bootps  
 30 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain  
 40 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain  
 50 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq www  
 60 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq 443  
 70 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.84 eq 22  
 80 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.85 eq 22  
 90 permit icmp 172.16.0.64 0.0.0.7 any  
 100 deny ip 172.16.0.64 0.0.0.7 172.16.0.0 0.0.0.127 log  
 110 permit ip 172.16.0.64 0.0.0.7 any  
ip access-list extended ACL\_STAFF\_IN  
 5 permit udp 172.16.0.0 0.0.0.31 host 224.0.0.102 eq 1985  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.0 0.0.0.31 host 172.16.0.77 eq bootps  
 30 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.75  
 40 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.76  
 50 deny ip any any log  
\!  
logging host 172.16.0.44 vrf OOB  
logging host 172.16.0.45 vrf OOB  
\!  
tacacs server TACACS1  
 address ipv4 172.16.0.44  
 key 7 104C001E27180B2B3F01383D213A033431001019110507  
tacacs server TACACS2  
 address ipv4 172.16.0.45  
 key 7 0306520C2400386C7D0C0B1312003B0D17393C2B3A3767  
\!  
\!  
\!  
\!  
control-plane  
 service-policy input system-cpp-policy  
\!  
\!  
banner motd ^CC  
WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW1 IS PROHIBITED.  
^C  
parser view eng-view  
 secret 9 $9$GMFqXOyeYzVToE\$5Q1RPr811hWivvi9By12ZESzR1Ln1r1Yniw7mxUkXcU  
 commands interface include shutdown  
 commands interface include switchport  
 commands configure include interface  
 commands exec include traceroute  
 commands exec include ping  
 commands exec include configure terminal  
 commands exec include configure  
 commands exec include all show  
 commands exec include clear  
\!  
\!  
\!  
line con 0  
 login authentication CONSOLE  
 stopbits 1  
line vty 0 2  
 access-class MGMT-ONLY in vrf-also  
 transport input ssh  
line vty 3 4  
 transport input none  
line vty 5 15  
 transport input none  
line vty 16 31  
 transport input ssh  
\!  
ntp server vrf OOB 172.16.0.44 prefer  
ntp server vrf OOB 172.16.0.45  
\!  
\!  
\!  
\!  
\!  
\!  
end

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW2 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin  
Password:

DSW2\>en  
Password:  
Nov  6 13:24:41.128: %SEC\_LOGIN-5-LOGIN\_SUCCESS: Login Success \[user: bigDonkBonkAdmin\] \[Source: LOCAL\] \[localport: 0\] at 13:24:41 SGT Fri Nov 6 2026  
DSW2\#terminal length 0  
DSW2\#show run  
DSW2\#show running-config  
Building configuration...

Current configuration : 21418 bytes  
\!  
\! Last configuration change at 17:01:27 SGT Wed Nov 4 2026 by bigDonkBonkAdmin  
\!  
version 17.12  
service timestamps debug datetime msec  
service timestamps log datetime msec localtime  
service password-encryption  
platform punt-keepalive disable-kernel-core  
\!  
hostname DSW2  
\!  
\!  
vrf definition Mgmt-vrf  
 \!  
 address-family ipv4  
 exit-address-family  
 \!  
 address-family ipv6  
 exit-address-family  
\!  
vrf definition OOB  
 \!  
 address-family ipv4  
 exit-address-family  
\!  
aaa new-model  
\!  
\!  
aaa group server tacacs+ tac-group  
 server name TACACS1  
 server name TACACS2  
 ip vrf forwarding OOB  
 ip tacacs source-interface Vlan99  
\!  
aaa authentication login default group tac-group local  
aaa authentication login CONSOLE group tac-group local  
aaa authorization config-commands  
aaa authorization exec default group tac-group local  
aaa authorization commands 15 default group tac-group local  
aaa accounting exec default start-stop group tac-group  
aaa accounting commands 15 default stop-only group tac-group  
\!  
\!  
aaa session-id common  
clock timezone SGT 8 0  
switch 1 provision c9300l-24t-4g  
\!  
\!  
\!  
\!  
ip routing  
\!  
\!  
\!  
\!  
\!  
no ip domain lookup  
ip domain name gruppesechs.sitict.net  
\!  
\!  
\!  
login on-success log  
vtp version 1  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
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
\!  
\!  
flow exporter bank-flow-exporter-1  
 description export flows to tacacs1 .44  
 destination 172.16.0.44 vrf OOB  
 source Vlan99  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow exporter bank-flow-exporter-2  
 description export flows to tacacs2 .45  
 destination 172.16.0.45 vrf OOB  
 source Vlan99  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow monitor bank-flow-monitor  
 description Redundant Dual-Collector Traffic Visibility  
 exporter bank-flow-exporter-1  
 exporter bank-flow-exporter-2  
 cache timeout active 60  
 record bank-flow-record  
\!  
\!  
crypto pki trustpoint SLA-TrustPoint  
 enrollment pkcs12  
 revocation-check crl  
 hash sha256  
\!  
crypto pki trustpoint TP-self-signed-1171067940  
 enrollment selfsigned  
 subject-name cn=IOS-Self-Signed-Certificate-1171067940  
 revocation-check none  
 rsakeypair TP-self-signed-1171067940  
 hash sha256  
\!  
\!  
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
\!  
\!  
license boot level network-advantage addon dna-advantage  
memory free low-watermark processor 130180  
\!  
diagnostic bootup level minimal  
\!  
spanning-tree mode rapid-pvst  
spanning-tree extend system-id  
spanning-tree vlan 23,46,69,99-100 priority 28672  
spanning-tree vlan 61,67,92 priority 24576  
\!  
\!  
\!  
enable secret 9 $9$ui3lRWZ18rc2gk\$TyVkEzhlcvLv.QD4tYgYBL6eA/1RMK8sB22TjZ2MTmM  
\!  
username sitstudent secret 9 $9$zf4pFjL6ptS1b.\$hJosjVwTFzCZfBS8me4q3MK4hhz5L9xAzomD4WeFIKU  
username sitadmin secret 9 $9$bBFADZPSjRbfFE\$HaH3FwJ/aqiLixuJ9zSqukF9qVs5Cf9xEQzVM7NE3hg  
username bigDonkBonkAdmin privilege 15 secret 9 $9$Zv0SY6K/Prj9.k\$NAuBqDewx1TxikHFTzecewvzGB4rAhr./HtLMfqrwwk  
username donkBonkEng1 view eng-view secret 9 $9$PAqP2QbbkXgjNU\$Qe2g1OVUAcaJ/1mrhrlI4EAMFK0O2Aij9WqIOZIXLPw  
username donkBonkEng2 view eng-view secret 9 $9$sQdLgfd2odSRak\$lVLKVijhbDv47Rhum/WHMijkc13eregl8/nwkxrT8Xo  
\!  
redundancy  
 mode sso  
crypto engine compliance shield disable  
\!  
\!  
\!  
\!  
\!  
transceiver type all  
 monitoring  
\!  
track 1 interface Vlan100 line-protocol  
\!  
\!  
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
  description ICMP redirect, ICMP\_GEN and BROADCAST  
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
\!  
policy-map system-cpp-policy  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
interface Port-channel1  
 description Trunk\_to\_SW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel2  
 description Trunk\_to\_SW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 46,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel3  
 description Trunk\_to\_SW3  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 61,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel4  
 description Trunk\_to\_SW4  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 67,99  
 switchport mode trunk  
 switchport nonegotiate  
 spanning-tree guard root  
\!  
interface Port-channel5  
 description Trunk\_to\_DSW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,46,61,67,69,92,99,100  
 switchport mode trunk  
 switchport nonegotiate  
\!  
interface GigabitEthernet0/0  
 vrf forwarding Mgmt-vrf  
 ip dhcp client client-id ascii cisco-70da.48fb.2500-Gi0/0  
 ip address 172.20.10.203 255.255.254.0  
 shutdown  
 negotiation auto  
\!  
interface GigabitEthernet1/0/1  
 description Uplink\_to\_SW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 1 mode active  
\!  
interface GigabitEthernet1/0/2  
 description Uplink\_to\_SW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 1 mode active  
\!  
interface GigabitEthernet1/0/3  
 description Uplink\_to\_SW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 46,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 2 mode active  
\!  
interface GigabitEthernet1/0/4  
 description Uplink\_to\_SW2  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 46,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 2 mode active  
\!  
interface GigabitEthernet1/0/5  
 description Uplink\_to\_SW3  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 61,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 3 mode active  
\!  
interface GigabitEthernet1/0/6  
 description Uplink\_to\_SW3  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 61,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 3 mode active  
\!  
interface GigabitEthernet1/0/7  
 description Uplink\_to\_SW4  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 67,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 4 mode active  
\!  
interface GigabitEthernet1/0/8  
 description Uplink\_to\_SW4  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 67,99  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 4 mode active  
\!  
interface GigabitEthernet1/0/9  
 description InterSwitch\_Link\_to\_DSW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,46,61,67,69,92,99,100  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 5 mode active  
\!  
interface GigabitEthernet1/0/10  
 description InterSwitch\_Link\_to\_DSW1  
 switchport trunk native vlan 821  
 switchport trunk allowed vlan 23,46,61,67,69,92,99,100  
 switchport mode trunk  
 switchport nonegotiate  
 channel-group 5 mode active  
\!  
interface GigabitEthernet1/0/11  
 description Inside\_Transit\_to\_ASA\_E1\_4\_and\_PA\_eth1\_7  
 switchport access vlan 100  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/12  
 description Inside\_Transit\_to\_ASA\_E1\_4\_and\_PA\_eth1\_7  
 switchport access vlan 100  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/13  
 description L2\_Bridge\_VLAN92\_Supermicro\_PA\_ASA  
 switchport access vlan 92  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/14  
 description L2\_Bridge\_VLAN92\_Supermicro\_PA\_ASA  
 switchport access vlan 92  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/15  
 description L2\_Bridge\_VLAN92\_Supermicro\_PA\_ASA  
 switchport access vlan 92  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
\!  
interface GigabitEthernet1/0/16  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/17  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/18  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/19  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/20  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/21  
 description UNUSED\_PORTS  
 switchport access vlan 727  
 switchport mode access  
 switchport nonegotiate  
 shutdown  
\!  
interface GigabitEthernet1/0/22  
 description OOB\_MGMT\_DEVICES\_R2\_ASA\_PROXMOX  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/0/23  
 description OOB\_MGMT\_DEVICES\_R2\_ASA\_PROXMOX  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/0/24  
 description OOB\_MGMT\_DEVICES\_R2\_ASA\_PROXMOX  
 switchport access vlan 99  
 switchport mode access  
 switchport nonegotiate  
 spanning-tree portfast  
 spanning-tree bpduguard enable  
\!  
interface GigabitEthernet1/1/1  
\!  
interface GigabitEthernet1/1/2  
\!  
interface GigabitEthernet1/1/3  
\!  
interface GigabitEthernet1/1/4  
\!  
interface AppGigabitEthernet1/0/1  
\!  
interface Vlan1  
 no ip address  
\!  
interface Vlan23  
 description Staff\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.3 255.255.255.224  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_STAFF\_IN in  
 standby version 2  
 standby 23 ip 172.16.0.1  
 standby 23 priority 90  
 standby 23 preempt  
 standby 23 track 1 decrement 20  
\!  
interface Vlan46  
 description ATM\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.51 255.255.255.248  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_ATM\_IN in  
 standby version 2  
 standby 46 ip 172.16.0.49  
 standby 46 priority 90  
 standby 46 preempt  
 standby 46 track 1 decrement 20  
\!  
interface Vlan61  
 description HR\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.59 255.255.255.248  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_HR\_IN in  
 standby version 2  
 standby 61 ip 172.16.0.57  
 standby 61 priority 110  
 standby 61 preempt  
 standby 61 track 1 decrement 20  
\!  
interface Vlan67  
 description IT\_VLAN\_Gateway  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.67 255.255.255.248  
 ip helper-address 172.16.0.77  
 no ip redirects  
 no ip proxy-arp  
 ip access-group ACL\_IT\_IN in  
 standby version 2  
 standby 67 ip 172.16.0.65  
 standby 67 priority 110  
 standby 67 preempt  
 standby 67 track 1 decrement 20  
\!  
interface Vlan99  
 description OOB\_Management\_SVI  
 vrf forwarding OOB  
 ip address 172.16.0.36 255.255.255.240  
 no ip redirects  
 no ip proxy-arp  
\!  
interface Vlan100  
 description Inside\_Transit\_SVI\_to\_ASA\_and\_PA  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.99 255.255.255.248  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 10  
\!  
router ospf 1  
 router-id 4.4.4.4  
 auto-cost reference-bandwidth 10000  
 passive-interface default  
 no passive-interface Vlan100  
 network 172.16.0.0 0.0.0.31 area 0  
 network 172.16.0.48 0.0.0.7 area 0  
 network 172.16.0.56 0.0.0.7 area 0  
 network 172.16.0.64 0.0.0.7 area 0  
 network 172.16.0.96 0.0.0.7 area 0  
\!  
ip forward-protocol nd  
ip http server  
ip http authentication local  
ip http secure-server  
ip http client source-interface GigabitEthernet0/0  
ip route 0.0.0.0 0.0.0.0 172.16.0.97  
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO\_NOT\_DELETE\_THIS\_ROUTE  
ip route vrf OOB 0.0.0.0 0.0.0.0 172.16.0.33  
ip ssh bulk-mode 131072  
ip ssh time-out 60  
\!  
\!  
ip access-list standard MGMT-ONLY  
 10 permit 172.16.0.44  
 20 permit 172.16.0.45  
 30 deny   any log  
\!  
ip access-list extended ACL\_ATM\_IN  
 5 permit udp 172.16.0.48 0.0.0.7 host 224.0.0.102 eq 1985  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.48 0.0.0.7 host 172.16.0.77 eq bootps  
 30 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.75  
 40 permit ip 172.16.0.48 0.0.0.7 host 172.16.0.76  
 50 deny ip any any log  
ip access-list extended ACL\_HR\_IN  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.77 eq bootps  
 30 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.74 eq 445  
 40 permit icmp 172.16.0.56 0.0.0.7 host 172.16.0.74  
 50 permit udp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain  
 60 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.83 eq domain  
 70 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq www  
 80 permit tcp 172.16.0.56 0.0.0.7 host 172.16.0.82 eq 443  
 90 permit icmp 172.16.0.56 0.0.0.7 any  
 100 deny ip 172.16.0.56 0.0.0.7 172.16.0.0 0.0.0.127 log  
 110 permit ip 172.16.0.56 0.0.0.7 any  
ip access-list extended ACL\_IT\_IN  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.77 eq bootps  
 30 permit udp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain  
 40 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.83 eq domain  
 50 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq www  
 60 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.82 eq 443  
 70 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.84 eq 22  
 80 permit tcp 172.16.0.64 0.0.0.7 host 172.16.0.85 eq 22  
 90 permit icmp 172.16.0.64 0.0.0.7 any  
 100 deny ip 172.16.0.64 0.0.0.7 172.16.0.0 0.0.0.127 log  
 110 permit ip 172.16.0.64 0.0.0.7 any  
ip access-list extended ACL\_STAFF\_IN  
 5 permit udp 172.16.0.0 0.0.0.31 host 224.0.0.102 eq 1985  
 6 permit udp host 172.16.0.77 eq bootps any eq bootps  
 10 permit udp any host 255.255.255.255 eq bootps  
 20 permit udp 172.16.0.0 0.0.0.31 host 172.16.0.77 eq bootps  
 30 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.75  
 40 permit ip 172.16.0.0 0.0.0.31 host 172.16.0.76  
 50 deny ip any any log  
\!  
logging host 172.16.0.44 vrf OOB  
logging host 172.16.0.45 vrf OOB  
\!  
tacacs server TACACS1  
 address ipv4 172.16.0.44  
 key 7 110B1002351D122C372F39322D2105230014010C135208  
tacacs server TACACS2  
 address ipv4 172.16.0.45  
 key 7 060406266E4110393612001D09161A2A373B243A301755  
\!  
\!  
\!  
\!  
control-plane  
 service-policy input system-cpp-policy  
\!  
\!  
banner motd ^CC  
WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS DSW2 IS PROHIBITED.  
^C  
parser view eng-view  
 secret 9 $9$iLSSJ2gvntiSIU\$M4igyhZVnWS6WfCziZQYfG2VKeXOXcKKpA5vnrnS5QY  
 commands interface include shutdown  
 commands interface include switchport  
 commands configure include interface  
 commands exec include traceroute  
 commands exec include ping  
 commands exec include configure terminal  
 commands exec include configure  
 commands exec include all show  
 commands exec include clear  
\!  
\!  
\!  
line con 0  
 login authentication CONSOLE  
 stopbits 1  
line vty 0 2  
 access-class MGMT-ONLY in vrf-also  
 transport input ssh  
line vty 3 4  
 transport input none  
line vty 5 15  
 transport input none  
line vty 16 31  
 transport input ssh  
\!  
ntp server vrf OOB 172.16.0.44 prefer  
ntp server vrf OOB 172.16.0.45  
\!  
\!  
\!  
\!  
\!  
\!  
end

aaa session-id common ACCESS TO GRUPPESECHS IS PROHIBITED.  
clock timezone SGT 8 0  
\!sername: bigDonkBonkAdmin  
no ip domain lookup  
ip domain name gruppesechs.sitict.net  
\!1\>en  
\!ov  6 13:27:08.131: %SEC\_LOGIN-5-LOGIN\_SUCCESS: Login Success \[user: bigDonkBonkAdmin\] \[Source: LOCAL\] \[localport: 0  
\! at 13:27:08 SGT Fri Nov 6 2026  
\!assword:  
\!1\#  
\!1\#clear  
\! Type "clear ?" for a list of subcommands  
\!1\#  
\!  
login on-success log  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
subscriber templating  
vtp version 1  
multilink bundle-name authenticated  
\!  
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
\!  
\!  
flow exporter bank-flow-exporter-1  
 description export flows to tacacs1 .44  
 destination 172.16.0.44 vrf OOB  
 source GigabitEthernet0/1/0  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow exporter bank-flow-exporter-2  
 description export flows to tacacs2 .45  
 destination 172.16.0.45 vrf OOB  
 source GigabitEthernet0/1/0  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow monitor bank-flow-monitor  
 description Redundant Dual-Collector Traffic Visibility  
 exporter bank-flow-exporter-1  
 exporter bank-flow-exporter-2  
 cache timeout active 60  
 record bank-flow-record  
\!  
\!  
pae  
\!  
\!  
crypto pki trustpoint SLA-TrustPoint  
 enrollment pkcs12  
 revocation-check crl  
 hash sha256  
\!  
crypto pki trustpoint TP-self-signed-4263099988  
 enrollment selfsigned  
 subject-name cn=IOS-Self-Signed-Certificate-4263099988  
 revocation-check none  
 rsakeypair TP-self-signed-4263099988  
 hash sha256  
\!  
\!  
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
  69666963 6174652D 34323633 30393939 3838301E 170D3236 31303237 30353034  
  34365A17 0D333631 30323630 35303434 365A3031 312F302D 06035504 030C2649  
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
  300D0609 2A864886 F70D0101 0B050003 82010100 90593BAA 6D977EB5 E5015EE0  
  F56013EE 98BDF469 DEFE50A6 8B043F48 608F926C 117D1E32 1F2090C9 6FFE09AA  
  0BAB300F CB3508A0 02CB2EF8 3666BA28 567471B2 FFDADC3A 58D4592D DDBA931B  
  6D7E0908 B89F0E8D C03117B5 C5B27C76 D08A7A0C 4AFA0BDA 44E7A116 5A527D5D  
  91AA7358 E1E0FD22 EE837236 C59FB755 E7985BD3 9C94F350 31B8438F 7C449B3A  
  908B31B3 4A07BBE2 B7DDE50E 87D36ED0 0DB0AB27 F6ED8DCB 119C987B 016A5641  
  D63C6AA1 EC68E5DE 12FECDD7 4385D729 D6CF13B5 CA20C45D D94513CB E27DD4A7  
  D1C58919 540DB93A EE0CC717 118ECC4F 4402412B F3495091 183E88AB FB3DDED3  
  BAFB14AB E5D87995 B29A13D6 80863161 B2A9D537  
        quit  
\!  
\!  
diagnostic bootup level minimal  
\!  
license udi pid C8200L-1N-4T sn FGL2803L1L4  
memory free low-watermark processor 61453  
\!  
spanning-tree extend system-id  
\!  
\!  
enable secret 9 $9$1nrmYP7P5myij.\$gbqMpIY7KxWw1sJhpoR2zDzFC5O2g1j9yn0AUI1aoBA  
\!  
username sitstudent secret 9 $9$tsQR6TpaVC7s5U\$6el8g9/3KuQy7K5IWWWTZDIsHb5G/IVch1QuzDCLV1M  
username sitadmin secret 9 $9$4k9PaKIZWKXEr.\$0cL5lKDmWrG1nZVovVi6MDQMu6BwgWAgqXvOXvXxYmc  
username bigDonkBonkAdmin privilege 15 secret 9 $9$4mioq4v/Fw5pz.\$zTHTW5crc2Na7CXUDHoyqKllm0UuFMLOyhLhPf1c46Q  
username donkBonkEng1 view eng-view secret 9 $9$Vnah75d0txSRy.\$IRCGz4u8KblHNwsu8t5LYRSqdp282x0WLJC6vvpVruo  
username donkBonkEng2 view eng-view secret 9 $9$cVPcAkiS/QSR5.\$CdrITp/2E0VYzQZkvISWxSkFAOpbW5Do1wvgdc2SYTw  
\!  
redundancy  
 mode none  
\!  
\!  
\!  
\!  
track 1 ip sla 1 reachability  
\!  
track 2 ip sla 2 reachability  
\!  
\!  
\!  
\!  
\!  
interface GigabitEthernet0/0/0  
 description WAN\_Link\_to\_ISP1  
 ip flow monitor bank-flow-monitor input  
 ip address 172.17.9.93 255.255.255.252  
 no ip redirects  
 no ip proxy-arp  
 ip access-group WAN-IN in  
 negotiation auto  
\!  
interface GigabitEthernet0/0/1  
 description Primary\_Transit\_to\_PA440\_eth1\_5  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.90 255.255.255.248  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 10  
 negotiation auto  
\!  
interface GigabitEthernet0/0/2  
 no ip address  
 negotiation auto  
\!  
interface GigabitEthernet0/0/3  
 no ip address  
 negotiation auto  
\!  
interface GigabitEthernet0/1/0  
 description OOB\_Management\_Gateway\_VLAN99  
 vrf forwarding OOB  
 ip address 172.16.0.33 255.255.255.240  
 no ip redirects  
 no ip proxy-arp  
 negotiation auto  
\!  
interface GigabitEthernet0/1/1  
 description Direct\_InterRouter\_Link\_to\_R2  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.109 255.255.255.252  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 5  
 negotiation auto  
\!  
router ospf 1  
 router-id 1.1.1.1  
 auto-cost reference-bandwidth 10000  
 passive-interface default  
 no passive-interface GigabitEthernet0/0/1  
 no passive-interface GigabitEthernet0/1/1  
 network 172.16.0.88 0.0.0.7 area 0  
 network 172.16.0.108 0.0.0.3 area 0  
 default-information originate always metric 10 metric-type 1  
\!  
ip forward-protocol nd  
no ip http server  
ip http secure-server  
\!  
ip route 0.0.0.0 0.0.0.0 172.17.9.94 track 1  
ip route 203.149.210.184 255.255.255.248 172.16.0.92 track 2  
ip route 0.0.0.0 0.0.0.0 172.17.9.94 250  
ip route 203.149.210.184 255.255.255.248 172.16.0.110 250  
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO\_NOT\_DELETE\_THIS\_ROUTE  
ip ssh bulk-mode 131072  
ip ssh time-out 60  
\!  
ip access-list standard MGMT-ONLY  
 10 permit 172.16.0.44  
 20 permit 172.16.0.45  
 30 deny   any log  
\!  
ip access-list extended WAN-IN  
 10 deny tcp any host 172.17.9.94 eq 22 log  
 15 permit ospf host 172.17.9.94 any  
 20 deny tcp any host 172.17.9.94 eq telnet log  
 25 permit icmp any any echo-reply  
 26 permit icmp any any unreachable  
 27 permit icmp any any time-exceeded  
 30 permit icmp host 172.17.9.93 host 172.17.9.94 echo-reply  
 40 permit icmp any 203.149.210.184 0.0.0.7  
 50 permit ip any 203.149.210.184 0.0.0.7  
 60 permit tcp any host 172.17.9.94 established  
 70 deny ip any any log  
\!  
ip sla 1  
 icmp-echo 172.17.9.93 source-interface GigabitEthernet0/0/0  
  frequency 5  
ip sla schedule 1 life forever start-time now  
ip sla 2  
 icmp-echo 172.16.0.92  
  frequency 5  
ip sla schedule 2 life forever start-time now  
logging host 172.16.0.44 vrf OOB  
logging host 172.16.0.45 vrf OOB  
\!  
tacacs server TACACS1  
 address ipv4 172.16.0.44  
 key 7 00061A01265412263C245E580C0B351601181B0B382F75  
tacacs server TACACS2  
 address ipv4 172.16.0.45  
 key 7 14151B0C2E0B330B172D212327013717101241564A5706  
\!  
\!  
\!  
\!  
control-plane  
\!  
banner motd ^CC  
WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R1 IS PROHIBITED.  
^C  
parser view eng-view  
 secret 9 $9$6U/CdQOdMD7oEE\$Zq8eJ5CrYqADEY2wPMad8gMX07Yn.bcWFfOkVEzxGrM  
 commands interface include shutdown  
 commands configure include interface  
 commands exec include traceroute  
 commands exec include ping  
 commands exec include configure terminal  
 commands exec include configure  
 commands exec include all show  
 commands exec include clear  
\!  
\!  
line con 0  
 login authentication CONSOLE  
 stopbits 1  
line aux 0  
line vty 0 2  
 access-class MGMT-ONLY in vrf-also  
 transport input ssh  
line vty 3 4  
 transport input none  
line vty 5 15  
 transport input none  
line vty 16 31  
 transport input ssh  
\!  
ntp server vrf OOB 172.16.0.44 prefer  
ntp server vrf OOB 172.16.0.45  
\!  
\!  
\!  
\!  
\!  
\!  
end

R1\#

C  
WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R2 IS PROHIBITED.

WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS IS PROHIBITED.

Username: bigDonkBonkAdmin  
Password:

R2\>en  
Nov  6 13:30:51.172: %SEC\_LOGIN-5-LOGIN\_SUCCESS: Login Success \[user: bigDonkBon                                                                                                                                                             kAdmin\] \[Source: LOCAL\] \[localport: 0\] at 13:30:51 SGT Fri Nov 6 2026  
Password:  
R2\#terminal length 0  
R2\#show run  
R2\#show running-config  
Building configuration...

Current configuration : 10479 bytes  
\!  
\! Last configuration change at 10:30:10 SGT Wed Nov 4 2026 by bigDonkBonkAdmin  
\!  
version 17.12  
service timestamps debug datetime msec  
service timestamps log datetime msec localtime  
service password-encryption  
platform qfp utilization monitor load 80  
platform punt-keepalive disable-kernel-core  
\!  
hostname R2  
\!  
boot-start-marker  
boot-end-marker  
\!  
\!  
vrf definition Mgmt-vrf  
 rd 1:1  
 \!  
 address-family ipv4  
  route-target export 1:1  
  route-target import 1:1  
 exit-address-family  
\!  
vrf definition OOB  
 \!  
 address-family ipv4  
 exit-address-family  
\!  
aaa new-model  
\!  
\!  
aaa group server tacacs+ tac-group  
 server name TACACS1  
 server name TACACS2  
 ip vrf forwarding OOB  
 ip tacacs source-interface GigabitEthernet0/1/0  
\!  
aaa authentication login default group tac-group local  
aaa authentication login CONSOLE group tac-group local  
aaa authorization config-commands  
aaa authorization exec default group tac-group local  
aaa authorization commands 15 default group tac-group local  
aaa accounting exec default start-stop group tac-group  
aaa accounting commands 15 default stop-only group tac-group  
\!  
\!  
aaa session-id common  
clock timezone SGT 8 0  
\!  
no ip domain lookup  
ip domain name gruppesechs.sitict.net  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
login on-success log  
\!  
\!  
\!  
\!  
\!  
\!  
\!  
subscriber templating  
vtp version 1  
multilink bundle-name authenticated  
\!  
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
\!  
\!  
flow exporter bank-flow-exporter-1  
 description export flows to tacacs1 .44  
 destination 172.16.0.44 vrf OOB  
 source GigabitEthernet0/1/0  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow exporter bank-flow-exporter-2  
 description export flows to tacacs2 .45  
 destination 172.16.0.45 vrf OOB  
 source GigabitEthernet0/1/0  
 transport udp 2055  
 template data timeout 60  
\!  
\!  
flow monitor bank-flow-monitor  
 description Redundant Dual-Collector Traffic Visibility  
 exporter bank-flow-exporter-1  
 exporter bank-flow-exporter-2  
 cache timeout active 60  
 record bank-flow-record  
\!  
\!  
pae  
\!  
\!  
crypto pki trustpoint TP-self-signed-3634272404  
 enrollment selfsigned  
 subject-name cn=IOS-Self-Signed-Certificate-3634272404  
 revocation-check none  
 rsakeypair TP-self-signed-3634272404  
 hash sha256  
\!  
crypto pki trustpoint SLA-TrustPoint  
 enrollment pkcs12  
 revocation-check crl  
 hash sha256  
\!  
\!  
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
\!  
\!  
diagnostic bootup level minimal  
\!  
license udi pid C8200L-1N-4T sn FGL2803L1LW  
memory free low-watermark processor 61453  
\!  
spanning-tree extend system-id  
\!  
\!  
enable secret 9 $9$.mEbOD.p/j2Nrk\$rLB0TGbqqBv1qqo9HwE0LqwuhthnkzzJCM.jKaXsIew  
\!  
username sitstudent secret 9 $9$cgiRaZ0wKiAIt.\$nT5uu.c96AFFmFEyYEFWNtQBWO0p1wQM9s59.IX/lzY  
username sitadmin secret 9 $9$MwzS1uEPwNtB5U\$Bek6yHDXhs2fEk4afqwwdehbRtuqs9pmOqTzweL4tEg  
username bigDonkBonkAdmin privilege 15 secret 9 $9$5SmJmOEPNRZt..\$njumx3d9/EMAOewSk3tR3SvY8txygWx9TtYWx4HXFcQ  
username donkBonkEng1 view eng-view secret 9 $9$DHCunJPH0xZ7F.\$tpELNiaPavDuoxVBp1paN/kO0EWnombAoTBz29UvHC2  
username donkBonkEng2 view eng-view secret 9 $9$gUjB45As6o22x.\$nfr7OGqIi/874MR4od7UcT6PKq1mkC7c4x.UhQa/ohw  
\!  
redundancy  
 mode none  
\!  
\!  
\!  
\!  
track 2 ip sla 2 reachability  
\!  
\!  
\!  
\!  
\!  
interface GigabitEthernet0/0/0  
 description PTSO\_Bypass\_Link\_to\_ASA\_E1\_2  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.113 255.255.255.252  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 500  
 negotiation auto  
\!  
interface GigabitEthernet0/0/1  
 description Secondary\_Transit\_to\_PA440\_eth1\_2  
 ip flow monitor bank-flow-monitor input  
 ip address 172.16.0.91 255.255.255.248  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 20  
 negotiation auto  
\!  
interface GigabitEthernet0/0/2  
 no ip address  
 negotiation auto  
\!  
interface GigabitEthernet0/0/3  
 no ip address  
 negotiation auto  
\!  
interface GigabitEthernet0/1/0  
 description OOB\_Management\_Interface\_VLAN99  
 vrf forwarding OOB  
 ip address 172.16.0.34 255.255.255.240  
 no ip redirects  
 no ip proxy-arp  
 negotiation auto  
\!  
interface GigabitEthernet0/1/1  
 description Direct\_InterRouter\_Link\_to\_R1  
 ip flow monitor bank-flow-monitor input  
 ip dhcp client client-id ascii cisco-60b9.c03a.4c89-Gi0/1/1  
 ip address 172.16.0.110 255.255.255.252  
 no ip redirects  
 no ip proxy-arp  
 ip ospf cost 5  
 negotiation auto  
\!  
router ospf 1  
 router-id 2.2.2.2  
 auto-cost reference-bandwidth 10000  
 passive-interface default  
 no passive-interface GigabitEthernet0/0/0  
 no passive-interface GigabitEthernet0/0/1  
 no passive-interface GigabitEthernet0/1/1  
 network 172.16.0.88 0.0.0.7 area 0  
 network 172.16.0.108 0.0.0.3 area 0  
 network 172.16.0.112 0.0.0.3 area 0  
\!  
ip forward-protocol nd  
ip http server  
ip http authentication local  
ip http secure-server  
ip http client source-interface GigabitEthernet0/1/1  
\!  
ip route 203.149.210.184 255.255.255.248 172.16.0.92 track 2  
ip route 203.149.210.184 255.255.255.248 172.16.0.114 250  
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 172.20.11.254 name DO\_NOT\_DELETE\_THIS\_ROUTE  
ip ssh bulk-mode 131072  
ip ssh time-out 60  
\!  
ip access-list standard MGMT-ONLY  
 10 permit 172.16.0.44  
 20 permit 172.16.0.45  
 30 deny   any log  
\!  
\!  
ip sla 2  
 icmp-echo 172.16.0.92  
  frequency 5  
ip sla schedule 2 life forever start-time now  
logging host 172.16.0.44 vrf OOB  
logging host 172.16.0.45 vrf OOB  
\!  
tacacs server TACACS1  
 address ipv4 172.16.0.44  
 key 7 110B1002351D122C372F39322D2105230014010C135208  
tacacs server TACACS2  
 address ipv4 172.16.0.45  
 key 7 02040D5C290916017F4B1B0F0005220A1F173D24362C61  
\!  
\!  
\!  
\!  
control-plane  
\!  
banner motd ^CC  
WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS R2 IS PROHIBITED.  
^C  
parser view eng-view  
 secret 9 $9$9GYqDmtqFVTVxE\$7Y4nWQTxvNxVgCCY/5tZEPjrdPPK2zdqpMS8eSMhxSo  
 commands interface include shutdown  
 commands configure include interface  
 commands exec include traceroute  
 commands exec include ping  
 commands exec include configure terminal  
 commands exec include configure  
 commands exec include all show  
 commands exec include clear  
\!  
\!  
line con 0  
 login authentication CONSOLE  
 stopbits 1  
line aux 0  
line vty 0 2  
 access-class MGMT-ONLY in vrf-also  
 transport input ssh  
line vty 3 4  
 transport input none  
line vty 5 15  
 transport input none  
line vty 16 31  
 transport input ssh  
\!  
ntp server vrf OOB 172.16.0.44 prefer  
ntp server vrf OOB 172.16.0.45  
\!  
\!  
\!  
\!  
\!  
\!  
end

ASA-1010\# terminal pager 0  
ASA-1010\# show run  
ASA-1010\# show running-config  
: Saved

:  
: Serial Number: JAD2801087C  
: Hardware:   FPR-1010, 7197 MB RAM, CPU Atom C3000 series 2200 MHz, 1 CPU (4 cores)  
:  
ASA Version 9.20(2)2  
\!  
hostname ASA-1010  
domain-name gruppesechs.sitict.net  
enable password \*\*\*\*\* pbkdf2  
service-module 1 keepalive-timeout 4  
service-module 1 keepalive-counter 6  
names  
no mac-address auto

\!  
interface Vlan1  
 shutdown  
 no nameif  
 no security-level  
 no ip address  
\!  
interface Vlan69  
 description Primary\_Internal\_Servers\_Gateway\_SVI  
 nameif server  
 security-level 60  
 ip address 172.16.0.73 255.255.255.248  
\!  
interface Vlan92  
 description Backup\_DMZ\_Servers\_Gateway\_SVI  
 shutdown  
 nameif dmz  
 security-level 50  
 ip address 172.16.0.86 255.255.255.248  
\!  
interface Ethernet1/1  
 description Primary\_Serial\_Link\_to\_PA440\_eth1\_3  
 no switchport  
 speed 1000  
 duplex full  
 nameif outside  
 security-level 0  
 ip address 172.16.0.106 255.255.255.252  
 ospf cost 10  
\!  
interface Ethernet1/2  
 description PTSO\_Bypass\_Link\_to\_R2\_G0\_0\_0  
 no switchport  
 speed 1000  
 duplex full  
 nameif wan-backup  
 security-level 0  
 ip address 172.16.0.114 255.255.255.252  
 ospf cost 500  
\!  
interface Ethernet1/3  
 description Inside\_Transit\_to\_DSW1\_G1\_0\_11  
 no switchport  
 speed 1000  
 duplex full  
 nameif inside  
 security-level 100  
 ip address 172.16.0.97 255.255.255.248  
 ospf cost 10  
\!  
interface Ethernet1/4  
 description Spare\_Link\_to\_DSW2\_G1\_0\_11\_SPARE  
 no switchport  
 shutdown  
 no nameif  
 no security-level  
 no ip address  
\!  
interface Ethernet1/5  
 description UNUSED\_PORT  
 switchport  
 shutdown  
 no security-level  
\!  
interface Ethernet1/6  
 description Link\_to\_DSW1\_G1\_0\_14\_VLAN69\_Bridge  
 switchport  
 switchport access vlan 69  
 no security-level  
\!  
interface Ethernet1/7  
 description Link\_to\_DSW2\_G1\_0\_15\_VLAN92\_Bridge  
 switchport  
 switchport access vlan 92  
 power inline auto  
 no security-level  
\!  
interface Ethernet1/8  
 description UNUSED\_PORT  
 switchport  
 power inline auto  
 shutdown  
 no security-level  
\!  
interface Management1/1  
 description OOB\_Management\_VLAN99  
 management-only  
 nameif management  
 security-level 100  
 ip address 172.16.0.42 255.255.255.240  
\!  
\!  
time-range TXN-WINDOW  
 periodic weekdays 1:00 to 3:00  
\!  
banner motd WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS ASA IS PROHIBITED.  
ftp mode passive  
clock timezone Singapore  
dns domain-lookup outside  
dns server-group DefaultDNS  
 name-server 208.67.222.222 outside  
 name-server 208.67.220.220 outside  
 domain-name gruppesechs.sitict.net  
no object-group-search access-control  
object network obj\_any  
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
object network PUB-PAT  
 host 203.149.210.185  
object network WEB-SVR-NAT  
 host 172.16.0.82  
object network DNS-SVR-NAT  
 host 172.16.0.83  
object network JUMP-SSH-NAT  
 host 172.16.0.84  
object-group network STAFF-ATM  
 network-object object STAFF  
 network-object object ATM  
object-group network HR-IT  
 network-object object HR  
 network-object object IT  
object-group network DSWs  
 network-object host 172.16.0.98  
 network-object host 172.16.0.99  
object-group network JUMP-SVRs  
 network-object object JUMP-SVR  
 network-object object JUMP2-SVR  
object-group network PAT-USERS  
 network-object object HR  
 network-object object IT  
 network-object object TXN-SVR  
 network-object object JUMP-SVR  
 network-object object JUMP2-SVR  
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
 network-object host 172.16.0.99  
access-group OUTSIDE\_IN in interface wan-backup  
access-group OUTSIDE\_IN in interface outside  
access-group INSIDE\_IN in interface inside  
access-group SERVER\_IN in interface server  
access-group DMZ\_IN in interface dmz  
access-list INSIDE\_IN extended permit ospf any any  
access-list INSIDE\_IN extended permit udp object-group DSW-SVIS object DHCP-SVR eq bootps  
access-list INSIDE\_IN extended permit udp object-group DSWs object DHCP-SVR eq bootps  
access-list INSIDE\_IN extended permit tcp object HR object FILE-SVR eq 445  
access-list INSIDE\_IN extended permit icmp object HR object FILE-SVR  
access-list INSIDE\_IN extended permit ip object-group STAFF-ATM object ACCT-SVR  
access-list INSIDE\_IN extended permit ip object-group STAFF-ATM object TXN-SVR  
access-list INSIDE\_IN extended permit udp object-group HR-IT object DNS-SVR eq domain  
access-list INSIDE\_IN extended permit tcp object-group HR-IT object DNS-SVR eq domain  
access-list INSIDE\_IN extended permit tcp object-group HR-IT object WEB-SVR eq www  
access-list INSIDE\_IN extended permit tcp object-group HR-IT object WEB-SVR eq https  
access-list INSIDE\_IN extended permit tcp object IT object-group JUMP-SVRs eq ssh  
access-list INSIDE\_IN extended deny ip any object SERVERS log  
access-list INSIDE\_IN extended deny ip any object DMZ-SUBNET log  
access-list INSIDE\_IN extended deny ip object-group STAFF-ATM any log  
access-list INSIDE\_IN extended permit ip object-group HR-IT any  
access-list INSIDE\_IN extended deny ip any any log  
access-list SERVER\_IN extended permit udp object DHCP-SVR eq bootps object-group DSW-SVIS eq bootps  
access-list SERVER\_IN extended permit tcp object SERVERS any eq www  
access-list SERVER\_IN extended permit tcp object SERVERS any eq https  
access-list SERVER\_IN extended permit udp object SERVERS host 8.8.8.8 eq domain  
access-list SERVER\_IN extended deny ip object SERVERS object INTERNAL log  
access-list SERVER\_IN extended permit tcp object TXN-SVR any eq www time-range TXN-WINDOW  
access-list SERVER\_IN extended permit tcp object TXN-SVR any eq https time-range TXN-WINDOW  
access-list SERVER\_IN extended permit udp object TXN-SVR object DNS-SVR eq domain time-range TXN-WINDOW  
access-list SERVER\_IN extended deny ip any any log  
access-list DMZ\_IN extended deny ip object DMZ-SUBNET object INTERNAL log  
access-list DMZ\_IN extended deny ip object DMZ-SUBNET object SERVERS log  
access-list DMZ\_IN extended permit udp object-group JUMP-SVRs any eq ntp  
access-list DMZ\_IN extended permit udp object-group JUMP-SVRs any eq domain  
access-list DMZ\_IN extended deny ip any any log  
access-list OUTSIDE\_IN extended permit ospf any any  
access-list OUTSIDE\_IN extended permit tcp any object WEB-SVR eq www  
access-list OUTSIDE\_IN extended permit tcp any object WEB-SVR eq https  
access-list OUTSIDE\_IN extended permit udp any object DNS-SVR eq domain  
access-list OUTSIDE\_IN extended permit tcp any object DNS-SVR eq domain  
access-list OUTSIDE\_IN extended permit tcp any object JUMP-SVR eq ssh  
access-list OUTSIDE\_IN extended deny ip any any log  
pager lines 24  
logging enable  
logging timestamp  
logging trap informational  
logging asdm informational  
logging host management 172.16.0.44  
logging host management 172.16.0.45  
mtu server 1500  
mtu dmz 1500  
mtu outside 1500  
mtu wan-backup 1500  
mtu inside 1500  
mtu management 1500  
icmp unreachable rate-limit 1 burst-size 1  
no asdm history enable  
arp timeout 14400  
no arp permit-nonconnected  
arp rate-limit 16384  
\!  
object network WEB-SVR-NAT  
 nat (dmz,wan-backup) static 203.149.210.189  
object network DNS-SVR-NAT  
 nat (dmz,wan-backup) static 203.149.210.190  
object network JUMP-SSH-NAT  
 nat (dmz,wan-backup) static 203.149.210.185 service tcp ssh 2222  
\!  
nat (inside,wan-backup) after-auto source dynamic PAT-USERS PUB-PAT  
nat (server,wan-backup) after-auto source dynamic PAT-USERS PUB-PAT  
nat (dmz,wan-backup) after-auto source dynamic PAT-USERS PUB-PAT  
router ospf 1  
 router-id 5.5.5.5  
 network 172.16.0.96 255.255.255.248 area 0  
 network 172.16.0.104 255.255.255.252 area 0  
 network 172.16.0.112 255.255.255.252 area 0  
 log-adj-changes  
 redistribute connected subnets  
\!  
route management 0.0.0.0 0.0.0.0 172.20.11.254 1  
route management 0.0.0.0 0.0.0.0 172.16.0.33 1  
timeout xlate 3:00:00  
timeout pat-xlate 0:00:30  
timeout conn 1:00:00 half-closed 0:10:00 udp 0:02:00 sctp 0:02:00 icmp 0:00:02  
timeout sunrpc 0:10:00 h323 0:05:00 h225 1:00:00 mgcp 0:05:00 mgcp-pat 0:05:00  
timeout sip 0:30:00 sip\_media 0:02:00 sip-invite 0:03:00 sip-disconnect 0:02:00  
timeout sip-provisional-media 0:02:00 uauth 0:05:00 absolute  
timeout tcp-proxy-reassembly 0:01:00  
timeout floating-conn 0:00:00  
timeout conn-holddown 0:00:15  
timeout igp stale-route 0:01:10  
aaa-server TACACS protocol tacacs+  
aaa-server TACACS (management) host 172.16.0.44  
 key \*\*\*\*\*  
aaa-server TACACS (management) host 172.16.0.45  
 key \*\*\*\*\*  
user-identity default-domain LOCAL  
aaa authentication ssh console TACACS LOCAL  
aaa authentication enable console TACACS LOCAL  
aaa accounting command TACACS  
aaa authentication login-history  
http 192.168.45.0 255.255.255.0 management  
no snmp-server location  
no snmp-server contact  
crypto ipsec security-association pmtu-aging infinite  
crypto ca trustpoint \_SmartCallHome\_ServerCA  
 no validation-usage  
 crl configure  
crypto ca trustpoint \_SmartCallHome\_ServerCA2  
 no validation-usage  
 crl configure  
crypto ca trustpool policy  
 auto-import  
crypto ca certificate chain \_SmartCallHome\_ServerCA  
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
crypto ca certificate chain \_SmartCallHome\_ServerCA2  
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
no threat-detection statistics tcp-intercept  
ntp server 172.16.0.44 source management prefer  
ntp server 172.16.0.45 source management  
dynamic-access-policy-record DfltAccessPolicy  
username sitstudent password \*\*\*\*\* pbkdf2 privilege 15  
username sitadmin password \*\*\*\*\* pbkdf2 privilege 15  
username bigDonkBonkAdmin password \*\*\*\*\* pbkdf2 privilege 15  
username donkBonkEng2 password \*\*\*\*\* pbkdf2 privilege 5  
username donkBonkEng1 password \*\*\*\*\* pbkdf2 privilege 5  
\!  
class-map inspection\_default  
 match default-inspection-traffic  
\!  
\!  
policy-map type inspect dns preset\_dns\_map  
 parameters  
  message-length maximum client auto  
  message-length maximum 512  
  no tcp-inspection  
policy-map global\_policy  
 class inspection\_default  
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
  inspect dns preset\_dns\_map  
  inspect icmp  
  inspect icmp error  
policy-map type inspect dns migrated\_dns\_map\_2  
 parameters  
  message-length maximum client auto  
  message-length maximum 512  
  no tcp-inspection  
policy-map type inspect dns migrated\_dns\_map\_1  
 parameters  
  message-length maximum client auto  
  message-length maximum 512  
  no tcp-inspection  
\!  
service-policy global\_policy global  
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
  destination address http https\://tools.cisco.com/its/service/oddce/services/DDCEService  
  destination transport-method http  
 profile CiscoTAC-1  
  no active  
  destination address http https\://tools.cisco.com/its/service/oddce/services/DDCEService  
  destination address email callhome@cisco.com  
  destination transport-method http  
  subscribe-to-alert-group diagnostic  
  subscribe-to-alert-group environment  
  subscribe-to-alert-group inventory periodic monthly  
  subscribe-to-alert-group configuration periodic monthly  
  subscribe-to-alert-group telemetry periodic daily  
Cryptochecksum:0def4eabdc786e34162b9b3deee3bdff  
: end

bigDonkBonkAdmin@PA-440\> set cli pager off  
bigDonkBonkAdmin@PA-440\> set cli config-output-format set  
bigDonkBonkAdmin@PA-440\> sonfigure  
Unknown command: sonfigure  
bigDonkBonkAdmin@PA-440\> configure  
Entering configuration mode  
\[edit\]  
bigDonkBonkAdmin@PA-440\# show  
set deviceconfig system ip-address 172.16.0.41  
set deviceconfig system netmask 255.255.255.240  
set deviceconfig system update-server updates.paloaltonetworks.com  
set deviceconfig system update-schedule threats recurring weekly day-of-week wednesday  
set deviceconfig system update-schedule threats recurring weekly at 01:02  
set deviceconfig system update-schedule threats recurring weekly action download-only  
set deviceconfig system timezone Asia/Singapore  
set deviceconfig system service disable-telnet yes  
set deviceconfig system service disable-http yes  
set deviceconfig system service disable-ssh no  
set deviceconfig system service disable-https no  
set deviceconfig system hostname PA-440  
set deviceconfig system device-telemetry device-health-performance yes  
set deviceconfig system device-telemetry product-usage yes  
set deviceconfig system device-telemetry threat-prevention yes  
set deviceconfig system device-telemetry region Americas  
set deviceconfig system domain gruppesechs.sitict.net  
set deviceconfig system login-banner "WARNING: UNAUTHORIZED ACCESS TO GRUPPESECHS PA-440 IS PROHIBITED."  
set deviceconfig system type static  
set deviceconfig system default-gateway 172.16.0.33  
set deviceconfig system ntp-servers primary-ntp-server ntp-server-address 172.16.0.44  
set deviceconfig system ntp-servers secondary-ntp-server ntp-server-address 172.16.0.45  
set deviceconfig system authentication-profile SEQ-ADMIN  
set deviceconfig system permitted-ip 172.16.0.44/32  
set deviceconfig system permitted-ip 172.16.0.45/32  
set deviceconfig setting config rematch yes  
set deviceconfig setting management hostname-type-in-syslog FQDN  
set network interface ethernet ethernet1/1 link-state down  
set network interface ethernet ethernet1/2 layer2 lldp enable no  
set network interface ethernet ethernet1/5 layer2 lldp enable no  
set network interface ethernet ethernet1/3 layer3 ip 172.16.0.105/30  
set network interface ethernet ethernet1/3 layer3 interface-management-profile PING-ONLY  
set network interface ethernet ethernet1/4 layer2 lldp enable no  
set network interface ethernet ethernet1/6 layer2 lldp enable no  
set network interface ethernet ethernet1/6 link-state down  
set network interface ethernet ethernet1/7 layer2 lldp enable no  
set network interface ethernet ethernet1/7 link-state down  
set network interface ethernet ethernet1/8 layer2 lldp enable no  
set network interface loopback units  
set network interface vlan units vlan.10 ip 172.16.0.92/29  
set network interface vlan units vlan.10 interface-management-profile PING-ONLY  
set network interface vlan units vlan.92 ip 172.16.0.81/29  
set network interface vlan units vlan.92 interface-management-profile PING-ONLY  
set network interface vlan units vlan.100 ip 172.16.0.100/29  
set network interface vlan units vlan.100 interface-management-profile PING-ONLY  
set network interface vlan units vlan.69 ip 172.16.0.78/29  
set network interface vlan units vlan.69 interface-management-profile PING-ONLY  
set network interface tunnel units  
set network vlan OUTSIDE virtual-interface interface vlan.10  
set network vlan OUTSIDE interface \[ ethernet1/2 ethernet1/5 \]  
set network vlan VL-DMZ interface ethernet1/4  
set network vlan VL-DMZ virtual-interface interface vlan.92  
set network vlan VL-INSIDE-BYPASS interface \[ ethernet1/6 ethernet1/7 \]  
set network vlan VL-INSIDE-BYPASS virtual-interface interface vlan.100  
set network vlan VL-SERVER interface ethernet1/8  
set network vlan VL-SERVER virtual-interface interface vlan.69  
set network virtual-wire  
set network profiles monitor-profile default interval 3  
set network profiles monitor-profile default threshold 5  
set network profiles monitor-profile default action wait-recover  
set network profiles interface-management-profile PING-ONLY ping yes  
set network ike crypto-profiles ike-crypto-profiles default encryption \[ aes-128-cbc 3des \]  
set network ike crypto-profiles ike-crypto-profiles default hash sha1  
set network ike crypto-profiles ike-crypto-profiles default dh-group group2  
set network ike crypto-profiles ike-crypto-profiles default lifetime hours 8  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-128 encryption aes-128-cbc  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-128 hash sha256  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-128 dh-group group19  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-128 lifetime hours 8  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-256 encryption aes-256-cbc  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-256 hash sha384  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-256 dh-group group20  
set network ike crypto-profiles ike-crypto-profiles Suite-B-GCM-256 lifetime hours 8  
set network ike crypto-profiles ipsec-crypto-profiles default esp encryption \[ aes-128-cbc 3des \]  
set network ike crypto-profiles ipsec-crypto-profiles default esp authentication sha1  
set network ike crypto-profiles ipsec-crypto-profiles default dh-group group2  
set network ike crypto-profiles ipsec-crypto-profiles default lifetime hours 1  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-128 esp encryption aes-128-gcm  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-128 esp authentication none  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-128 dh-group group19  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-128 lifetime hours 1  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-256 esp encryption aes-256-gcm  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-256 esp authentication none  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-256 dh-group group20  
set network ike crypto-profiles ipsec-crypto-profiles Suite-B-GCM-256 lifetime hours 1  
set network ike crypto-profiles global-protect-app-crypto-profiles default encryption aes-128-cbc  
set network ike crypto-profiles global-protect-app-crypto-profiles default authentication sha1  
set network qos profile default class-bandwidth-type mbps class class1 priority real-time  
set network qos profile default class-bandwidth-type mbps class class2 priority high  
set network qos profile default class-bandwidth-type mbps class class3 priority high  
set network qos profile default class-bandwidth-type mbps class class4 priority medium  
set network qos profile default class-bandwidth-type mbps class class5 priority medium  
set network qos profile default class-bandwidth-type mbps class class6 priority low  
set network qos profile default class-bandwidth-type mbps class class7 priority low  
set network qos profile default class-bandwidth-type mbps class class8 priority low  
set network virtual-router default protocol bgp enable no  
set network virtual-router default protocol bgp dampening-profile default cutoff 1.25  
set network virtual-router default protocol bgp dampening-profile default reuse 0.5  
set network virtual-router default protocol bgp dampening-profile default max-hold-time 900  
set network virtual-router default protocol bgp dampening-profile default decay-half-life-reachable 300  
set network virtual-router default protocol bgp dampening-profile default decay-half-life-unreachable 900  
set network virtual-router default protocol bgp dampening-profile default enable yes  
set network virtual-router default protocol ospf router-id 6.6.6.6  
set network virtual-router default protocol ospf area 0.0.0.0 type normal  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.10 enable yes  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.10 passive no  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.10 metric 10  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.10 link-type broadcast  
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/3 enable yes  
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/3 passive no  
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/3 metric 10  
set network virtual-router default protocol ospf area 0.0.0.0 interface ethernet1/3 link-type broadcast  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.92 enable yes  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.92 passive no  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.92 metric 10  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.100 enable yes  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.100 passive no  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.100 metric 500  
set network virtual-router default protocol ospf area 0.0.0.0 interface vlan.100 link-type broadcast  
set network virtual-router default protocol ospf enable yes  
set network virtual-router default interface \[ vlan.10 ethernet1/3 vlan.92 vlan.100 \]  
set network virtual-router default routing-table ip static-route DEFAULT-OUT destination 0.0.0.0/0  
set network virtual-router default routing-table ip static-route DEFAULT-OUT interface vlan.10  
set network virtual-router default routing-table ip static-route DEFAULT-OUT nexthop ip-address 172.16.0.90  
set shared application  
set shared application-group  
set shared service  
set shared service-group  
set shared botnet configuration http dynamic-dns enabled yes  
set shared botnet configuration http dynamic-dns threshold 5  
set shared botnet configuration http malware-sites enabled yes  
set shared botnet configuration http malware-sites threshold 5  
set shared botnet configuration http recent-domains enabled yes  
set shared botnet configuration http recent-domains threshold 5  
set shared botnet configuration http ip-domains enabled yes  
set shared botnet configuration http ip-domains threshold 10  
set shared botnet configuration http executables-from-unknown-sites enabled yes  
set shared botnet configuration http executables-from-unknown-sites threshold 5  
set shared botnet configuration other-applications irc yes  
set shared botnet configuration unknown-applications unknown-tcp destinations-per-hour 10  
set shared botnet configuration unknown-applications unknown-tcp sessions-per-hour 10  
set shared botnet configuration unknown-applications unknown-tcp session-length maximum-bytes 100  
set shared botnet configuration unknown-applications unknown-tcp session-length minimum-bytes 50  
set shared botnet configuration unknown-applications unknown-udp destinations-per-hour 10  
set shared botnet configuration unknown-applications unknown-udp sessions-per-hour 10  
set shared botnet configuration unknown-applications unknown-udp session-length maximum-bytes 100  
set shared botnet configuration unknown-applications unknown-udp session-length minimum-bytes 50  
set shared botnet report topn 100  
set shared botnet report scheduled yes  
set shared server-profile tacplus TACACS server JUMP address 172.16.0.44  
set shared server-profile tacplus TACACS server JUMP secret \-AQ==VUmGNWPdWuJt2J9MFpDMqf7If6U=puGOOXAA9pi+zTF76jUsHHEctcNmZGGxiXuI8n2UGZE=  
set shared server-profile tacplus TACACS server JUMP2 address 172.16.0.45  
set shared server-profile tacplus TACACS server JUMP2 secret \-AQ==qmQnvaHTF5quvUkoQXHi8EiZZvQ=puGOOXAA9pi+zTF76jUsHOVRSn255prHAE5++8r8X6c=  
set shared server-profile tacplus TACACS timeout 3  
set shared server-profile tacplus TACACS protocol PAP  
set shared authentication-profile AUTH-TACACS method tacplus server-profile TACACS  
set shared authentication-profile AUTH-TACACS allow-list all  
set shared authentication-profile AUTH-LOCAL method local-database  
set shared authentication-profile AUTH-LOCAL allow-list all  
set shared authentication-sequence SEQ-ADMIN authentication-profiles \[ AUTH-TACACS AUTH-LOCAL \]  
set shared log-settings syslog SYSLOG-JUMP server JUMP server 172.16.0.44  
set shared log-settings syslog SYSLOG-JUMP server JUMP transport UDP  
set shared log-settings syslog SYSLOG-JUMP server JUMP port 514  
set shared log-settings syslog SYSLOG-JUMP server JUMP format BSD  
set shared log-settings syslog SYSLOG-JUMP server JUMP facility LOG\_USER  
set shared log-settings syslog SYSLOG-JUMP server JUMP2 server 172.16.0.45  
set shared log-settings syslog SYSLOG-JUMP server JUMP2 transport UDP  
set shared log-settings syslog SYSLOG-JUMP server JUMP2 port 514  
set shared log-settings syslog SYSLOG-JUMP server JUMP2 format BSD  
set shared log-settings syslog SYSLOG-JUMP server JUMP2 facility LOG\_USER  
set zone outside network layer3 vlan.10  
set zone outside-l2 network layer2 \[ ethernet1/5 ethernet1/2 \]  
set zone inside network layer3 \[ ethernet1/3 vlan.100 \]  
set zone inside-l2 network layer2 \[ ethernet1/6 ethernet1/7 \]  
set zone dmz network layer3 vlan.92  
set zone dmz-l2 network layer2 ethernet1/4  
set zone server network layer3 vlan.69  
set zone server-l2 network layer2 ethernet1/8  
set service-group  
set service TCP-2222 protocol tcp port 2222  
set service UDP-67 protocol udp port 67  
set schedule TXN-WINDOW schedule-type recurring daily 01:00-03:00  
set rulebase security rules ALLOW-OSPF from \[ outside inside server \]  
set rulebase security rules ALLOW-OSPF to \[ outside inside server \]  
set rulebase security rules ALLOW-OSPF source any  
set rulebase security rules ALLOW-OSPF destination any  
set rulebase security rules ALLOW-OSPF source-user any  
set rulebase security rules ALLOW-OSPF category any  
set rulebase security rules ALLOW-OSPF application ospf  
set rulebase security rules ALLOW-OSPF service any  
set rulebase security rules ALLOW-OSPF action allow  
set rulebase security rules ALLOW-OSPF log-end no  
set rulebase security rules INBOUND-WEB from outside  
set rulebase security rules INBOUND-WEB to dmz  
set rulebase security rules INBOUND-WEB source any  
set rulebase security rules INBOUND-WEB destination PUB-WEB  
set rulebase security rules INBOUND-WEB source-user any  
set rulebase security rules INBOUND-WEB category any  
set rulebase security rules INBOUND-WEB application \[ web-browsing ssl \]  
set rulebase security rules INBOUND-WEB service application-default  
set rulebase security rules INBOUND-WEB action allow  
set rulebase security rules INBOUND-WEB log-end yes  
set rulebase security rules INBOUND-DNS from outside  
set rulebase security rules INBOUND-DNS to dmz  
set rulebase security rules INBOUND-DNS source any  
set rulebase security rules INBOUND-DNS destination PUB-DNS  
set rulebase security rules INBOUND-DNS source-user any  
set rulebase security rules INBOUND-DNS category any  
set rulebase security rules INBOUND-DNS application dns  
set rulebase security rules INBOUND-DNS service application-default  
set rulebase security rules INBOUND-DNS action allow  
set rulebase security rules INBOUND-DNS log-end yes  
set rulebase security rules JUMP-SSH-IN from outside  
set rulebase security rules JUMP-SSH-IN to dmz  
set rulebase security rules JUMP-SSH-IN source any  
set rulebase security rules JUMP-SSH-IN destination PUB-PAT  
set rulebase security rules JUMP-SSH-IN source-user any  
set rulebase security rules JUMP-SSH-IN category any  
set rulebase security rules JUMP-SSH-IN application ssh  
set rulebase security rules JUMP-SSH-IN service TCP-2222  
set rulebase security rules JUMP-SSH-IN action allow  
set rulebase security rules JUMP-SSH-IN log-end yes  
set rulebase security rules INTERNAL-TO-DNS from inside  
set rulebase security rules INTERNAL-TO-DNS to dmz  
set rulebase security rules INTERNAL-TO-DNS source INTERNET-USERS  
set rulebase security rules INTERNAL-TO-DNS destination DNS-SVR  
set rulebase security rules INTERNAL-TO-DNS source-user any  
set rulebase security rules INTERNAL-TO-DNS category any  
set rulebase security rules INTERNAL-TO-DNS application dns  
set rulebase security rules INTERNAL-TO-DNS service application-default  
set rulebase security rules INTERNAL-TO-DNS action allow  
set rulebase security rules INTERNAL-TO-DNS log-end yes  
set rulebase security rules INTERNAL-TO-WEB from inside  
set rulebase security rules INTERNAL-TO-WEB to dmz  
set rulebase security rules INTERNAL-TO-WEB source INTERNET-USERS  
set rulebase security rules INTERNAL-TO-WEB destination WEB-SVR  
set rulebase security rules INTERNAL-TO-WEB source-user any  
set rulebase security rules INTERNAL-TO-WEB category any  
set rulebase security rules INTERNAL-TO-WEB application \[ web-browsing ssl \]  
set rulebase security rules INTERNAL-TO-WEB service application-default  
set rulebase security rules INTERNAL-TO-WEB action allow  
set rulebase security rules INTERNAL-TO-WEB log-end yes  
set rulebase security rules IT-TO-JUMP-SSH from inside  
set rulebase security rules IT-TO-JUMP-SSH to dmz  
set rulebase security rules IT-TO-JUMP-SSH source IT-NET  
set rulebase security rules IT-TO-JUMP-SSH destination \[ JUMP-SVR JUMP2-SVR \]  
set rulebase security rules IT-TO-JUMP-SSH source-user any  
set rulebase security rules IT-TO-JUMP-SSH category any  
set rulebase security rules IT-TO-JUMP-SSH application ssh  
set rulebase security rules IT-TO-JUMP-SSH service application-default  
set rulebase security rules IT-TO-JUMP-SSH action allow  
set rulebase security rules IT-TO-JUMP-SSH log-end yes  
set rulebase security rules PTSO-HR-TO-FILE from inside  
set rulebase security rules PTSO-HR-TO-FILE to server  
set rulebase security rules PTSO-HR-TO-FILE source HR-NET  
set rulebase security rules PTSO-HR-TO-FILE destination FILE-SVR  
set rulebase security rules PTSO-HR-TO-FILE source-user any  
set rulebase security rules PTSO-HR-TO-FILE category any  
set rulebase security rules PTSO-HR-TO-FILE application \[ ms-ds-smb ping \]  
set rulebase security rules PTSO-HR-TO-FILE service application-default  
set rulebase security rules PTSO-HR-TO-FILE action allow  
set rulebase security rules PTSO-HR-TO-FILE log-end yes  
set rulebase security rules PTSO-STAFF-ATM-SVR from inside  
set rulebase security rules PTSO-STAFF-ATM-SVR to server  
set rulebase security rules PTSO-STAFF-ATM-SVR source STAFF-ATM  
set rulebase security rules PTSO-STAFF-ATM-SVR destination \[ ACCT-SVR TXN-SVR \]  
set rulebase security rules PTSO-STAFF-ATM-SVR source-user any  
set rulebase security rules PTSO-STAFF-ATM-SVR category any  
set rulebase security rules PTSO-STAFF-ATM-SVR application any  
set rulebase security rules PTSO-STAFF-ATM-SVR service any  
set rulebase security rules PTSO-STAFF-ATM-SVR action allow  
set rulebase security rules PTSO-STAFF-ATM-SVR log-end yes  
set rulebase security rules PTSO-DHCP-RELAY from inside  
set rulebase security rules PTSO-DHCP-RELAY to server  
set rulebase security rules PTSO-DHCP-RELAY source DSWs  
set rulebase security rules PTSO-DHCP-RELAY destination DHCP-SVR  
set rulebase security rules PTSO-DHCP-RELAY source-user any  
set rulebase security rules PTSO-DHCP-RELAY category any  
set rulebase security rules PTSO-DHCP-RELAY application dhcp  
set rulebase security rules PTSO-DHCP-RELAY service UDP-67  
set rulebase security rules PTSO-DHCP-RELAY action allow  
set rulebase security rules PTSO-DHCP-RELAY log-end yes  
set rulebase security rules HR-IT-INTERNET from inside  
set rulebase security rules HR-IT-INTERNET to outside  
set rulebase security rules HR-IT-INTERNET source INTERNET-USERS  
set rulebase security rules HR-IT-INTERNET destination any  
set rulebase security rules HR-IT-INTERNET source-user any  
set rulebase security rules HR-IT-INTERNET category any  
set rulebase security rules HR-IT-INTERNET application any  
set rulebase security rules HR-IT-INTERNET service any  
set rulebase security rules HR-IT-INTERNET action allow  
set rulebase security rules HR-IT-INTERNET log-end yes  
set rulebase security rules TXN-INTERNET-TIMED from \[ inside server \]  
set rulebase security rules TXN-INTERNET-TIMED to \[ outside dmz \]  
set rulebase security rules TXN-INTERNET-TIMED source TXN-SVR  
set rulebase security rules TXN-INTERNET-TIMED destination any  
set rulebase security rules TXN-INTERNET-TIMED source-user any  
set rulebase security rules TXN-INTERNET-TIMED category any  
set rulebase security rules TXN-INTERNET-TIMED application \[ web-browsing ssl dns \]  
set rulebase security rules TXN-INTERNET-TIMED service application-default  
set rulebase security rules TXN-INTERNET-TIMED schedule TXN-WINDOW  
set rulebase security rules TXN-INTERNET-TIMED action allow  
set rulebase security rules TXN-INTERNET-TIMED log-end yes  
set rulebase security rules DMZ-OUTBOUND-NTP from dmz  
set rulebase security rules DMZ-OUTBOUND-NTP to outside  
set rulebase security rules DMZ-OUTBOUND-NTP source \[ JUMP-SVR JUMP2-SVR \]  
set rulebase security rules DMZ-OUTBOUND-NTP destination any  
set rulebase security rules DMZ-OUTBOUND-NTP source-user any  
set rulebase security rules DMZ-OUTBOUND-NTP category any  
set rulebase security rules DMZ-OUTBOUND-NTP application \[ ntp dns \]  
set rulebase security rules DMZ-OUTBOUND-NTP service application-default  
set rulebase security rules DMZ-OUTBOUND-NTP action allow  
set rulebase security rules DMZ-OUTBOUND-NTP log-end yes  
set rulebase security rules TEMP-SVR-UPDATES from \[ inside dmz \]  
set rulebase security rules TEMP-SVR-UPDATES to outside  
set rulebase security rules TEMP-SVR-UPDATES source \[ SERVERS DMZ-SUBNET \]  
set rulebase security rules TEMP-SVR-UPDATES destination any  
set rulebase security rules TEMP-SVR-UPDATES source-user any  
set rulebase security rules TEMP-SVR-UPDATES category any  
set rulebase security rules TEMP-SVR-UPDATES application any  
set rulebase security rules TEMP-SVR-UPDATES service any  
set rulebase security rules TEMP-SVR-UPDATES action allow  
set rulebase security rules TEMP-SVR-UPDATES log-end yes  
set rulebase security rules DNS-SVR-FORWARD from dmz  
set rulebase security rules DNS-SVR-FORWARD to outside  
set rulebase security rules DNS-SVR-FORWARD source DNS-SVR  
set rulebase security rules DNS-SVR-FORWARD destination \[ DNS-FORWARDERS-1 DNS-FORWARDERS-2 ICT-DNS \]  
set rulebase security rules DNS-SVR-FORWARD source-user any  
set rulebase security rules DNS-SVR-FORWARD category any  
set rulebase security rules DNS-SVR-FORWARD application dns  
set rulebase security rules DNS-SVR-FORWARD service application-default  
set rulebase security rules DNS-SVR-FORWARD action allow  
set rulebase security rules DNS-SVR-FORWARD log-end yes  
set rulebase security rules DENY-ALL rule-type interzone  
set rulebase security rules DENY-ALL from any  
set rulebase security rules DENY-ALL to any  
set rulebase security rules DENY-ALL source any  
set rulebase security rules DENY-ALL destination any  
set rulebase security rules DENY-ALL source-user any  
set rulebase security rules DENY-ALL category any  
set rulebase security rules DENY-ALL application any  
set rulebase security rules DENY-ALL service any  
set rulebase security rules DENY-ALL action deny  
set rulebase security rules DENY-ALL log-end yes  
set rulebase nat rules NAT-WEB from outside  
set rulebase nat rules NAT-WEB to outside  
set rulebase nat rules NAT-WEB source any  
set rulebase nat rules NAT-WEB destination PUB-WEB  
set rulebase nat rules NAT-WEB service service-http  
set rulebase nat rules NAT-WEB destination-translation translated-address WEB-SVR  
set rulebase nat rules NAT-DNS from outside  
set rulebase nat rules NAT-DNS to outside  
set rulebase nat rules NAT-DNS source any  
set rulebase nat rules NAT-DNS destination PUB-DNS  
set rulebase nat rules NAT-DNS service any  
set rulebase nat rules NAT-DNS destination-translation translated-address DNS-SVR  
set rulebase nat rules NAT-JUMP-SSH from outside  
set rulebase nat rules NAT-JUMP-SSH to outside  
set rulebase nat rules NAT-JUMP-SSH source any  
set rulebase nat rules NAT-JUMP-SSH destination PUB-PAT  
set rulebase nat rules NAT-JUMP-SSH service TCP-2222  
set rulebase nat rules NAT-JUMP-SSH destination-translation translated-address JUMP-SVR  
set rulebase nat rules NAT-JUMP-SSH destination-translation translated-port 22  
set rulebase nat rules NAT-OUTBOUND source-translation dynamic-ip-and-port translated-address PUB-PAT  
set rulebase nat rules NAT-OUTBOUND from \[ inside server dmz \]  
set rulebase nat rules NAT-OUTBOUND to outside  
set rulebase nat rules NAT-OUTBOUND source \[ HR-NET IT-NET TXN-SVR JUMP-SVR JUMP2-SVR DNS-SVR \]  
set rulebase nat rules NAT-OUTBOUND destination any  
set rulebase nat rules NAT-OUTBOUND service any  
set rulebase nat rules TEMP-SVR-NAT from \[ inside dmz \]  
set rulebase nat rules TEMP-SVR-NAT to outside  
set rulebase nat rules TEMP-SVR-NAT source \[ SERVERS DMZ-SUBNET \]  
set rulebase nat rules TEMP-SVR-NAT destination any  
set rulebase nat rules TEMP-SVR-NAT service any  
set rulebase nat rules TEMP-SVR-NAT source-translation dynamic-ip-and-port translated-address PUB-PAT  
set rulebase nat rules NAT-WEB-HTTPS from outside  
set rulebase nat rules NAT-WEB-HTTPS to outside  
set rulebase nat rules NAT-WEB-HTTPS source any  
set rulebase nat rules NAT-WEB-HTTPS destination PUB-WEB  
set rulebase nat rules NAT-WEB-HTTPS service service-https  
set rulebase nat rules NAT-WEB-HTTPS destination-translation translated-address WEB-SVR  
set application-group  
set application  
set address-group INTERNET-USERS static \[ HR-NET IT-NET \]  
set address-group STAFF-ATM static \[ STAFF ATM \]  
set address-group DSWs static \[ DSW1-SVI DSW2-SVI \]  
set address-group HR-IT static \[ HR-NET IT-NET \]  
set address STAFF ip-netmask 172.16.0.0/27  
set address ATM ip-netmask 172.16.0.48/29  
set address FILE-SVR ip-netmask 172.16.0.74/32  
set address ACCT-SVR ip-netmask 172.16.0.75/32  
set address TXN-SVR ip-netmask 172.16.0.76/32  
set address DHCP-SVR ip-netmask 172.16.0.77/32  
set address WEB-SVR ip-netmask 172.16.0.82/32  
set address DNS-SVR ip-netmask 172.16.0.83/32  
set address JUMP-SVR ip-netmask 172.16.0.84/32  
set address JUMP2-SVR ip-netmask 172.16.0.85/32  
set address PUB-PAT ip-netmask 203.149.210.185/32  
set address PUB-WEB ip-netmask 203.149.210.189/32  
set address PUB-DNS ip-netmask 203.149.210.190/32  
set address HR-NET ip-netmask 172.16.0.56/29  
set address IT-NET ip-netmask 172.16.0.64/29  
set address SERVERS ip-netmask 172.16.0.72/29  
set address DMZ-SUBNET ip-netmask 172.16.0.80/29  
set address DSW1-SVI ip-netmask 172.16.0.98/32  
set address DSW2-SVI ip-netmask 172.16.0.99/32  
set address DNS-FORWARDERS-1 ip-netmask 8.8.8.8/32  
set address DNS-FORWARDERS-2 ip-netmask 1.1.1.1/32  
set address ICT-DNS ip-netmask 172.18.4.32/32  
set mgt-config users admin phash $5$fqknpuqq\$2b1Qq0KiHI/lppBjjtzwE7hIHHl52xfICtuQCLxwa5.  
set mgt-config users admin permissions role-based superuser yes  
set mgt-config users sitstudent permissions role-based superuser yes  
set mgt-config users sitstudent description admin  
set mgt-config users sitstudent phash $5$yuukakqc\$mEfHLjFbt1YQWbIriisTGRed/ChCRO57CpldUabCNB7  
set mgt-config users bigDonkBonkAdmin permissions role-based superuser yes  
set mgt-config users bigDonkBonkAdmin phash $5$utsiybia\$lbXWHMP4osKRzqNS4yvBclKU4XQsYp5IPD1C9o8quRD  
set mgt-config users donkBonkEng1 phash $5$vsoppprv\$E/ssv9JrNuta3QJdYtvarHCm3NsbEazvWRxABl0DTVB  
set mgt-config users donkBonkEng1 permissions role-based superreader yes  
set mgt-config users donkBonkEng2 phash $5$gopucdnf\$Z2XxLe51K1.N4N/pZWOMz0y38KkTz8xYme3qnrPhBC1  
set mgt-config users donkBonkEng2 permissions role-based superreader yes  
set mgt-config password-complexity enabled yes  
set mgt-config password-complexity minimum-length 8  
\[edit\]  
