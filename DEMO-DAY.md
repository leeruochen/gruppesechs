# Demo Day Checklist

**Last updated:** 6 November 2026

Things that can only be done once the final details are given on the day.

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
