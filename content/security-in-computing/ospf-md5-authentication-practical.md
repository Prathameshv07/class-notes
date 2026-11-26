---
id: ospf-md5-authentication-practical
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

## Before the lab

**Install or open**
- **Cisco Packet Tracer 8.x or later.** It is free: create a Cisco Networking Academy (Skills for All) account and download it from there.
- Nothing else. This practical builds the topology that Practicals 2 to 8 reuse, so **save your file carefully** at the end.

**Revise first**
- IP addressing and subnet masks; what a /30 network is (2 usable addresses, used for router-to-router links).
- OSPF basics: what a neighbour is, what area 0 is, what a wildcard mask is (0.0.0.255 matches a /24).
- [[authentication]]: the idea of a shared secret.

**Cisco CLI in one minute**

| Prompt | Mode | How you get there |
|---|---|---|
| `R1>` | User EXEC (look only) | Starting point |
| `R1#` | Privileged EXEC (show, save) | `enable` |
| `R1(config)#` | Global configuration | `configure terminal` |
| `R1(config-if)#` | Interface configuration | `interface g0/1` |
| `R1(config-router)#` | Routing protocol configuration | `router ospf 1` |

`exit` goes back one level; `end` goes straight back to `R1#`. Press **Tab** to complete a command and **?** to list options.

## Setup

**Tool:** Cisco Packet Tracer (8.x or later). GNS3 with a Cisco IOS image also works with the same commands.

### Devices and topology

Three 2911 routers (R1, R2, R3), two 2960 switches (S1, S3), two PCs (PC-A, PC-C).

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

## Steps

### Part A. Build the topology

1. **Add three routers.** In the device box at the bottom, choose Network Devices → Routers → **2911**, and drag three onto the workspace. Click each one → Config tab → Display Name: R1, R2, R3.
   → **You should see:** three router icons labelled R1, R2, R3.
2. **Add two switches and two PCs.** Network Devices → Switches → **2960** (two: S1, S3); End Devices → **PC** (two: PC-A, PC-C).
   → **You should see:** all seven devices on the workspace.
3. **Connect the routers.** Connections (lightning icon) → **Copper Cross-Over**. Click R1 → GigabitEthernet0/1, then R2 → GigabitEthernet0/1. Repeat for R2 G0/2 to R3 G0/1.
   → **You should see:** a line with a red dot at each end. The dots stay red until the interfaces are switched on in Part B.
4. **Connect the LANs.** **Copper Straight-Through**: PC-A FastEthernet0 → S1 Fa0/1, and S1 Gig0/1 → R1 G0/0. Do the same for PC-C → S3 → R3 G0/0.
   → **You should see:** the PC–switch dots turn amber, then green within about 30 seconds. The router-side dot stays red for now.
5. **Address the PCs.** Click PC-A → Desktop → IP Configuration → Static. Enter IP 192.168.1.3, mask 255.255.255.0, gateway 192.168.1.1. Do the same for PC-C with its values from the table.
   → **You should see:** the values saved; no error message.

### Part B. Basic settings and OSPF

6. **Open R1's command line.** Click R1 → **CLI** tab. If asked `Would you like to enter the initial configuration dialog? [yes/no]:`, type `no` and press Enter.
   → **You should see:** the prompt `Router>`.
7. **Name the router and stop name lookups.**
   ```text
   enable
   configure terminal
   hostname R1
   no ip domain-lookup
   ```
   → **You should see:** the prompt change to `R1(config)#`.
8. **Address and switch on R1's interfaces.**
   ```text
   interface g0/0
    ip address 192.168.1.1 255.255.255.0
    no shutdown
   interface g0/1
    ip address 10.1.1.1 255.255.255.252
    no shutdown
   exit
   ```
   → **You should see:** `%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up`, and the same for G0/1. The router-side dots on the workspace turn green.
9. **Start OSPF on R1.**
   ```text
   router ospf 1
    router-id 1.1.1.1
    network 192.168.1.0 0.0.0.255 area 0
    network 10.1.1.0 0.0.0.3 area 0
    passive-interface g0/0
   end
   ```
   → **You should see:** no message yet. R1 has no OSPF neighbour until R2 is configured.
10. **Configure R2 the same way.** Use hostname R2, its two interface addresses, `router-id 2.2.2.2`, and `network 10.1.1.0 0.0.0.3 area 0` and `network 10.2.2.0 0.0.0.3 area 0`. R2 has no LAN, so it needs no `passive-interface`.
    → **You should see:** after a few seconds, `%OSPF-5-ADJCHG: Process 1, Nbr 1.1.1.1 on GigabitEthernet0/1 from LOADING to FULL, Loading Done`.
11. **Configure R3 the same way.** Use hostname R3, its addresses, `router-id 3.3.3.3`, `network 192.168.3.0 0.0.0.255 area 0`, `network 10.2.2.0 0.0.0.3 area 0` and `passive-interface g0/0`.
    → **You should see:** `%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/1 from LOADING to FULL, Loading Done`.
12. **Test before adding security.** PC-A → Desktop → Command Prompt → `ping 192.168.3.3`.
    → **You should see:** replies. The first reply may be `Request timed out.` while addresses are being learned; that is normal.

### Part C. Add MD5 authentication

13. **Add the key and enable authentication on R1.**
    ```text
    configure terminal
    interface g0/1
     ip ospf message-digest-key 1 md5 MD5pa55
    exit
    router ospf 1
     area 0 authentication message-digest
    end
    ```
    → **You should see:** within about 40 seconds, `%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/1 from FULL to DOWN, Neighbor Down: Dead timer expired`. **This is expected:** R2 does not know the key yet, so R1 now ignores it. (Use the **Fast Forward Time** button at the bottom of Packet Tracer to skip waiting.)
14. **Add the key on both of R2's OSPF interfaces, then enable authentication.**
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
    → **You should see:** the neighbour to R1 come back (`from LOADING to FULL`), and the neighbour to R3 go down, because R3 has no key yet.
15. **Add the key and authentication on R3.** Use the same commands as R1, on R3's G0/1.
    → **You should see:** `%OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on GigabitEthernet0/1 from LOADING to FULL, Loading Done`.
16. **Check authentication on an interface.** On R1: `show ip ospf interface g0/1`.
    → **You should see:** the lines `Message digest authentication enabled` and `Youngest key id is 1`.
17. **Check neighbours.** On R2: `show ip ospf neighbor`.
    → **You should see:** two neighbours, 1.1.1.1 and 3.3.3.3, both in state `FULL`.
18. **Test end to end.** From PC-A: `ping 192.168.3.3`.
    → **You should see:** four replies.
19. **Save everything.** On each router: `copy running-config startup-config`, press Enter at `Destination filename [startup-config]?`. Then File → Save As → `practical1.pkt`.
    → **You should see:** `Building configuration... [OK]` on each router.

## Code

Complete configuration for your journal. R2 and R3 follow the same pattern with their own addresses and router IDs (Steps 10, 11, 14 and 15).

**R1**

```text
hostname R1
no ip domain-lookup
interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface g0/1
 ip address 10.1.1.1 255.255.255.252
 ip ospf message-digest-key 1 md5 MD5pa55
 no shutdown
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 passive-interface g0/0
 area 0 authentication message-digest
```

**R2**

```text
hostname R2
no ip domain-lookup
interface g0/1
 ip address 10.1.1.2 255.255.255.252
 ip ospf message-digest-key 1 md5 MD5pa55
 no shutdown
interface g0/2
 ip address 10.2.2.2 255.255.255.252
 ip ospf message-digest-key 1 md5 MD5pa55
 no shutdown
router ospf 1
 router-id 2.2.2.2
 network 10.1.1.0 0.0.0.3 area 0
 network 10.2.2.0 0.0.0.3 area 0
 area 0 authentication message-digest
```

**R3**

```text
hostname R3
no ip domain-lookup
interface g0/0
 ip address 192.168.3.1 255.255.255.0
 no shutdown
interface g0/1
 ip address 10.2.2.1 255.255.255.252
 ip ospf message-digest-key 1 md5 MD5pa55
 no shutdown
router ospf 1
 router-id 3.3.3.3
 network 192.168.3.0 0.0.0.255 area 0
 network 10.2.2.0 0.0.0.3 area 0
 passive-interface g0/0
 area 0 authentication message-digest
```

## Output

> [!warning] Replace with your own output
> The blocks below show what correct output looks like. After running the lab, paste your real output here, or save a screenshot in `content/assets/img/security-in-computing/` and link it with alt text.

**Authentication active on the interface** (`show ip ospf interface g0/1` on R1, relevant lines):

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

### What Steps 7–11 do

- `hostname`, `no ip domain-lookup`: name the router; stop it from treating a typo as a hostname to look up.
- `ip address … / no shutdown`: address each interface and turn it on.
- `router ospf 1`: start OSPF process 1. The number is local to the router.
- `router-id`: give each router a fixed, easy-to-read identity.
- `network … area 0`: run OSPF on interfaces in that range. The second value is a **wildcard mask** (0.0.0.3 matches a /30).
- `passive-interface g0/0`: advertise the LAN, but send no OSPF packets towards PCs. A PC on that LAN cannot then form a fake neighbourship.

### What Steps 13–15 do

- `ip ospf message-digest-key 1 md5 MD5pa55`: on this interface, key number 1 with the secret `MD5pa55`. ==The key ID and the key must match on both ends of a link.==
- `area 0 authentication message-digest`: every interface in area 0 must use MD5 authentication.

Before sending, the router computes an MD5 hash over the OSPF packet plus the secret key and attaches it. The receiver recomputes the hash with its own key. If the two hashes differ, the packet is dropped. ==The key itself is never sent over the link.==

### Why the neighbour went down in Step 13

As soon as R1 had authentication and R2 did not, R1 dropped R2's unauthenticated hellos. After the dead timer (40 seconds) the neighbourship went down and routes disappeared. When R2 was configured too, it returned to FULL. This is the security feature working: a router without the right key is ignored.

> [!beyond] Beyond the syllabus
> MD5 is now considered weak. Newer IOS versions support OSPFv2 authentication with HMAC-SHA using a key chain (`key chain`, `cryptographic-algorithm hmac-sha-256`). Packet Tracer may not support it; real networks should prefer it.

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| `% Invalid input detected at '^' marker.` | A typing mistake, or the command was typed in the wrong mode | Check the `^` position for the typo. Check the prompt: interface commands need `(config-if)#`, OSPF commands need `(config-router)#` |
| `% Incomplete command.` | Part of the command is missing | Type `?` after the command to see what is expected |
| Router freezes on `Translating "netwrok"...domain server (255.255.255.255)` | A mistyped word in `R1>` or `R1#` mode was treated as a hostname to look up | Wait about 30 seconds (or press Ctrl+Shift+6). Prevent it with `no ip domain-lookup` (Step 7) |
| The router-side link dot stays red | Interface not switched on, or wrong cable | Check `show ip interface brief`: if it says `administratively down`, enter the interface and type `no shutdown`. If the cable is wrong, delete it and use the **Automatically Choose Connection Type** cable |
| `show ip ospf neighbor` shows nothing (before Part C) | A wrong `network` statement or wildcard mask, or an interface is down | Check `show ip protocols` and `show ip interface brief`; compare each `network` line with the addressing table |
| After Part C, a neighbour never returns to FULL | The key or key ID differs on the two ends (keys are case-sensitive: `MD5pa55` is not `md5pa55`), or `area 0 authentication message-digest` is missing on one router | Re-enter the key on both interfaces of that link, exactly as given, and check the area command on both routers |
| PC ping shows `Request timed out.` only for the first reply | Address learning (ARP) on the first packet | Normal; ping again |
| PC ping shows `Reply from 192.168.1.1: Destination host unreachable.` | The router has no route to the far network (OSPF not complete) | Check `show ip route ospf` on R1; fix OSPF neighbours first |

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
