1.1 brute force prevention
added fail 2 ban here, so step 11 is done.

for routers
ip access-list extended ACL-COPP-SSH-TRUSTED
 permit tcp host 172.16.0.44 any eq 22

ip access-list extended ACL-COPP-SSH-RESTRICTED
 permit tcp host 172.16.0.45 any eq 22

class-map match-all CM-COPP-SSH-TRUSTED
 match access-group name ACL-COPP-SSH-TRUSTED

class-map match-all CM-COPP-SSH-RESTRICTED
 match access-group name ACL-COPP-SSH-RESTRICTED

// targets ssh from .44 and .45(jumphosts), set 1 million bits per second for police
// conform action to transmit, even if brute force is happening
// for restricted, set to 8000 bits per second, it will drop ssh if it exceeds 8000 bits,
// typical ssh takes like around 100 bits, so 8000 bits would detect brute force.
policy-map PM-COPP-CONTROL-PLANE
 class CM-COPP-SSH-TRUSTED
  police 1000000 conform-action transmit exceed-action transmit
 class CM-COPP-SSH-RESTRICTED
  police 8000 conform-action transmit exceed-action drop

control-plane
 service-policy input PM-COPP-CONTROL-PLANE
exit

for dsw
ip access-list extended ACL-COPP-ROUTING
 permit ospf any any
 permit udp any host 224.0.0.102 eq 1985

ip access-list extended ACL-COPP-SSH-TRUSTED
 permit tcp host 172.16.0.44 any eq 22

ip access-list extended ACL-COPP-SSH-RESTRICTED
 permit tcp host 172.16.0.45 any eq 22

class-map match-all CM-COPP-ROUTING
 match access-group name ACL-COPP-ROUTING

class-map match-all CM-COPP-SSH-TRUSTED
 match access-group name ACL-COPP-SSH-TRUSTED

class-map match-all CM-COPP-SSH-RESTRICTED
 match access-group name ACL-COPP-SSH-RESTRICTED

policy-map PM-COPP-SWITCH
 class CM-COPP-ROUTING
  police rate 1000 pps conform-action transmit exceed-action transmit
 class CM-COPP-SSH-TRUSTED
  police rate 100 pps conform-action transmit exceed-action transmit
 class CM-COPP-SSH-RESTRICTED
  police rate 5 pps conform-action drop exceed-action drop

control-plane
 service-policy input PM-COPP-SWITCH
exit


for sw

ip access-list extended ACL-COPP-SSH-TRUSTED
 permit tcp host 172.16.0.44 any eq 22

ip access-list extended ACL-COPP-SSH-RESTRICTED
 permit tcp host 172.16.0.45 any eq 22

class-map match-all CM-COPP-SSH-TRUSTED
 match access-group name ACL-COPP-SSH-TRUSTED

class-map match-all CM-COPP-SSH-RESTRICTED
 match access-group name ACL-COPP-SSH-RESTRICTED

policy-map PM-COPP-ACCESS
 class CM-COPP-SSH-TRUSTED
  police rate 50 pps conform-action transmit exceed-action transmit
 class CM-COPP-SSH-RESTRICTED
  police rate 2 pps conform-action drop exceed-action drop

control-plane
 service-policy input PM-COPP-ACCESS
exit

original idea of using block-for doesnt work
quiet-mode does not prevent .44 from still accessing the ssh as they are in the same vrf.

1.2
these commands do no exist. i dont need them tbh

1.3
step 8 is done but with bigBoyNTPkey as the key
1.3 is done for all routers and dsw, sw had no changes
does pa and asa need different ntp configs for this?


2 is done.
except for no permit icmp any 203.149.210.184 0.0.0.7 for R1

3.1 is done

4 is done.

5.1 done
ASA-1010(config)# show service-policy interface inside

Interface inside:
  Service-policy: PM_INSIDE_PROTECT
    Class-map: CM_SVR_PROTECT
      Set connection policy: conn-max 500 embryonic-conn-max 50 per-client-max 50 per-client-embryonic-max 10
        current embryonic conns 0, current conns 0, drop 0
      Set connection timeout policy:
        embryonic 0:00:15
        DCD: disabled, retry-interval 0:00:15, max-retries 5
        DCD: client-probe 0, server-probe 0, conn-expiration 0
ASA-1010(config)# show threat-detection statistics top tcp-intercept
Top 10 protected servers under attack (sorted by average rate)
Monitoring window size: 30 mins    Sampling interval: 30 secs

did not do 5.2, want to keep to my notes.

5.3
ASA-1010(config)# aaa local authentication attempts max-fail 5
ASA-1010(config)# aaa authorization command TACACS LOCAL
added both since both tacacs are up and running

5.4
ASA-1010(config)# show ntp status
Clock is synchronized, stratum 4, reference is 172.16.0.44
nominal freq is 99.9984 Hz, actual freq is 99.9983 Hz, precision is 2**6
reference time is ee9883f4.faf6586f (00:51:00.980 +08 Sat Nov 7 2026)
clock offset is 0.0228 msec, root delay is 9.26 msec
root dispersion is 15892.33 msec, peer dispersion is 15890.63 msec
ASA-1010(config)# show ntp associations detail

172.16.0.44 configured, our_master, sane, valid, stratum 16
ref ID 23.106.249.200, time ee98837d.1eaf9b1a (00:49:01.119 +08 Sat Nov 7 2026)
our mode client, peer mode server, our poll intvl 64, peer poll intvl 64
root delay 8.96 msec, root disp 1.69, reach 1, sync dist 15896.942
delay 0.31 msec, offset 0.0228 msec, dispersion 15890.63
precision 2**25, version 4
org time ee9883f4.faed8d58 (00:51:00.980 +08 Sat Nov 7 2026)
rcv time ee9883f4.faf6586f (00:51:00.980 +08 Sat Nov 7 2026)
xmt time ee9883f4.fade6e8f (00:51:00.979 +08 Sat Nov 7 2026)
filtdelay =     0.31    0.00    0.00    0.00    0.00    0.00    0.00    0.00
filtoffset =    0.02    0.00    0.00    0.00    0.00    0.00    0.00    0.00
filterror =    15.63 16000.0 16000.0 16000.0 16000.0 16000.0 16000.0 16000.0

172.16.0.45 configured, insane, invalid, stratum 16
ref ID 15.235.197.63, time ee9882c0.fc293427 (00:45:52.985 +08 Sat Nov 7 2026)
our mode client, peer mode server, our poll intvl 64, peer poll intvl 64
root delay 5.75 msec, root disp 2.03, reach 0, sync dist 22.247
delay 0.53 msec, offset 1032.4586 msec, dispersion 16000.00
precision 2**25, version 4
org time 00000000.00000000 (14:28:16.000 +08 Thu Feb 7 2036)
rcv time 00000000.00000000 (14:28:16.000 +08 Thu Feb 7 2036)
xmt time ee98841f.f239eb80 (00:51:43.946 +08 Sat Nov 7 2026)
filtdelay =     0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
filtoffset =    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
filterror =  16000.0 16000.0 16000.0 16000.0 16000.0 16000.0 16000.0 16000.0
ASA-1010(config)#

fails? 5.4

6.1 6.2 done
6.3 shows this
[edit]
bigDonkBonkAdmin@PA-440# set network profiles dos-protection DOS-PER-CLIENT type classified

Invalid syntax.
[edit]
bigDonkBonkAdmin@PA-440# set network profiles dos-protection DOS-PER-CLIENT resource sessions enabled yes

Invalid syntax.
[edit]
bigDonkBonkAdmin@PA-440# set network profiles dos-protection DOS-PER-CLIENT resource sessions max-concurrent-limit 50

Invalid syntax.
[edit]
bigDonkBonkAdmin@PA-440# set rulebase dos rules DOS-DMZ-SERVERS from zone outside

[edit]
bigDonkBonkAdmin@PA-440# set rulebase dos rules DOS-DMZ-SERVERS to zone dmz

[edit]
bigDonkBonkAdmin@PA-440# set rulebase dos rules DOS-DMZ-SERVERS source any destination [ WEB-SVR DNS-SVR JUMP-SVR ] source-user any service any

[edit]
bigDonkBonkAdmin@PA-440# set rulebase dos rules DOS-DMZ-SERVERS action protect

[edit]
bigDonkBonkAdmin@PA-440# set rulebase dos rules DOS-DMZ-SERVERS protection classified profile DOS-PER-CLIENT classification-criteria address source-ip-only

Server error :  protection -> classified -> profile 'DOS-PER-CLIENT' is not a valid reference
 protection -> classified -> profile is invalid
[edit]

6.4, 6.5, 6.6 done
i havent commited due to the invalids in 6.3

i dont understand the need for 7?
is it for verified ospf?

9 is done.

10 is done.