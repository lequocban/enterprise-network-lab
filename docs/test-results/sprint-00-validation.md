# Sprint 0 — Validation Evidence Register

**Date of review:** 2026-10-09  
**Evidence source:** user-supplied console output and HQ EVE-NG topology screenshot in project conversation.

> Important: This file is a **transcription/summary of supplied evidence**; it is not a fresh automated test executed against the user's live EVE-NG instance.

## TC-NET001-01 — EVE resource check

| Check | Command | Observed |
|---|---|---|
| Memory | `free -h` | total `7.7Gi`; available `6.3Gi`; swap `4.0Gi`, unused |
| Filesystem | `df -h /` | root FS `116G`, `12G` used, `100G` available (11% used) |
| vCPU | `nproc` | `4` |
| Virtualization flags | `egrep -c '(vmx|svm)' /proc/cpuinfo` | `4` |

**Result:** PASS for basic resource visibility and virtualization flags; actual peak-load stability requires later observation. Original attempt `df -f /` was corrected to `df -h /`.

## TC-NET001-02 — Router-to-router connectivity

- R1 `Gi0/0`: `10.255.0.1/30`; R2 address `10.255.0.2/30`.
- R1 interface messages indicated link and protocol **up**.
- Observed `ping 10.255.0.2`: `!!!!.` or `.!!!!` pattern (four successes, one timeout), **success rate 80% (4/5)**, RTT min/avg/max **1/1/2 ms**.
- `write memory` returned `[OK]` on R1.

**Result:** PASS for established L3 reachability. **Not confirmed**: later 10/10 ping or reverse R2→R1 output. A missed first probe can be consistent with ARP resolution but cannot be proven from this result alone.

## TC-NET002-01 — Core SVI / connected route

CORE1 observed:

```text
Vlan10  10.10.10.2  YES NVRAM  up  up
VLAN 10 MGMT active with Et1/0
C 10.10.10.0/24 is directly connected, Vlan10
L 10.10.10.2/32 is directly connected, Vlan10
```

**Result:** PASS for CORE1 VLAN10 SVI and connected prefix. Earlier `down/down` was resolved after adding an active VLAN 10 access port.

## TC-NET002-02 — HSRP adjacency & role election

**CORE1:**

```text
Interface Grp Pri P State  Active Standby     Virtual IP
Vl10       10  110 P Active local 10.10.10.3 10.10.10.1
```

**CORE2:**

```text
Interface Grp Pri P State  Active      Standby Virtual IP
Vl10       10  100 P Standby 10.10.10.2 local 10.10.10.1
```

Additionally, CORE2 `Vlan10 10.10.10.3` and CORE1 `Vlan10 10.10.10.2` were `up/up`.

**Result:** **PASS — verified from live CLI output supplied by user.** Both HSRP peers see each other and agree on a single active node and VIP.

## TC-NET002-03 — HSRP switchover

**Procedure proposed:** Shut SVI VLAN 10 on CORE1; verify CORE2 Active; restore CORE1 with preempt; verify CORE1 Active again.

**Evidence:** User subsequently confirmed “đã xong net 002 và mọi thứ đều ổn”. No timestamped before/during/after `show standby brief` log was supplied.

**Result:** **REPORTED PASS / evidence to archive.** This is a role-switchover test, **not** a demonstrated client traffic failover under complete CORE1 power loss.

## Not yet verified (not Sprint 0 PASS claims)

- PC-MGMT ping to 10.10.10.1 in an archived output file.
- Inter-VLAN routing for VLANs 20/30/40/50/60.
- RSTP root election and uplink failure/convergence.
- LACP EtherChannel operation.
- ASAv and Alpine console/service tests archived in the repository.
- WAN, OSPF, BGP, VPN, NAT, firewall enforcement, monitoring, automation.

## Recommended screenshots / captures

- `diagrams/hq-campus-topology.svg` — reconstructed from supplied screenshot; original screenshot should also be archived.
- `screenshots/sprint-00-eve-resources.png` — pending.
- `screenshots/sprint-00-router-ping.png` — pending.
- `screenshots/sprint-00-hsrp-active-standby.png` — pending.
- `screenshots/sprint-00-hsrp-switchover.png` — pending.