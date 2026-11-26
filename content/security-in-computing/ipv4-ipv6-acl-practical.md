---
id: ipv4-ipv6-acl-practical
title: "Practical 6: IPv4 and IPv6 ACLs for Remote Access Control and Attack Mitigation"
subject: security-in-computing
unit: 2
type: practical
practical: 6
syllabus: [UITVI-002]
status: draft
sources:
  - "Cisco, Security Configuration Guide: Access Control Lists, IPv4 and IPv6 (verify exact title for your IOS version)"
  - "Cisco Networking Academy, CCNA Security Packet Tracer activities: Configure IP ACLs to Mitigate Attacks; Configure IPv6 ACLs (verify)"
  - "RFC 2827 (BCP 38), Network Ingress Filtering"
---

## Aim

To restrict remote (SSH) management of a router to one administrator PC, to filter spoofed and unwanted ICMP traffic with an IPv4 ACL, and to configure and apply an IPv6 ACL.

## Concepts used

- [[extended-numbered-acl-practical]]: ACL syntax, wildcard masks, implicit deny.
- [[ssh-remote-access-practical]]: SSH on R3.
- [[threats-and-attacks]]: spoofing and reconnaissance (ping sweeps).

**Why this matters.** Even with SSH, anyone who can reach a router can try to guess its password. Limiting management to one admin PC removes that attack path for everyone else. Attackers also scan networks with ping sweeps and fake (spoof) source addresses to hide. Simple ACLs at the edge block both. Modern networks run IPv6 alongside IPv4, and an IPv4-only filter leaves the IPv6 path wide open.

## Before the lab

**Install or open**
- Cisco Packet Tracer 8.x or later.
- Your saved file **practical4.pkt** (no ACLs yet). Do **not** start from Practical 5.
- R1's console login: **Admin1 / admin1pa55**, enable secret **ciscoenpa55**. R3's SSH login: **SSHadmin / ciscosshpa55**.

**Revise first**
- ACL basics from [[extended-numbered-acl-practical]]: top-down order, implicit deny, wildcard masks.
- IPv6 addresses: written in hexadecimal, `::` replaces a run of zeros, `/64` is the usual LAN prefix.
- [[threats-and-attacks]]: what spoofing and a ping sweep are.

## Setup

**Tool:** Cisco Packet Tracer (8.x or later).

### Devices and topology

The Practical 4 topology. Three parts:

| Part | Router | What you configure |
|---|---|---|
| A | R3 | Standard ACL on the VTY lines: SSH only from PC-A |
| B | R1 | Extended ACL inbound from the WAN: block spoofed sources and incoming pings |
| C | All | IPv6 addresses and OSPFv3, then an IPv6 ACL on R3 |

### IPv6 addressing (Part C)

| Device | Interface | IPv6 address | Gateway |
|---|---|---|---|
| R1 | G0/0 / G0/1 | 2001:db8:1::1/64 / 2001:db8:12::1/64 | — |
| R2 | G0/1 / G0/2 | 2001:db8:12::2/64 / 2001:db8:23::2/64 | — |
| R3 | G0/1 / G0/0 | 2001:db8:23::1/64 / 2001:db8:3::1/64 | — |
| PC-A | NIC | 2001:db8:1::3/64 | 2001:db8:1::1 |
| Server | NIC | 2001:db8:1::5/64 | 2001:db8:1::1 |
| PC-C | NIC | 2001:db8:3::3/64 | 2001:db8:3::1 |

## Steps

### Part A. SSH to R3 only from PC-A

1. **Open your Practical 4 file and save a copy** as `practical6.pkt`.
   → **You should see:** the Practical 4 topology, all links green.
2. **Confirm SSH works from PC-C now.** PC-C: `ssh -l SSHadmin 192.168.3.1`.
   → **You should see:** `Password:` and then `R3>`. Type `exit`.
3. **Create a standard ACL that matches only PC-A.** On R3:
   ```text
   configure terminal
   access-list 10 permit host 192.168.1.3
   ```
   → **You should see:** no message.
4. **Apply it to the VTY lines.**
   ```text
   line vty 0 4
    access-class 10 in
   end
   ```
   → **You should see:** the prompt return to `R3#`.
5. **Test from PC-C.** `ssh -l SSHadmin 192.168.3.1`
   → **You should see:** `% Connection refused by remote host`.
6. **Test from PC-A.** `ssh -l SSHadmin 192.168.3.1`
   → **You should see:** `Password:`, then `R3>` after the password. Type `exit`.

### Part B. Edge filtering on R1

7. **Confirm the baseline.** PC-C: `ping 192.168.1.3`.
   → **You should see:** replies.
8. **Create the anti-spoofing lines of ACL 120 on R1.**
   ```text
   configure terminal
   access-list 120 remark Block spoofed sources and incoming pings
   access-list 120 deny ip 192.168.1.0 0.0.0.255 any
   access-list 120 deny ip 127.0.0.0 0.255.255.255 any
   ```
   → **You should see:** no message.
9. **Add the ICMP and permit lines.**
   ```text
   access-list 120 deny icmp any 192.168.1.0 0.0.0.255 echo
   access-list 120 permit ip any any
   ```
   → **You should see:** no message.
10. **Apply ACL 120 inbound on R1's WAN interface.**
    ```text
    interface g0/1
     ip access-group 120 in
    end
    ```
    → **You should see:** the prompt return to `R1#`.
11. **Test incoming ping.** PC-C: `ping 192.168.1.3`.
    → **You should see:** `Reply from 10.1.1.1: Destination host unreachable.` (or `Request timed out.`).
12. **Test outgoing ping.** PC-A: `ping 192.168.3.3`.
    → **You should see:** replies. Echo **replies** coming back are allowed.
13. **Check the matches.** On R1: `show access-lists`.
    → **You should see:** matches on the `deny icmp … echo` line.

### Part C. IPv6 and an IPv6 ACL

14. **Turn on IPv6 routing and OSPFv3 on R1.**
    ```text
    configure terminal
    ipv6 unicast-routing
    ipv6 router ospf 10
     router-id 1.1.1.1
    exit
    ```
    → **You should see:** no message.
15. **Add IPv6 addresses to R1's interfaces and put them in OSPFv3.**
    ```text
    interface g0/0
     ipv6 address 2001:db8:1::1/64
     ipv6 ospf 10 area 0
    interface g0/1
     ipv6 address 2001:db8:12::1/64
     ipv6 ospf 10 area 0
    end
    ```
    → **You should see:** no message yet.
16. **Repeat Steps 14–15 on R2** with router ID 2.2.2.2 and its addresses from the table.
    → **You should see:** `%OSPFv3-5-ADJCHG: Process 10, Nbr 1.1.1.1 on GigabitEthernet0/1 from LOADING to FULL, Loading Done`.
17. **Repeat Steps 14–15 on R3** with router ID 3.3.3.3 and its addresses.
    → **You should see:** `%OSPFv3-5-ADJCHG: Process 10, Nbr 2.2.2.2 on GigabitEthernet0/1 from LOADING to FULL, Loading Done`.
18. **Give the PCs and the server IPv6 addresses.** Desktop → IP Configuration → IPv6 Configuration → **Static**, using the table.
    → **You should see:** the IPv6 address and gateway saved on each device.
19. **Test IPv6.** PC-C: `ping 2001:db8:1::3`.
    → **You should see:** replies.
20. **Create the IPv6 ACL on R3.**
    ```text
    configure terminal
    ipv6 access-list BLOCK-PING-PCA
     deny icmp 2001:db8:3::/64 host 2001:db8:1::3
     permit ipv6 any any
    exit
    ```
    → **You should see:** the prompt `R3(config)#`.
21. **Apply it inbound on R3's LAN interface.**
    ```text
    interface g0/0
     ipv6 traffic-filter BLOCK-PING-PCA in
    end
    ```
    → **You should see:** the prompt return to `R3#`.
22. **Test the IPv6 ACL.** PC-C: `ping 2001:db8:1::3`, then `ping 2001:db8:1::5`.
    → **You should see:** the ping to PC-A fails; the ping to the server replies.
23. **Check the matches.** On R3: `show ipv6 access-list`.
    → **You should see:** matches on the `deny icmp` line.
24. **Save everything.** `copy running-config startup-config` on each router; File → Save.
    → **You should see:** `Building configuration... [OK]`.

## Code

**R3** (Parts A and C):

```text
access-list 10 permit host 192.168.1.3
line vty 0 4
 access-class 10 in
ipv6 unicast-routing
ipv6 router ospf 10
 router-id 3.3.3.3
interface g0/0
 ipv6 address 2001:db8:3::1/64
 ipv6 ospf 10 area 0
 ipv6 traffic-filter BLOCK-PING-PCA in
interface g0/1
 ipv6 address 2001:db8:23::1/64
 ipv6 ospf 10 area 0
ipv6 access-list BLOCK-PING-PCA
 deny icmp 2001:db8:3::/64 host 2001:db8:1::3
 permit ipv6 any any
```

**R1** (Parts B and C):

```text
access-list 120 remark Block spoofed sources and incoming pings
access-list 120 deny ip 192.168.1.0 0.0.0.255 any
access-list 120 deny ip 127.0.0.0 0.255.255.255 any
access-list 120 deny icmp any 192.168.1.0 0.0.0.255 echo
access-list 120 permit ip any any
ipv6 unicast-routing
ipv6 router ospf 10
 router-id 1.1.1.1
interface g0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 ospf 10 area 0
interface g0/1
 ip access-group 120 in
 ipv6 address 2001:db8:12::1/64
 ipv6 ospf 10 area 0
```

**R2** (Part C):

```text
ipv6 unicast-routing
ipv6 router ospf 10
 router-id 2.2.2.2
interface g0/1
 ipv6 address 2001:db8:12::2/64
 ipv6 ospf 10 area 0
interface g0/2
 ipv6 address 2001:db8:23::2/64
 ipv6 ospf 10 area 0
```

## Output

> [!warning] Replace with your own output
> The blocks below show what correct output looks like. After running the lab, paste your real output here, or save a screenshot in `content/assets/img/security-in-computing/` and link it with alt text.

**Test results**

| Test | Result |
|---|---|
| PC-A `ssh -l SSHadmin 192.168.3.1` | Login prompt; works |
| PC-C `ssh -l SSHadmin 192.168.3.1` | `% Connection refused by remote host` |
| PC-C `ping 192.168.1.3` | Destination host unreachable (blocked at R1) |
| PC-A `ping 192.168.3.3` | Replies (outgoing pings and their replies are allowed) |
| PC-C `ping 2001:db8:1::3` | Fails (blocked at R3) |
| PC-C `ping 2001:db8:1::5` | Replies |

**IPv6 ACL matches** (`show ipv6 access-list` on R3):

```output
IPv6 access list BLOCK-PING-PCA
    deny icmp 2001:DB8:3::/64 host 2001:DB8:1::3 (4 match(es)) sequence 10
    permit ipv6 any any (8 match(es)) sequence 20
```

## Explanation

### Part A: access-class on VTY lines

- ==A standard ACL matches only the source address==, which is all that is needed to say "only from PC-A".
- ==`access-class` applies an ACL to the router's own remote-access lines==, unlike `ip access-group`, which filters traffic passing through an interface.
- Even with the right password, SSH from any other address is refused.

### Part B: anti-spoofing and ICMP filtering

- `deny ip 192.168.1.0 0.0.0.255 any` inbound on the WAN side: ==packets arriving from outside should never carry R1's own LAN addresses as source==; if they do, they are spoofed.
- `deny ip 127.0.0.0 …`: loopback addresses must never appear on a real network.
- `deny icmp any 192.168.1.0 0.0.0.255 echo`: ==blocks ping sweeps into the LAN== while still allowing echo **replies** to pings that PC-A sends out.

### Part C: IPv6 ACLs

- ==IPv6 ACLs are always named==, use prefix lengths (`/64`) instead of wildcard masks, and are applied with `ipv6 traffic-filter`.
- Each IPv6 ACL ends with implicit **permits** for Neighbor Discovery (needed to find neighbours), then an implicit deny.
- IPv4 ACLs do not filter IPv6 traffic; ==dual-stack networks need both==.

> [!beyond] Beyond the syllabus
> Internet service providers apply the same anti-spoofing idea at scale (BCP 38): they drop packets whose source address could not have come from the customer's network, which makes spoofed DDoS attacks harder to launch.

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| `% Invalid input detected at '^' marker.` on `ip access-group 10 in` under `line vty 0 4` | Lines use `access-class`, not `access-group` | Use `access-class 10 in` |
| SSH from PC-A is also refused after Step 4 | The host address in ACL 10 is wrong | Check `show access-lists 10`; it must be `permit host 192.168.1.3`. Fix it from R3's console |
| PC-A's own pings to PC-C fail after Part B | ICMP was blocked too broadly (for example `deny icmp any any`) | Block only `echo` towards 192.168.1.0; echo-replies must still come in |
| `% IPv6 routing not enabled` | `ipv6 unicast-routing` was not entered first | Enter `ipv6 unicast-routing` in global configuration, then repeat the step |
| No `%OSPFv3-5-ADJCHG` message, IPv6 pings fail between routers | `ipv6 ospf 10 area 0` missing on an interface, or an address typo | Check `show ipv6 ospf neighbor` and `show ipv6 interface brief` on both routers |
| IPv6 ping from a PC fails before the ACL | Wrong IPv6 address or gateway on the PC | The gateway must be the router's address on that LAN (for PC-C, 2001:db8:3::1) |
| `% Invalid input detected` while typing the IPv6 ACL | A wildcard mask was used (`0.0.0.255` style) | IPv6 ACLs use prefixes: `2001:db8:3::/64` |

## Try it

1. Allow SSH to R3 from PC-A **and** from R2's interface 10.2.2.2. Which single line do you add to ACL 10?
2. Change the IPv6 ACL so that PC-C's LAN may ping PC-A but may not reach the server on HTTP (`deny tcp 2001:db8:3::/64 host 2001:db8:1::5 eq www`). Test with the browser at `http://[2001:db8:1::5]`.

## Viva questions

1. Difference between `access-class` and `ip access-group`? *`access-class` filters access to the router's VTY lines; `ip access-group` filters traffic through an interface.*
2. Why is a standard ACL enough for VTY restriction? *Only the source address of the admin PC needs checking.*
3. What is IP spoofing? *Faking the source address of packets to hide the attacker or impersonate a trusted host.*
4. Why block incoming ICMP echo but not echo-reply? *To stop outside ping sweeps while still letting inside users test connectivity outward.*
5. How do IPv6 ACLs differ from IPv4 ACLs? *Always named, use prefix lengths, applied with `ipv6 traffic-filter`, and have implicit permits for Neighbor Discovery.*
6. Does an IPv4 ACL filter IPv6 traffic? *No; each protocol needs its own ACL.*

## Sources

- Cisco, *Security Configuration Guide: Access Control Lists* for IOS, IPv4 and IPv6 (verify exact title for your IOS version).
- Cisco Networking Academy, CCNA Security Packet Tracer activities: Configure IP ACLs to Mitigate Attacks; Configure IPv6 ACLs (verify).
- RFC 2827 (BCP 38), *Network Ingress Filtering*.
