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

## Setup

**Tool:** Cisco Packet Tracer (8.x or later).

### Devices and topology

Open your saved file from **Practical 4** (no ACLs yet). Same routers, PCs and server.

R1's console now asks for a login (Practical 4): use **Admin1 / admin1pa55**.

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

## Code

### Part A. SSH to R3 only from PC-A

```text
R3# configure terminal
R3(config)# access-list 10 permit host 192.168.1.3
R3(config)# line vty 0 4
R3(config-line)# access-class 10 in
R3(config-line)# end
```

### Part B. Edge filtering on R1 (traffic arriving from R2)

```text
R1# configure terminal
R1(config)# access-list 120 remark Block spoofed sources and incoming pings
R1(config)# access-list 120 deny ip 192.168.1.0 0.0.0.255 any
R1(config)# access-list 120 deny ip 127.0.0.0 0.255.255.255 any
R1(config)# access-list 120 deny icmp any 192.168.1.0 0.0.0.255 echo
R1(config)# access-list 120 permit ip any any
R1(config)# interface g0/1
R1(config-if)# ip access-group 120 in
R1(config-if)# end
```

### Part C1. IPv6 addresses and OSPFv3 (R1 shown; repeat on R2 and R3 with their addresses)

```text
R1(config)# ipv6 unicast-routing
R1(config)# ipv6 router ospf 10
R1(config-rtr)# router-id 1.1.1.1
R1(config-rtr)# exit
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:db8:1::1/64
R1(config-if)# ipv6 ospf 10 area 0
R1(config-if)# interface g0/1
R1(config-if)# ipv6 address 2001:db8:12::1/64
R1(config-if)# ipv6 ospf 10 area 0
```

Use router IDs 2.2.2.2 on R2 and 3.3.3.3 on R3. Set the PC and server IPv6 addresses in Desktop → IP Configuration, then check PC-C can ping 2001:db8:1::3.

### Part C2. IPv6 ACL on R3: PC-C's LAN may not ping PC-A

```text
R3(config)# ipv6 access-list BLOCK-PING-PCA
R3(config-ipv6-acl)# deny icmp 2001:db8:3::/64 host 2001:db8:1::3
R3(config-ipv6-acl)# permit ipv6 any any
R3(config-ipv6-acl)# exit
R3(config)# interface g0/0
R3(config-if)# ipv6 traffic-filter BLOCK-PING-PCA in
R3(config-if)# end
```

### Verify

```text
PC-A> ssh -l SSHadmin 192.168.3.1
PC-C> ssh -l SSHadmin 192.168.3.1
PC-C> ping 192.168.1.3
PC-A> ping 192.168.3.3
PC-C> ping 2001:db8:1::3
PC-C> ping 2001:db8:1::5
R1# show access-lists
R3# show ipv6 access-list
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
