---
id: security-in-computing
title: Security in Computing
type: index
class: Sem 6 BSc IT
codes: "UITVI-001 (theory, 2 credits), UITVI-002 (practical, 2 credits)"
---

Topic list in lecture order, mapped from the university syllabus (BOS 05/06/2026).
Faded links are not written yet.

**Reference books (from syllabus):** Pfleeger, *Security in Computing*; Stallings, *Cryptography and Network Security*; Godbole, *Information Systems Security*.
**Additional:** NIST guidelines, OWASP, CIS Benchmarks.

## Theory

### Module 1 · Information Security Fundamentals and Data Protection

**Unit 1 · Information Security Fundamentals (8 h)**
1. [[information-security-overview]]: what it is, why it matters, how practice evolved
2. [[security-methodology]]: building a security program, business processes vs technical controls
3. [[justifying-security-investment]]
4. [[threats-and-attacks]]
5. [[risk-analysis]]
6. [[cia-triad]]
7. [[defense-models]]: lollipop vs onion
8. [[zones-of-trust]]
9. [[network-defense-best-practices]]

**Unit 2 · Authentication, Encryption and Database Security (7 h)**
1. [[authentication]]
2. [[authorization]]
3. [[symmetric-encryption]]
4. [[public-key-encryption]]
5. [[public-key-infrastructure]]
6. [[storage-security]]: evolution, risk remediation, modern protection
7. [[database-security-layers]]
8. [[database-application-security]]
9. [[database-backup-and-recovery]]
10. [[database-auditing-and-monitoring]]: incl. server maintenance

### Module 2 · Network, System and Application Security

**Unit 3 · Network and System Security (8 h)**
1. [[secure-network-design]]: performance, availability, security
2. [[network-device-security]]: switch and router hardening
3. [[firewalls]]: concepts and capabilities
4. [[firewall-design-and-deployment]]
5. [[wireless-network-security]]
6. [[ids-ips]]
7. [[siem]]
8. [[voip-and-pbx-security]]
9. [[os-security-models]]

**Unit 4 · Cloud, Application and Physical Security (7 h)**
1. [[virtual-machine-and-cloud-security]]
2. [[secure-application-design]]: incl. secure development lifecycle
3. [[web-application-security]]
4. [[client-application-security]]
5. [[remote-administration-security]]
6. [[physical-security-basics]]: asset classification, vulnerability assessment, site selection
7. [[physical-access-control]]: access control systems, locks, intrusion detection

## Practicals (Packet Tracer / GNS3)

**Unit 1 · Router Security and Authentication**
1. [[prac-ospf-md5-authentication]]
2. [[prac-ntp-syslog]]
3. [[prac-ssh-remote-access]]
4. [[prac-aaa-local-authentication]]: verify via console and remote client

**Unit 2 · ACLs, Firewall and IPS Configuration**
5. [[prac-extended-numbered-acl]]
6. [[prac-ipv4-ipv6-acl]]: remote access control, attack mitigation
7. [[prac-zone-based-firewall]]
8. [[prac-ios-ips]]: incl. modifying signatures

**Unit 3 · Layer 2 Security and VPN Configuration**
9. [[prac-layer2-security]]: root bridge, STP protection, port security, CAM overflow
10. [[prac-vlan-security]]
11. [[prac-site-to-site-ipsec-vpn]]

**Unit 4 · ASA Firewall and Secure Network Services**
12. [[prac-asa-basic-settings]]: security levels, routing, NAT, inspection
13. [[prac-asa-dhcp-aaa-ssh]]
14. [[prac-asa-dmz-static-nat-acl]]
