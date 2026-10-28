# Punggol Bank Network — Handover Document

**Rack:** 4E (`gruppesechs.sitict.net`)
**Topology:** Full PTSO (Parallel Topologically, Serial Operationally)
**Primary goal this session:** Achieve end-to-end internet access for HR/IT users through the serial security pipeline.
**Status at handover:** HR (and IT, same policy) can reach the internet (verified `ping 8.8.8.8` succeeds from the HR host). Internal serial path confirmed working end to end.

---

## 1. Data flow pipeline (normal operation)

```
End Users (HR/IT)
  -> Access Switches (SW3 = HR / SW4 = IT)
  -> Distribution Switches (DSW1 / DSW2, HSRP)
  -> Firepower 1010 ASA  (inside -> outside, stateful inspection)
  -> PA-440 NGFW         (inside -> outside, App-ID + outbound PAT)
  -> Edge Router R1
  -> ISP1 gateway (172.17.9.94)
  -> Internet
```

Outbound internet traffic from HR/IT is **PAT-translated once, on the PA-440**, to the public address `203.149.210.185`. The ASA does **not** NAT in the normal path. R1 does **not** NAT — it only routes.

---

## 2. Rack 4E addressing (verified against ISP appendix formula)

Formula for Rack 4E: row `r=4`, column `E=5`, so `n=45`; `f = n - 4r - 6 = 45 - 16 - 6 = 23`.

| Item | Value |
|------|-------|
| WAN link block (R1 <-> ISP1) | `172.17.9.92/30` |
| R1 WAN IP (G0/0/0) | `172.17.9.93` |
| ISP1 gateway | `172.17.9.94` |
| Public NAT block | `203.149.210.184/29` |
| Outbound PAT address (PUB-PAT) | `203.149.210.185` |

> NOTE: These values are **correct for Rack 4E**. The WAN `/30` (`172.17.9.92/30`) is a point-to-point link and is **NOT internet-routable** — the ISP only routes the public `/29` to the internet. This is why `R1# ping 8.8.8.8` FAILS (sourced from `172.17.9.93`) but an HR host PAT'd to `203.149.210.185` SUCCEEDS. R1 failing to ping 8.8.8.8 is EXPECTED and is not a fault.

### Key internal subnets / addresses referenced below
- HR VLAN 61: `172.16.0.56/29`, VIP `.57`, DSW1 SVI `.58`, DSW2 SVI `.59`, test host `.60`
- IT VLAN 67: `172.16.0.64/29`
- Inside transit (DSW <-> ASA/PA, VLAN 100): `172.16.0.96/29` — ASA inside `.97`, DSW1 `.98`, DSW2 `.99`, PA vlan.100 `.100`
- PA <-> ASA serial link: `172.16.0.104/30` — PA eth1/3 `.105`, ASA E1/1 `.106`
- Routers <-> PA (VLAN 10 on PA): `172.16.0.88/29` — R1 `.90`, R2 `.91`, PA vlan.10 `.92`

---

## 3. Root causes found and fixed this session (in order discovered)

Each item below is a **real defect** that was blocking HR internet access. All fixes have been folded into the `.ttl` macro files (see Section 5) unless noted as "live only".

### 3.1 DSW ACLs too tight (initial concern)
- Added explicit `permit icmp 172.16.0.56 0.0.0.7 any` to `ACL_HR_IN` and `permit icmp 172.16.0.64 0.0.0.7 any` to `ACL_IT_IN`, placed before the anti-smurf `deny ip ... 172.16.0.0 0.0.0.127` line, on **both DSW1 and DSW2**. (Diagnostic aid; the final `permit ip ... any` already covered ICMP.)

### 3.2 PA-440 `HR-IT-INTERNET` App-ID too strict (bring-up)
- Temporarily loosened to `application any service any` during bring-up so sessions weren't dropped before App-ID identification.
- **Re-tightened** in the macro back to `application [ web-browsing ssl dns ping ] service application-default`.
- STATUS: the macro is tightened; the LIVE PA may still be loose (see Section 6 — not yet re-applied live at handover).

### 3.3 PA-440 additive/stale rulebase
- PAN-OS `set` config is additive across runs. Prior runs left stale rules (`DMZ-WEB-IN`, `DMZ-DNS-IN`, old `ALLOW-OSPF`, duplicate `PTSO-DHCP-RELAY`) and left `INBOUND-WEB`/`INBOUND-DNS` sitting **below** `DENY-ALL` (so they never matched).
- Created `pa440_cleanup.ttl` to delete all rules/NAT/objects by name for a clean slate.
- Re-ran main macro → rulebase now correct and in order, `DENY-ALL` last.

### 3.4 PA-440 `NAT-OUTBOUND` missing `from`/`to`/`source`
- The macro line only set `source-translation`, relying on a prior run for the rest → commit failed: `NAT-OUTBOUND is missing 'from'`.
- Fixed the macro line to include `from [ inside server dmz ] to outside source [ HR-NET IT-NET TXN-SVR JUMP-SVR JUMP2-SVR ] destination any service any source-translation dynamic-ip-and-port translated-address PUB-PAT`.

### 3.5 PA-440 `DSWs` address-group stale members
- Group had accumulated stale members `DSW1-IP`/`DSW2-IP` (from an old macro) alongside the intended `DSW1-SVI`/`DSW2-SVI`.
- Fixed live by `delete address-group DSWs static DSW1-IP` / `DSW2-IP`. The macro defines the group correctly as `[ DSW1-SVI DSW2-SVI ]`. The extended `pa440_cleanup.ttl` now also removes the stale objects.

### 3.6 SW3 Dynamic ARP Inspection dropping the static HR host  ⭐ (first real blocker)
- Symptom: `%SW_DAI-4-DHCP_SNOOPING_DENY ... Gi1/0/5, vlan 61 ([74d4.dd81.5ca2/172.16.0.60/.../172.16.0.57/...])`
- Cause: HR host uses a **static IP**, so DAI had no DHCP-snooping binding and dropped its ARP → it could never resolve its gateway.
- Fix (in `sw3.ttl`):
  ```
  arp access-list STATIC-HR
   permit ip host 172.16.0.60 mac host 74d4.dd81.5ca2
  ip arp inspection filter STATIC-HR vlan 61
  ip source binding 74d4.dd81.5ca2 vlan 61 172.16.0.60 interface GigabitEthernet1/0/5
  ```
  (ARP ACL fixes DAI; `ip source binding` fixes IP Source Guard / `ip verify source` for the static host.)

### 3.7 ASA factory default NAT (`obj_any`) hijacking HR traffic  ⭐
- Packet-tracer showed HR being PAT'd to the ASA's own outside IP (`172.16.0.106`) by `object network obj_any / nat (any,outside) dynamic interface` — breaks the serial design (PA must do the single PAT).
- Fix (in `firepower1010.ttl`, added to cleanup section): `object network obj_any` then `no nat (any,outside) dynamic interface`.
- NOTE: the `obj_any` object itself still exists live (harmless with the NAT removed); the macro removes the NAT on re-run.

### 3.8 ASA OSPF `ip_multicast_ctl failed to get channel 6/7`  ⭐
- Cause: on the Firepower 1010 ASA, OSPF cannot join multicast groups on VLAN SVIs backed by switchports. `Vlan69` (server, E1/6) and `Vlan92` (dmz, E1/7) are such SVIs. The error fired even on `clear configure router ospf` (teardown), ruling out paste-speed/scale.
- An earlier attempt using `passive-interface server` / `passive-interface dmz` FAILED — that ASA build rejects IOS-style `passive-interface <nameif>`.
- Fix (in `firepower1010.ttl`): run OSPF only on the 3 routed physical transit links (`inside` E1/3 `172.16.0.96/29`, `outside` E1/1 `172.16.0.104/30`, `wan-backup` E1/2 `172.16.0.112/30`) and advertise server/dmz subnets via `redistribute connected subnets` instead of `network` statements.
- Verified: neighbors to R2/PA/DSW1 reach FULL; DSW2 stays `2WAY/DROTHER` (NORMAL on a broadcast segment — the ASA only goes FULL with DR/BDR); no multicast errors.

### 3.9 ASA command-authorization lockout  ⭐
- `aaa authorization command TACACS LOCAL` was enabled, but the TACACS servers (`.44`/`.45` = VM102/VM106) are not built yet. With TACACS unreachable, per-command authorization fell back to LOCAL and DENIED everything (even `exit`, `show conn`).
- Recovery: removed the line live (`no aaa authorization command TACACS LOCAL`).
- Fix (in `firepower1010.ttl`): the `aaa authorization command TACACS LOCAL` line is **commented out** with instructions to re-enable ONLY after TACACS is built and verified, and with a console session open as a safety net.

### 3.10 DSW default route bypassing the ASA  ⭐
- DSW default `0.0.0.0/0` was an OSPF-learned external route via `172.16.0.100` (PA vlan.100 bypass), sending HR traffic straight to the PA and skipping the ASA — breaking serial inspection and causing return-path asymmetry.
- Fix (in `dsw1.ttl` and `dsw2.ttl`): tracked static default forcing the serial path through the ASA:
  ```
  ip sla 10
   icmp-echo 172.16.0.97 source-interface Vlan100
   frequency 5
  ip sla schedule 10 life forever start-time now
  track 10 ip sla 10 reachability
  ip route 0.0.0.0 0.0.0.0 172.16.0.97 track 10
  ```
  Static (AD 1) beats the OSPF default (AD 110) in normal operation; if the ASA (.97) becomes unreachable, the tracked static withdraws and the OSPF backup via the PA (.100) takes over.

### 3.11 PA-440 Layer-2 loop on VLAN 100 (bypass ports)  ⭐
- The PA bypass ports `eth1/6` (DSW1 G1/0/12) and `eth1/7` (DSW2 G1/0/12) were UP and bridged into `vlan.100` (VL-INSIDE-BYPASS). Combined with the DSW1<->DSW2 Po5 trunk (also VLAN 100), this formed an L2 loop. The PA runs no STP on these L2 ports. Symptom: duplicate `inside-l2 / intrazone-default` sessions flooding eth1/7<->eth1/6, MAC flapping, intermittent loss of L3 reachability on the inside segment.
- The design intends these as FAILOVER-ONLY ports (admin-down in normal operation). The macro had dropped the `link-state down`.
- Fix (in `pa440.ttl`): `set network interface ethernet ethernet1/6 link-state down` (and `1/7`, `1/8`).
- WARNING: do NOT manually bring eth1/6/7 up during normal operation or the loop returns.

### 3.12 DSW1 <-> DSW2 Po5 trunk physically down  ⭐
- `Po5(SD)`, members `Gi1/0/9(D) Gi1/0/10(D)`, `notconnect`. This is a **physical cabling** problem (not config). It was the long-standing item from the original handoff.
- Impact: DSW2 is HSRP **active** for VLAN 61 (priority 110), so HR's gateway is DSW2. DSW2 reaches the ASA inside (.97) ONLY via Po5 -> DSW1 (because ASA E1/4, the direct DSW2 link, is kept shutdown). With Po5 down, DSW2 had no path to the ASA, so its static default via .97 would not install (`% Subnet not in table`), and HR got "destination host unreachable" (replied by DSW2 .59).
- Fix: RESEAT/REPLACE the cables between DSW1 Gi1/0/9–10 and DSW2 Gi1/0/9–10. After the cable was fixed, Po5 came up, DSW2 reached .97, the static default installed, and HR could forward.

### 3.13 R1 default route pointing at itself  ⭐ (final blocker)
- R1's running config had `ip route 0.0.0.0 0.0.0.0 172.17.9.93 track 1` — `.93` is **R1's own WAN interface IP**, not the ISP gateway. This was the ACTIVE (`*`) default; traffic to the internet went nowhere.
- There were THREE default routes present live: `...172.17.9.93 track 1`, `...172.17.9.94 track 1`, `...172.17.9.94 250`.
- Fix (live): `no ip route 0.0.0.0 0.0.0.0 172.17.9.93 track 1`. After removal, the correct `* 172.17.9.94` (track 1) became active.
- The macro `r1.ttl` was ALWAYS correct (`ip route 0.0.0.0 0.0.0.0 172.17.9.94 track 1`); the `.93` was a live typo only. A clean re-run of `r1.ttl` produces the correct route.
- The leftover `ip route 0.0.0.0 0.0.0.0 172.17.9.94 250` (floating, same next-hop) is harmless and NOT in the macro; a reload will not recreate it. Left in place.

---

## 4. Why `ping 8.8.8.8` behaves the way it does (important — avoid re-diagnosing)

- `R1# ping 8.8.8.8` FAILS. This is EXPECTED: R1 sources from its WAN IP `172.17.9.93`, which is on the point-to-point `/30` that the ISP does not route to the internet.
- HR host `ping 8.8.8.8` SUCCEEDS: HR is PAT'd to `203.149.210.185` (public `/29`), which the ISP routes to the internet and back.
- A peer rack confirmed real internet is reachable, so this is NOT a closed/simulated-internet limitation (an earlier hypothesis that was disproven).
- Pinging `8.8.8.8` by IP needs NO DNS. The DHCP/DNS servers being down does not affect it. Name-based access (e.g. `ping google.com`) WILL need the internal DNS server up with upstream forwarders.
- Do NOT test against `172.16.0.94` — that address does not exist (unused host in `172.16.0.88/29`). Valid test targets: `172.16.0.90` (R1), `172.17.9.94` (ISP gw), `8.8.8.8` (internet, from an HR/IT host only).

### PAN-OS ping gotcha
On the PA, `ping host <ip>` without a source uses the **management interface** and will fail for data-plane destinations. Always use `ping source 172.16.0.92 host <ip>` to test from the data plane.

---

## 5. Config files and what each contains

Location: `c:\Users\asus\Downloads\configs\`

| File | Device | Notes |
|------|--------|-------|
| `plan.md` | — | Original design document |
| `r1.ttl` | Edge Router R1 | Default route via `.94 track 1` (correct). WAN-IN ACL. IP SLA 1 (ISP) + SLA 2 (PA .92). |
| `r2.ttl` | Router R2 | Redundant transit / PTSO bypass |
| `dsw1.ttl` | DSW1 | Tracked static default via `.97` (SLA/track 10) added. ICMP permit in ACL_HR_IN/ACL_IT_IN. |
| `dsw2.ttl` | DSW2 | Same tracked static default. HSRP active for VLAN 61/67. |
| `sw1.ttl`–`sw4.ttl` | Access switches | SW3 = HR (has STATIC-HR DAI ACL + ip source binding). SW4 = IT. |
| `firepower1010.ttl` | ASA-1010 | obj_any NAT removed; OSPF on 3 transit links + redistribute connected; command-auth commented out; clock `Singapore`. |
| `pa440.ttl` | PA-440 | HR-IT-INTERNET tightened; eth1/6/7/8 link-state down; NAT-OUTBOUND complete; ALLOW-OSPF + PTSO-DHCP-RELAY + DSWs/UDP-67 objects. |
| `pa440_cleanup.ttl` | PA-440 | Deletes all rules/NAT/address-groups/addresses/services for a clean slate before re-running `pa440.ttl`. |

### Tera Term macro notes
- PA macro waits on `[edit]` (not `#`) — `#` appears in the login banner and mid-output and caused premature sends / command collisions.
- Long PAN-OS lines are sent in full by `sendln` even though the terminal wraps/overwrites the display. Garbled on-screen != corrupted on the wire. Collisions came from premature `wait` matches, now mitigated by `[edit]`.

---

## 6. CURRENT LIVE STATE vs MACROS (drift at handover)

| Item | Macro | Live device | Action needed |
|------|-------|-------------|---------------|
| PA `HR-IT-INTERNET` | tightened (`web-browsing ssl dns ping`) | still LOOSE (`application any service any`) | Apply tighten live, or run cleanup+`pa440.ttl` |
| R1 floating default `...94 250` | absent | present (harmless) | none |
| ASA `obj_any` object | re-entered, NAT removed | object exists, NAT already removed | none |
| Everything else | — | matches | none |

### To apply the PA tighten live (if not doing a full re-run):
```
configure
delete rulebase security rules HR-IT-INTERNET application
delete rulebase security rules HR-IT-INTERNET service
set rulebase security rules HR-IT-INTERNET application [ web-browsing ssl dns ping ] service application-default
commit
```
Then verify HR internet (web + DNS + ping) still works. If a needed app is missing, widen the app list.

---

## 7. RELOAD / REBUILD PROCEDURE  (CRITICAL)

> `write memory` is NOT permitted (per lab instructions). Therefore:
> - **PA-440 PERSISTS** its committed config across reloads.
> - **All Cisco devices (R1, R2, DSW1, DSW2, SW1–4, ASA) WIPE to startup on reload** and must be rebuilt by re-running their `.ttl` macros.

| Device | Persists? | After reload |
|--------|-----------|--------------|
| PA-440 | YES | Nothing required. For a clean/tightened rebuild: run `pa440_cleanup.ttl` then `pa440.ttl`. |
| ASA-1010 | NO | Re-run `firepower1010.ttl` |
| R1 | NO | Re-run `r1.ttl` |
| R2 | NO | Re-run `r2.ttl` |
| DSW1 | NO | Re-run `dsw1.ttl` |
| DSW2 | NO | Re-run `dsw2.ttl` |
| SW1–SW4 | NO | Re-run respective macros (SW3 includes the DAI static-host fix) |

### Non-config dependencies that a macro CANNOT restore
1. **Po5 cabling** — DSW1 Gi1/0/9–10 <-> DSW2 Gi1/0/9–10 must be physically connected, or DSW2 (HR's active gateway) cannot reach the ASA. Verify `show etherchannel summary` shows `Po5(SU)`.
2. **PA bypass ports eth1/6/7 must stay DOWN** in normal operation (the macro sets this). Do not manually bring them up or the VLAN 100 L2 loop returns.
3. **R1 default route must be `.94`, not `.93`** — the macro is correct; just don't hand-type `.93`.

---

## 8. OUTSTANDING / NOT YET DONE

- **Apply PA `HR-IT-INTERNET` tighten live** (see Section 6) — currently loose on the box.
- **Servers not built:** DHCP (Proxmox `.77`), DNS (VM101 `.83`, needs upstream forwarders for name resolution), File (VM103), Accounts (VM104), Transactions (VM105). TACACS1 (`.44`) and TACACS2 (`.45`) are UP.
- **DNS recursion:** internal DNS must forward to upstream (8.8.8.8 / 1.1.1.1) for internet name resolution; IP-based internet access already works.
- **Re-enable ASA command authorization** (`aaa authorization command TACACS LOCAL`) ONLY after TACACS is built and verified, with a console session open. It is currently commented out in the macro to prevent lockout.
- **IT VLAN end-to-end test** — IT uses the same `HR-IT-INTERNET` / `INTERNET-USERS` policy as HR; verify from an IT host.
- **Second HR host** — the SW3 DAI `STATIC-HR` ACL + `ip source binding` currently covers only the one test host (`172.16.0.60` / MAC `74d4.dd81.5ca2`). Add a matching pair for the 2nd HR host when present.
- **TXN-INTERNET-TIMED window** — final time-range to be confirmed on demo day (currently weekdays 01:00–03:00).

---

## 9. SECURITY NOTE
The `.ttl` files contain cleartext credentials (enable/user passwords, TACACS keys). If committing to GitHub, use a **PRIVATE** repo or scrub the secrets first.

---

## 10. VERIFICATION SNAPSHOT (at handover)
- HR host `172.16.0.60` -> `8.8.8.8`: SUCCESS (PAT'd to `203.149.210.185`).
- PA default route: `0.0.0.0/0 via 172.16.0.90` (R1), active, out vlan.10.
- PA OSPF neighbors: R1 (.90), R2 (.91), ASA (.97 and .106), DSW1 (.98), DSW2 (.99) — all FULL.
- ASA OSPF neighbors: R2/PA/DSW1 FULL; DSW2 2WAY/DROTHER (normal); no multicast errors.
- R1 default: `* 0.0.0.0/0 via 172.17.9.94` (track 1). `R1# ping 8.8.8.8` fails by design (WAN /30 not internet-routable).
- DSW default: static via `172.16.0.97` (ASA), tracked.
- Po5: UP after cable reseat.
- PA bypass eth1/6/7: DOWN (loop broken).
