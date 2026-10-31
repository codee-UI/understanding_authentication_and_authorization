# DMZ Network Guide (text version of the simulator)

## 1. Internet, intranet, extranet and DMZ

| Zone | Who can reach it | Trust |
|---|---|---|
| Internet | Anyone. Global TCP/IP network of networks; the web is one service among email, file transfer, etc. | None |
| Intranet | Only members, employees or authorized people; firewalled from outside. | High |
| Extranet | Intranet segment extended to partners, vendors and suppliers, isolated from other internet users. | Controlled |
| DMZ | Buffer segment between internet and intranet hosting public-facing services. Cannot start sessions into the private network. | Medium |

## 2. Topology

```
                         [NIDS 2]
                             .
 [Internet]--[FW+NIPS 1]--+--DMZ switch--[FW+NIPS 2]--+--LAN switch--[Database]
 Public IPs               |   |   |                  |     |
                       Web  Mail  Bastion            |   Workstations (private IPs)
                                                     |
                                       [Interior LAN: Data Center]--[NIDS 1]
```

Layouts: single firewall with three interfaces, or the more secure dual (back-to-back) firewall shown above.

## 3. Working principle, step by step

1. **Request arrives** from public IPs at the outer firewall and NIPS 1.
2. **Outer filtering**: ACL checks IP, port and protocol; NIPS inspects payloads and drops attacks.
3. **Delivery to a DMZ host**: only allowed services (web 80/443, mail 25, bastion/VPN).
4. **Service handles it**; if data is needed, it sends a narrow, logged query inward.
5. **Inner filtering**: inner firewall and NIPS 2 allow only that one flow (e.g. web server to database).
6. **Internal users go out** through a proxy or bastion host in the DMZ (NAT hides private IPs).
7. **Replies return, nothing starts inward**: the DMZ forwards only packets already requested.
8. **Monitor and log**: NIDS sensors alert administrators or a SIEM.

Rule set: Internet→DMZ only needed ports. Internet→intranet denied. Intranet→DMZ allowed. DMZ→intranet denied except narrow, logged exceptions. DMZ→internet replies and needed updates only.

## 4. Simulation scenarios (as shown in the web page)

| Scenario | Path | Result |
|---|---|---|
| 1. Visitor opens website | User → FW1 → DMZ → Web server, and back | Allowed; intranet untouched |
| 2. Web server fetches data | User → Web → FW2 → LAN → Database, and back | Allowed on one logged port |
| 3. Employee browses internet | Workstation → FW2 → Bastion/proxy → FW1 → Internet; reply returns | Allowed; DMZ host opens the outbound session, private IP hidden |
| 4. Attack on intranet | Attacker → FW1 | Blocked at outer firewall, logged by NIPS 1 |
| 5. Web server compromised | Attacker → Web → FW2 | Blocked: DMZ cannot initiate into private network; NIPS 2/NIDS 2 alert |
| 6. Partner (extranet) | Partner → FW1 → Bastion → FW2 → Database; then → Data Center | Only the specific service allowed; data center denied, NIDS 1 logs |

## 5. Glossary

| Term | Full form | Meaning |
|---|---|---|
| DMZ | Demilitarized Zone | Buffer network hosting public-facing services |
| LAN | Local Area Network | Private network inside an organization |
| TCP/IP | Transmission Control Protocol / Internet Protocol | Protocol suite of the internet |
| NIPS | Network Intrusion Prevention System | Inline; inspects and blocks attacks |
| NIDS | Network Intrusion Detection System | Passive; detects and alerts |
| FW / ACL | Firewall / Access Control List | Rule-based traffic filtering |
| Bastion host | Hardened edge server | Minimal, secured host exposed to untrusted networks |
| Perimeter network | Screened subnet | Another name for the DMZ |
| NAT | Network Address Translation | Maps private IPs to public IPs |
| VPN | Virtual Private Network | Encrypted tunnel for remote/partner access |
| WAF | Web Application Firewall | Filters HTTP attacks (SQLi, XSS) |
| SIEM | Security Information and Event Management | Log collection and correlation |
| HTTP/HTTPS | Hypertext Transfer Protocol (Secure) | Web protocol, ports 80/443 |
| SMTP | Simple Mail Transfer Protocol | Email delivery, port 25 |
| DNS | Domain Name System | Names to IP addresses |
| TLS | Transport Layer Security | Encrypts client-server traffic |
| URL | Uniform Resource Locator | Address of a web resource |
| B2B | Business-to-Business | Data exchange between organizations |

## 6. Pros and cons

**Pros**
- Internal network is not directly exposed to the internet.
- A compromised public server does not automatically expose internal resources.
- Fine-grained access control and logging per zone.
- One place to monitor exposed services; supports compliance (PCI DSS, NIST SP 800-41).
- Enables safe partner access (extranet style).

**Cons**
- Extra cost (firewalls, IPS/IDS, servers, licenses).
- Complex rules and maintenance; misconfiguration creates holes.
- Does not stop insiders or attacks hidden in allowed or encrypted traffic.
- The DMZ itself is a target and must be patched and hardened.
- Latency and possible single point of failure without redundancy.

## 7. Where DMZs are deployed

- **Commercial**: banks, e-commerce, cloud/SaaS (web, API, mail in DMZ; PCI DSS separation from cardholder data).
- **Defense and government**: boundary protection (NIST SP 800-53 SC-7, SP 800-41), public sites, email gateways, cross-domain guards.
- **Industrial and energy**: IT/OT DMZ (ISA/IEC 62443 zones and conduits, Purdue model level 3.5).
- **Healthcare**: patient portals and telehealth in DMZ, records in interior LAN.
- **Education and research**: public web/mail/library services, partner portals.
- **Telecom, ISPs, enterprises**: public DNS, mail relays, VPN concentrators, reverse proxies, supplier portals.

*Note: the interactive animation is in the web page; this file is its text equivalent.*
