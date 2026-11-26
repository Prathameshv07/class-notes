---
id: ntp-syslog-practical
title: "Practical 2: NTP and Syslog on Cisco Routers"
subject: security-in-computing
unit: 1
type: practical
practical: 2
syllabus: [UITVI-002]
status: draft
sources:
  - "Cisco, Basic System Management Configuration Guide: NTP (verify exact title for your IOS version)"
  - "Cisco Networking Academy, CCNA Security Packet Tracer activity: Configure Cisco Routers for Syslog, NTP, and SSH Operations (verify)"
  - "RFC 5905, Network Time Protocol Version 4; RFC 5424, The Syslog Protocol (severity levels)"
---

## Aim

To configure routers to take their time from an authenticated NTP server and to send their log messages, with timestamps, to a central syslog server.

## Concepts used

- [[ospf-md5-authentication-practical]]: this practical starts from that topology.
- [[information-security-overview]]: availability and integrity of records.
- Logging and monitoring are detective controls ([[security-methodology]], Q7).

**Why this matters.** When something goes wrong at 2 a.m., the first questions are "what happened, on which device, and when?" Logs answer the first two; correct time answers the third. If three routers have three different clocks, you cannot put events in order. If logs stay only on the router, an attacker who gets in can erase them. A central syslog server with synchronised time fixes both.

## Before the lab

**Install or open**
- Cisco Packet Tracer 8.x or later.
- Your saved file **practical1.pkt**, with OSPF and MD5 authentication working. If you do not have it, finish Practical 1 first.

**Revise first**
- The CLI modes table in [[ospf-md5-authentication-practical]] (Before the lab).
- Why logs and correct time matter: detective controls in [[security-methodology]].
- Ports: NTP uses **UDP 123**, syslog uses **UDP 514**.

## Setup

**Tool:** Cisco Packet Tracer (8.x or later).

### Devices and topology

The Practical 1 topology plus one **Server-PT** on switch S1. The one server runs both NTP and syslog.

```mermaid
flowchart LR
  accTitle: Topology for NTP and syslog
  accDescr: The practical 1 topology with one extra server, NTP and syslog server at 192.168.1.5, on switch S1 beside PC-A. R1, R2 and R3 all send logs to it and take time from it.
  SRV[NTP + Syslog server 192.168.1.5] --- S1[S1]
  PCA[PC-A 192.168.1.3] --- S1
  S1 --- R1[R1]
  R1 --- R2[R2] --- R3[R3]
  R3 --- S3[S3] --- PCC[PC-C 192.168.3.3]
```

### Server settings

| Setting | Value |
|---|---|
| IP address / mask / gateway | 192.168.1.5 / 255.255.255.0 / 192.168.1.1 |
| Services → NTP | On; Authentication **Enable**; Key **1**; Password **NTPpa55** |
| Services → SYSLOG | On |

All other addresses are the same as in Practical 1.

## Steps

### Part A. Add and prepare the server

1. **Open your Practical 1 file and save a copy.** File → Open → `practical1.pkt`, then File → Save As → `practical2.pkt`.
   → **You should see:** the Practical 1 topology, all links green.
2. **Add the server.** End Devices → **Server**. Connect its FastEthernet0 to S1 Fa0/2 with a **Copper Straight-Through** cable. Name it `NTP-Syslog`.
   → **You should see:** the link turn amber, then green.
3. **Address the server.** Server → Desktop → IP Configuration → Static: 192.168.1.5, 255.255.255.0, gateway 192.168.1.1.
   → **You should see:** the values saved.
4. **Turn on NTP with authentication.** Server → Services → **NTP**: Service **On**; Authentication **Enable**; Key `1`; Password `NTPpa55`.
   → **You should see:** the server's current date and time shown on the NTP page.
5. **Turn on syslog.** Server → Services → **SYSLOG**: Service **On**.
   → **You should see:** an empty message table.
6. **Check that every router can reach the server.** On R3: `ping 192.168.1.5`.
   → **You should see:** `Success rate is 100 percent (5/5)`. The first try may show `.!!!!` (80 percent); that is normal.

### Part B. Synchronise router clocks with NTP

7. **Look at the router's clock before NTP.** On R1: `show clock`.
   → **You should see:** a time with `*` in front and a date that is not today. The `*` means the clock is not trusted.
8. **Configure NTP with authentication on R1.**
   ```text
   configure terminal
   ntp authenticate
   ntp authentication-key 1 md5 NTPpa55
   ntp trusted-key 1
   ntp server 192.168.1.5 key 1
   ntp update-calendar
   end
   ```
   → **You should see:** no error message.
9. **Wait for synchronisation, then check.** Click **Fast Forward Time** two or three times, then on R1: `show ntp status`.
   → **You should see:** `Clock is synchronized, stratum 2, reference is 192.168.1.5`. Now `show clock` shows the server's date and time, without the `*`.
10. **Repeat Step 8 on R2 and R3,** then check `show ntp status` on each.
    → **You should see:** `Clock is synchronized` on both.

### Part C. Send logs to the syslog server

11. **Turn on timestamps and remote logging on R1.**
    ```text
    configure terminal
    service timestamps log datetime msec
    logging host 192.168.1.5
    end
    ```
    → **You should see:** the prompt return to `R1#` with no error.
12. **Repeat Step 11 on R2 and R3.**
    → **You should see:** no error on either router.
13. **Generate a log message on each router.** Shut and re-enable an unused interface (R1 and R3: g0/2; R2: g0/0):
    ```text
    configure terminal
    interface g0/2
     shutdown
     no shutdown
    end
    ```
    → **You should see:** on the router, lines such as `%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down`, now starting with today's date and time.
14. **Read the logs on the server.** Server → Services → SYSLOG.
    → **You should see:** entries from 192.168.1.1 (R1), 10.1.1.2 (R2) and 10.2.2.1 (R3), each with a time that matches the server's clock.
15. **Check the logging settings.** On R1: `show logging`.
    → **You should see:** `Logging to 192.168.1.5` in the output.
16. **Save everything.** `copy running-config startup-config` on each router; File → Save.
    → **You should see:** `Building configuration... [OK]`.

## Code

Complete configuration added in this practical. **The same commands go on R1, R2 and R3.**

```text
ntp authenticate
ntp authentication-key 1 md5 NTPpa55
ntp trusted-key 1
ntp server 192.168.1.5 key 1
ntp update-calendar
service timestamps log datetime msec
logging host 192.168.1.5
```

## Output

> [!warning] Replace with your own output
> The blocks below show what correct output looks like. After running the lab, paste your real output here, or save a screenshot in `content/assets/img/security-in-computing/` and link it with alt text.

**Router clock now synchronised** (`show ntp status` on R1, first line):

```output
Clock is synchronized, stratum 2, reference is 192.168.1.5
```

**Logging to the server is on** (`show logging` on R1, relevant lines):

```output
Syslog logging: enabled (0 messages dropped, 0 messages rate-limited, 0 flushes, 0 overruns, xml disabled, filtering disabled)
    ...
    Trap logging: level informational, 0 message lines logged
        Logging to 192.168.1.5  (udp port 514, audit disabled, link up),
```

**Messages received at the server** (Services → SYSLOG, example entries):

```output
Time                      HostName       Message
<date> 10:15:22.415       192.168.1.1    %SYS-5-CONFIG_I: Configured from console by console
<date> 10:16:03.120       10.2.2.1       %LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
```

## Explanation

### NTP commands

- `ntp authenticate`: accept time only from an authenticated source.
- `ntp authentication-key 1 md5 NTPpa55`: define key 1. ==The key number and password must match the NTP server.==
- `ntp trusted-key 1`: mark key 1 as trusted.
- `ntp server 192.168.1.5 key 1`: take time from this server, using key 1.
- `ntp update-calendar`: also write the correct time into the router's hardware clock, so it survives a reload.

Without authentication, an attacker could run a fake NTP server and shift a router's clock, making logs useless as evidence and breaking time-based checks such as certificate validity.

### Syslog commands

- `service timestamps log datetime msec`: stamp every log message with date and time to the millisecond.
- `logging host 192.168.1.5`: send a copy of every log message to the syslog server (UDP port 514).

==Central logging keeps a copy of events outside the device, so an attacker who breaks into a router cannot simply erase the evidence.==

### Syslog severity levels

Every message carries a level; in `%SYS-5-CONFIG_I` the **5** is the level.

| Level | Name | Level | Name |
|---|---|---|---|
| 0 | Emergency | 4 | Warning |
| 1 | Alert | 5 | Notification |
| 2 | Critical | 6 | Informational |
| 3 | Error | 7 | Debugging |

Lower number means more serious. By default the router sends levels 0–6 (informational and above) to the server.

> [!beyond] Beyond the syllabus
> In real networks, logs from hundreds of devices go into a SIEM (Security Information and Event Management) system, which correlates them and raises alerts. You meet SIEM in Unit 3 theory.

## Common errors and fixes

| Message or symptom | Cause | Fix |
|---|---|---|
| `show ntp status` keeps saying `Clock is unsynchronized, stratum 16, no reference clock` | NTP is off on the server, the key or password does not match, or the router cannot reach the server | Check Services → NTP is **On** with Key 1 and Password `NTPpa55` (case-sensitive). Ping 192.168.1.5 from the router. Then click Fast Forward Time and check again |
| `show clock` still starts with `*` | The router has not yet synchronised | Same checks as above; NTP takes a minute or two |
| Ping from a router to 192.168.1.5 fails | Wrong server address or gateway, or OSPF is broken | Check the server's IP Configuration (gateway 192.168.1.1) and `show ip route` on the router |
| The syslog table on the server stays empty | SYSLOG service is off, a typo in `logging host`, or no new events since configuring it | Turn SYSLOG **On**; check `show logging` for `Logging to 192.168.1.5`; repeat Step 13 to create new messages |
| `% Invalid input detected at '^' marker.` on `logging host 192.168.1.5` | Some Packet Tracer versions accept only the short form | Use `logging 192.168.1.5` instead |
| Log lines show a time such as `*00:15:22` with no date | `service timestamps log datetime msec` missing, or the clock is not synchronised | Add the timestamps command (Step 11) and fix NTP first |

## Try it

1. On R3, change the NTP key password to `WrongKey` (`ntp authentication-key 1 md5 WrongKey`), set a wrong clock with `clock set 00:00:00 1 Jan 2020` in privileged mode, and wait. Does `show ntp status` ever reach `synchronized`? Why not? Restore the correct key.
2. On R1, try `logging trap warnings`. Generate a `CONFIG_I` message again. Does it still reach the server? Explain using the severity table.

## Viva questions

1. Why must all network devices have the same time? *To put events from different devices in the correct order during troubleshooting and incident investigation; logs are also legal evidence.*
2. What is NTP and which port does it use? *Network Time Protocol, which synchronises clocks over a network; UDP port 123.*
3. What is a stratum in NTP? *The distance from a reference clock; stratum 1 is directly connected to one, and each hop adds 1.*
4. Why authenticate NTP? *So a router accepts time only from a trusted server, not from an attacker's fake server.*
5. What is syslog and which port does it use? *A standard for sending log messages to a central server; UDP port 514.*
6. In `%LINK-5-CHANGED`, what does 5 mean? *Severity level 5, Notification.*
7. Which is more serious, level 2 or level 6? *Level 2 (Critical); lower numbers are more serious.*
8. Why send logs to a server instead of keeping them only on the router? *The router's buffer is small and lost on reload, and an intruder could delete it; a central copy is safe and searchable.*

## Sources

- Cisco, *Basic System Management Configuration Guide*, chapters on NTP and system message logging (verify exact titles for your IOS version).
- Cisco Networking Academy, CCNA Security Packet Tracer activity on syslog, NTP and SSH (verify).
- RFC 5905, *Network Time Protocol Version 4*; RFC 5424, *The Syslog Protocol*.
