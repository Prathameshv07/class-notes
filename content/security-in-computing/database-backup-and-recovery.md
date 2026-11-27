---
id: database-backup-and-recovery
title: Database Backup and Recovery
subject: security-in-computing
unit: 2
type: theory
syllabus: [UITVI-001]
status: draft
sources:
  - "Mark Rhodes-Ousley, Information Security: The Complete Reference, 2nd ed., chapter on database security, backup and recovery (verify)"
  - "NIST SP 800-34 Rev. 1, Contingency Planning Guide for Federal Information Systems"
  - "Abraham Silberschatz, Henry F. Korth and S. Sudarshan, Database System Concepts, chapter on recovery systems (verify)"
---

## Why should you care?

On result day, a database admin runs a cleanup script on the wrong database and deletes the marks table. Or ransomware encrypts the server at 2 a.m. Or the disk simply fails.

In each case, one question decides whether this is a bad hour or a disaster: **can we get the data back, how recent is it, and how fast?**

The AIIMS ransomware attack in 2022 was recovered because a copy of the database survived on servers the malware did not reach. Many organisations discover during an attack that their backups were missing, incomplete, or encrypted along with everything else. Backup and recovery is the availability and integrity insurance of every database.

## In one line

Database backup and recovery means regularly copying database data and logs to safe locations, and having a tested procedure to restore the database to a correct, recent state after failure, error or attack.

## Explained

### Q1. Why are database backups needed?

==A backup is a copy of data kept separately so the original can be restored if it is lost or damaged.==

Databases need backups against:

- ==**Hardware failure:**== disk, server or storage crash.
- ==**Human error:**== wrong DELETE, DROP or update.
- ==**Software bugs**== that corrupt data.
- ==**Attacks:**== ransomware, sabotage by insiders.
- ==**Disasters:**== fire, flood, power failure at the site.

Backups protect **availability** and **integrity** in the [[cia-triad]].

### Q2. What are the types of database backup? ⭐

| Type | What it copies | Restore needs | Trade-off |
|---|---|---|---|
| ==Full== | The entire database | Only this backup | Slow and large, simplest restore |
| ==Differential== | Changes since the last **full** backup | Last full + last differential | Grows daily; faster restore |
| ==Incremental== | Changes since the last backup of **any** type | Last full + every incremental since | Smallest and fastest to take; slowest restore |
| ==Transaction log backup== | The log of every committed change | Full + logs, replayed in order | Allows point-in-time recovery |

### Q3. What are hot and cold backups?

- ==**Cold (offline) backup:**== the database is shut down, then its files are copied. Simple and consistent, but the system is unavailable meanwhile.
- ==**Hot (online) backup:**== taken while the database keeps running, using the DBMS's backup tools and logs to keep the copy consistent. Needed for systems that must run 24×7, such as banking or IRCTC.

A plain file copy of a running database is often **corrupt**; always use the DBMS's own backup method.

### Q4. What are RPO and RTO? ⭐

==**Recovery Point Objective (RPO):** the maximum amount of data, measured in time, that the organisation can afford to lose.==

==**Recovery Time Objective (RTO):** the maximum time allowed to restore the system after a failure.==

| System | Typical RPO | Typical RTO |
|---|---|---|
| Bank core system | Seconds (almost zero loss) | Minutes |
| College ERP | A few hours | A day |
| Archive of old records | A week | Several days |

RPO decides **how often** to back up; RTO decides **how fast** recovery must be, and so the method and budget.

### Q5. How is a database recovered? ⭐

1. ==**Find the cause and stop the damage**== (isolate infected servers, stop the faulty job).
2. **Restore the last good full backup.**
3. ==**Apply differential or incremental backups**== in order.
4. ==**Replay transaction logs**== up to just before the failure (**point-in-time recovery**), for example to 10:42 a.m., one minute before the wrong DELETE.
5. ==**Verify the data**== (row counts, sample checks) before users return.
6. ==**Record what happened**== and improve the process.

### Q6. What are the best practices for secure backups?

- ==**3-2-1 rule:**== 3 copies, on 2 types of media, with 1 off-site.
- ==**Keep one copy offline or immutable**== so ransomware cannot encrypt or delete it.
- ==**Encrypt backups**== and protect the keys separately; a stolen backup is a full data leak.
- ==**Restrict backup access:**== separate accounts; backup admins cannot also delete backups alone.
- ==**Test restores regularly:**== a backup is only proven when it has been restored successfully.
- ==**Monitor backup jobs**== and alert on failures.

### Q7. Differentiate between differential and incremental backups. ⭐ [2M]

| Point | Differential | Incremental |
|---|---|---|
| ==Copies changes since== | The last **full** backup | The last backup **of any type** |
| ==Size over the week== | Grows each day | Small each day |
| ==Restore needs== | Last full + last differential | Last full + every incremental since |
| ==Restore speed== | Faster | Slower |

### Q8. What is point-in-time recovery? [2M]

==Point-in-time recovery restores a database to its exact state at a chosen moment, by restoring a backup and replaying transaction logs up to that time.==

- **Example:** a wrong DELETE ran at 10:43, so the database is recovered to 10:42.
- ==**Needs transaction log backups**==, taken regularly.

> [!beyond] Beyond the syllabus
> Critical systems also use **replication** to a standby database at another site (for example a disaster-recovery data centre in another city). Replication protects against site failure, but it also copies mistakes and ransomware damage instantly, so it complements backups and never replaces them.

## Analogy and real-life example

**Analogy: important certificates.** You keep your original mark sheets in a file at home, a scanned copy in your email, and a photocopy at a relative's house. If the house floods, you still have two copies. If you never checked that the scan is readable, you may discover too late that it is blurred. That is the 3-2-1 rule and restore testing.

**Real example: banks and RBI.** Banks run their core banking databases with near-zero RPO, replicating to a disaster-recovery site and keeping backups. RBI guidelines expect banks to have tested business continuity and disaster recovery arrangements, because even a few hours of lost transactions would be unacceptable.

## Diagram

```mermaid
flowchart LR
  accTitle: Weekly backup schedule and point-in-time recovery
  accDescr: A full backup is taken on Sunday, incremental backups each following night, and transaction logs every 15 minutes. To recover from an error on Wednesday morning, restore Sunday's full backup, apply the Monday and Tuesday incrementals, then replay logs up to just before the error.
  F[Sunday: full backup] --> I1[Monday: incremental]
  I1 --> I2[Tuesday: incremental]
  I2 --> L[Wednesday: log backups every 15 min]
  L --> E[10:43 wrong DELETE]
  E -.->|recover to 10:42| R[Restore full + incrementals + logs]
```

## Key points

> [!important] Key points
> - Backups protect against hardware failure, human error, software bugs, attacks and disasters (availability and integrity).
> - Types: full, differential (since last full), incremental (since last backup), transaction log (point-in-time recovery).
> - Cold backup: database offline. Hot backup: database running, using DBMS tools.
> - RPO: maximum acceptable data loss (decides backup frequency). RTO: maximum acceptable downtime (decides recovery speed).
> - Recovery: stop damage → restore full → apply differential/incremental → replay logs → verify → record.
> - Best practices: 3-2-1 rule, offline/immutable copy, encryption, restricted access, regular restore tests, monitoring.

## Common mistakes

> [!warning] Common mistake
> Mixing up differential and incremental. Differential = changes since the last **full** backup; incremental = changes since the last backup **of any type**.

> [!warning] Common mistake
> Treating replication as a backup. Replication copies a wrong DELETE or ransomware encryption to the standby within seconds.

## Sources

- Mark Rhodes-Ousley, *Information Security: The Complete Reference*, 2nd ed., McGraw-Hill, chapter on database security (verify).
- NIST Special Publication 800-34 Revision 1, *Contingency Planning Guide for Federal Information Systems*.
- Abraham Silberschatz, Henry F. Korth and S. Sudarshan, *Database System Concepts*, McGraw-Hill, chapter on recovery systems (verify).
