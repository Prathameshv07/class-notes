---
id: prac-ospf-md5-authentication
title: "Practical 1: OSPF MD5 Authentication"
subject: security-in-computing
unit: 1
type: practical
practical: 1
syllabus: [UITVI-002]
status: draft
sources:
  - "Cisco, IP Routing: OSPF Configuration Guide, 'OSPF message digest (MD5) authentication' (verify exact chapter title for your IOS version)"
  - "Cisco Networking Academy, CCNA Security lab: Securing the Router for Administrative Access / OSPF authentication (verify)"
  - "RFC 2328, OSPF Version 2, Appendix D (Authentication)"
---

## Aim

To configure OSPF routing between three routers and secure it with MD5 authentication, so that routers accept routing updates only from neighbours that know the shared key.

## Concepts used

- [[authentication]]: proving identity with a shared secret (Unit 2 theory).
- [[network-device-security]]: hardening routers and switches (Unit 3 theory).
- OSPF basics from your Sem 5 networking course: neighbours, areas, `network` statements.

**Why this matters.** OSPF routers believe whatever routes their neighbours advertise. An attacker who plugs a laptop or rogue router into the network can send fake OSPF updates and redirect traffic through their machine. With MD5 authentication, every OSPF packet carries a hash made with a secret key, so packets from anyone without the key are dropped.

## Setup

**Tool:** Cisco Packet Tracer (8.x or later). GNS3 with a Cisco IOS image also works with the same commands.

### Devices and topology

Three 2911 routers (R1, R2, R3), two 2960 switches, two PCs. Connect routers to each other with copper cross-over cables; PCs to switches and switches to routers with copper straight-through cables.

```mermaid
flowchart LR
  accTitle: Topology for OSPF MD5 authentication
  accDescr: PC-A connects through switch S1 to router R1. R1 connects to R2, R2 connects to R3, and R3 connects through switch S3 to PC-C. All three routers are in OSPF area 0.
  PCA[PC-A 192.168.1.3] --- S1[S1] --- R1[R1]
  R1 ---|10.1.1.0/30| R2[R2]
  R2 ---|10.2.2.0/30| R3[R3]
  R3 --- S3[S3] --- PCC[PC-C 192.168.3.3]
```

### Addressing table

| Device | Interface | IP address | Subnet mask | Default gateway |
|---|---|---|---|---|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | — |
| R1 | G0/1 | 10.1.1.1 | 255.255.255.252 | — |
| R2 | G0/1 | 10.1.1.2 | 255.255.255.252 | — |
| R2 | G0/2 | 10.2.2.2 | 255.255.255.252 | — |
| R3 | G0/1 | 10.2.2.1 | 255.255.255.252 | — |
| R3 | G0/0 | 192.168.3.1 | 255.255.255.0 | — |
| PC-A | NIC | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |
| PC-C | NIC | 192.168.3.3 | 255.255.255.0 | 192.168.3.1 |

Set the PC addresses in Desktop → IP Configuration.

## Code

### Step 1. Basic settings and OSPF on R1

```text
enable
configure terminal
hostname R1
no ip domain-lookup
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface g0/1
 ip address 10.1.1.1 255.255.255.252
 no shutdown
exit
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 passive-interface g0/0
end
```

### Step 2. Basic settings and OSPF on R2

```text
enable
configure terminal
hostname R2
no ip domain-lookup
interface g0/1
 ip address 10.1.1.2 255.255.255.252
 no shutdown
interface g0/2
 ip address 10.2.2.2 255.255.255.252
 no shutdown
exit
router ospf 1
 router-id 2.2.2.2
 network 10.1.1.0 0.0.0.3 area 0
 network 10.2.2.0 0.0.0.3 area 0
end
```

### Step 3. Basic settings and OSPF on R3

```text
enable
configure terminal
hostname R3
no ip domain-lookup
interface g0/0
 ip address 192.168.3.1 255.255.255.0
 no shutdown
interface g0/1
 ip address 10.2.2.1 255.255.255.252
 no shutdown
exit
router ospf 1
 router-id 3.3.3.3
 network 192.168.3.0 0.0.0.255 area 0
 network 10.2.2.0 0.0.0.3 area 0
 passive-interface g0/0
end
```

Before going on, check that PC-A can ping PC-C. OSPF works, but without any authentication yet.

### Step 4. Enable MD5 authentication on all three routers

On **R1**:

```text
configure terminal
interface g0/1
 ip ospf message-digest-key 1 md5 MD5pa55
exit
router ospf 1
 area 0 authentication message-digest
end
```

On **R2** (two OSPF links, so the key goes on both):

```text
configure terminal
interface g0/1
 ip ospf message-digest-key 1 md5 MD5pa55
interface g0/2
 ip ospf message-digest-key 1 md5 MD5pa55
exit
router ospf 1
 area 0 authentication message-digest
end
```

On **R3**:

```text
configure terminal
interface g0/1
 ip ospf message-digest-key 1 md5 MD5pa55
exit
router ospf 1
 area 0 authentication message-digest
end
```

### Step 5. Verify

```text
R1# show ip ospf interface g0/1
R2# show ip ospf neighbor
R1# show ip route ospf
PC-A> ping 192.168.3.3
```

## Output

> [!warning] Replace with your own output
> The blocks below show what correct output looks like. After running the lab, paste your real output here, or save a screenshot in `content/assets/img/security-in-computing/` and link it with alt text.

**Authentication active on the interface** (`show ip ospf interface g0/1` on R1, last lines):

```output
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 10.1.1.1/30, Area 0
  Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1
  ...
  Message digest authentication enabled
    Youngest key id is 1
```

**Both neighbours back in FULL state** (`show ip ospf neighbor` on R2):

```output
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:33    10.1.1.1        GigabitEthernet0/1
3.3.3.3           1   FULL/DR         00:00:37    10.2.2.1        GigabitEthernet0/2
```

DR and BDR roles may differ in your lab, depending on which router started first. What matters is `FULL`.

**End-to-end reachability** (PC-A command prompt):

```output
C:\>ping 192.168.3.3

Pinging 192.168.3.3 with 32 bytes of data:

Reply from 192.168.3.3: bytes=32 time=1ms TTL=125
Reply from 192.168.3.3: bytes=32 time=1ms TTL=125
Reply from 192.168.3.3: bytes=32 time=1ms TTL=125
Reply from 192.168.3.3: bytes=32 time=1ms TTL=125

Ping statistics for 192.168.3.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

## Explanation

### What Steps 1–3 do

- `hostname`, `no ip domain-lookup`: name the router; stop it from treating a typo as a hostname to look up.
- `ip address … / no shutdown`: address each interface and turn it on.
- `router ospf 1`: start OSPF process 1. The number is local to the router.
- `router-id`: give each router a fixed, easy-to-read identity.
- `network … area 0`: run OSPF on interfaces in that range. The second value is a **wildcard mask** (0.0.0.3 matches a /30).
- `passive-interface g0/0`: advertise the LAN, but send no OSPF packets towards PCs. A PC on that LAN cannot then form a fake neighbourship.

### What Step 4 does

- `ip ospf message-digest-key 1 md5 MD5pa55`: on this interface, key number 1 with the secret `MD5pa55`. ==The key ID and the key must match on both ends of a link.==
- `area 0 authentication message-digest`: every interface in area 0 must use MD5 authentication.

Before sending, the router computes an MD5 hash over the OSPF packet plus the secret key and attaches it. The receiver recomputes the hash with its own key. If the two hashes differ, the packet is dropped. ==The key itself is never sent over the link.==

### What you will notice while configuring

As soon as R1 has authentication and R2 does not, their neighbourship goes down after the dead timer (40 seconds). Routes disappear and pings fail. When R2 is configured too, the neighbourship returns to FULL. This is the security feature working: a router without the right key is ignored.

> [!beyond] Beyond the syllabus
> MD5 is now considered weak. Newer IOS versions support OSPFv2 authentication with HMAC-SHA using a key chain (`key chain`, `cryptographic-algorithm hmac-sha-256`). Packet Tracer may not support it; real networks should prefer it.

## Try it

Change the key on R3 only:

```text
R3(config)# interface g0/1
R3(config-if)# no ip ospf message-digest-key 1 md5 MD5pa55
R3(config-if)# ip ospf message-digest-key 1 md5 WrongKey
```

Wait 40 seconds, then run `show ip ospf neighbor` on R2 and ping PC-C from PC-A. What changes, and why? Restore the correct key and check that everything recovers.

## Viva questions

1. Why does a routing protocol need authentication? *A rogue router could inject fake routes and redirect or black-hole traffic.*
2. What is the difference between plain-text and MD5 authentication in OSPF? *Plain text sends the password inside every packet; MD5 sends only a hash made with the key, so the key is never visible on the link.*
3. Which two commands enable OSPF MD5 authentication? *`ip ospf message-digest-key <id> md5 <key>` on the interface and `area <n> authentication message-digest` under `router ospf`.*
4. What happens if the keys on two neighbours do not match? *Packets fail the hash check, the neighbourship drops after the dead interval, and routes through that link are lost.*
5. Which command shows that authentication is active on an interface? *`show ip ospf interface <interface>`: "Message digest authentication enabled".*
6. Why is `passive-interface` used on LAN interfaces? *To advertise the LAN network without sending OSPF packets to hosts, so no one on the LAN can form an OSPF neighbourship.*
7. Is MD5 still a strong choice today? *No; it has known weaknesses. HMAC-SHA authentication is preferred where supported.*

## Sources

- Cisco, *IP Routing: OSPF Configuration Guide*, section on OSPF MD5 (message digest) authentication (verify exact title for your IOS version).
- Cisco Networking Academy, CCNA Security lab on securing routing protocols with OSPF authentication (verify).
- RFC 2328, *OSPF Version 2*, Appendix D: Authentication.
