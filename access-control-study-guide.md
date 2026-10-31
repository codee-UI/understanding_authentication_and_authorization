# Access Control, Identity and Reference Monitor: Study Guide

Companion to `access-control-simulator.html`.

## Access Control Fundamental Concepts

Access control is the set of procedures that permit access to controlled areas and information according to policy. Every decision involves three elements: subjects, objects and rules.

### Working principle

1. A subject (user, process, program, device) decides it needs a service. Subjects are active and should carry a permission or clearance level (least privilege).
2. The subject sends a request. An unknown subject with no permissions must be denied everything.
3. Objects hold no access logic of their own, so a separate layer (IAM, OS, reference monitor) intercepts the request.
4. The layer checks the rules the object’s owner recorded in the rule base or access control list.
5. If permitted, the passive object responds to the request; otherwise it is never touched.
6. Roles flip: a user clicking Delete is a subject; the file system receiving it is an object, but when it updates the disk directory it becomes a subject. Subject = seeking, object = sought.

### Pros

- Enforces least privilege and a layered defence
- Covers physical and logical assets (buildings, server rooms, networks, apps)
- Owner-defined rules give clear accountability
- Rules can be audited and reviewed

### Cons

- Stale permissions drift from current job duties
- Large rule bases are hard to maintain
- Failure or compromise of the control layer exposes everything
- Objects cannot protect themselves

### Where it is deployed

- **Commercial:** Banking apps, ERP roles, badge-controlled offices, cloud IAM policies
- **Defense:** Clearance levels and need-to-know on classified networks, secure facilities
- **Other:** Hospital records, university LMS and grade systems, smart buildings

## Reference Monitor Concept

An abstract machine that mediates every access by a subject to an object, using owner-defined rules, and logs every attempt. First described by James P. Anderson (1972, Computer Security Technology Planning Study). It must be small and simple. Try the simulation below.

### Working principle

1. Subject requests access to data on the server.
2. Reference monitor intercepts the request (complete mediation).
3. It checks the rule base written by the data owner.
4. It decides: permit or deny.
5. Permitted: subject reaches the object. Denied: object is untouched.
6. The attempt, permitted or denied, is logged.

### Required properties

- **Complete mediation**: Every access to a protected resource is checked, with no bypass path.
- **Tamperproof**: Immune to modification by subjects or attackers.
- **Verifiable**: Small enough that its correctness can be proven.

### Pros

- Single enforcement point is easier to reason about
- Complete audit trail
- Policy separated from the objects
- Provable security when kept small

### Cons

- Performance cost on every access
- Only an abstract ideal; real systems approximate it
- A bug or bypass voids all guarantees
- Hard to keep minimal as features grow

### Where it is deployed

- **Commercial:** Windows Security Reference Monitor, SELinux/AppArmor, cloud API authorisers
- **Defense:** High-assurance and multilevel secure systems, cross-domain guards
- **Other:** Banking mainframes (RACF), healthcare record systems

## TCB, Reference Monitor and Security Kernel

The Trusted Computing Base (TCB) is everything that must be trusted to enforce policy. The security kernel is the practical implementation of the reference monitor inside the TCB, extending into hardware (TPM) and firmware (UEFI).

### Working principle

1. Hardware such as the TPM provides cryptographic functions and secure key storage.
2. Firmware (BIOS/UEFI) starts first and verifies what it loads.
3. Secure boot ensures only trusted, verified software loads at startup.
4. The security kernel starts and provides access control, audit and integrity (self-protection).
5. The kernel implements the reference monitor: complete mediation, tamperproof, verifiable.
6. All user and program requests pass through the kernel, which stays modular, isolated and fault tolerant.

### Implementation by platform

| Platform | TCB | Reference monitor | Security kernel |
|---|---|---|---|
| Windows | NT kernel, drivers, Secure Boot, TPM | Security Reference Monitor (ACLs, auditing) | DEP, ASLR, Credential Guard |
| Apple | XNU kernel, Secure Enclave, T2 chip | Security Framework, sandbox, Gatekeeper | XNU, SIP, Gatekeeper |
| Linux | Kernel, glibc, core services (systemd) | LSM framework: SELinux, AppArmor | LSM modules, capabilities |
| Mainframe | z/OS kernel, zSeries CPUs, firmware | SAF with RACF | z/OS security kernel, crypto services |

### Pros

- Strong, hardware-rooted trust
- Modularity, isolation, fault tolerance
- Self-protecting integrity
- Origins in Multics (Honeywell, 1970s)

### Cons

- Larger TCB means more attack surface
- Needs trustworthy hardware and firmware
- Costly to design, certify and update
- Kernel flaws affect the whole system

### Where it is deployed

- **Commercial:** Laptops and phones with TPM/Secure Enclave, cloud hypervisors, ATMs
- **Defense:** Trusted OS and hardened Linux (SELinux originated with NSA), avionics
- **Other:** IBM z/OS in banks and insurers, medical devices, IoT secure boot

## Identity as a Service (IDaaS)

A cloud subscription in which a third party provides IAM: single sign-on, provisioning and deprovisioning for cloud and on-premises systems. Gartner 2024 leaders listed: Ping Identity, Microsoft Entra ID, Okta.

### Working principle

1. User opens an application and is redirected to the IDaaS login.
2. The provider authenticates the user (password, MFA) against its directory.
3. It checks current role and access policies to decide permissions.
4. It issues a signed token (SAML or OIDC) so the user gets single sign-on.
5. The application trusts the token and serves data held by the CSP under agreed SLAs.
6. On joiner, mover or leaver events the provider provisions or deprovisions accounts automatically.

### Pros

- One login across many systems (SSO)
- Lower infrastructure cost; vendor keeps it patched
- Automated provisioning removes leftover access
- Scales with cloud adoption

### Cons

- Depends on a third party: outage or compromise hits everyone
- Needs SLAs and compliance assurance for data
- Stale permissions still a risk if roles are not reviewed
- Vendor lock-in

### Where it is deployed

- **Commercial:** SaaS suites (Microsoft 365, Salesforce), retail and fintech workforce logins
- **Defense:** Government-cloud IAM, contractor federation (with accredited providers)
- **Other:** Universities (staff and student SSO), healthcare, e-government portals

## Authentication and Authorization Vulnerabilities

The identification, authentication, authorization and accounting (IAAA) systems are the main battlefield. Select an attack to step through it; for IDOR, try the live demo.

### Authentication bypass

1. Attacker looks for a way around the login: physical USB in a data centre, a router with flashable firmware, or an odd feature-activation sequence.
2. Example: “AirBnB attack”, flashing an attacker-friendly firmware onto a rental or hotel router.
3. The system trusts the path and never asks for authentication.

**Defence:** Physical controls, signed firmware, no implicit trust of inside paths.

### Forced authentication

1. Machine-in-the-middle intercepts the dialogue; spear phishing sends documents with links to the attacker’s server.
2. The target is tricked into authenticating to the attacker, who captures credential hashes.
3. Cracked hashes reveal valid identities for further access.

**Defence:** Block outbound auth to unknown hosts, MFA, email filtering.

### Authorization bypass / forced browsing

1. Attacker overrides a URL the app generated (forced browsing).
2. The endpoint is poorly defended and returns content.
3. Weak privilege-escalation control opens the wider environment.

**Defence:** Check authorization on every request, server side.

### IDOR

1. Attacker sees ?accountId=1&admin=true in the URL.
2. Changes it to accountId=2 to see what happens.
3. Without an ownership check the server returns account 2 with no IAAA involvement.

**Defence:** Verify the object belongs to the session user; use indirect references.

### Direct API interaction

1. Like IDOR but done at byte level against API parameters.
2. Attacker experiments with values, IDs and fields.
3. Lax server checks return or change data.

**Defence:** Schema validation, per-object authorization, rate limits.

### TOCTOU

1. Time of Check: privilege is validated and granted.
2. In the narrow window, attacker swaps the object or exploits the flaw.
3. Time of Use: the stale permission is applied to the wrong state. Detection needs logs correlated to a trusted time server.

**Defence:** Atomic check-and-use, locking, trusted reference clock.

### Plugin / DLL substitution

1. Unmanaged systems accept corrupted .dll, PAM modules or Mac authorization plugins.
2. The substitute code replaces trustworthy code.
3. The security kernel and reference monitor trusted path is broken; the attacker owns the system.

**Defence:** Code signing, secure boot, managed and monitored systems.

### Why they work

- Why attacks work: design or config errors, trust assumptions, weak endpoint checks
- Lists are ephemeral: new ones appear as old ones close

### Defences

- Defences: complete mediation, MFA, least privilege, server-side checks, signed firmware, time-synchronised logs

### Where it is deployed

- **Commercial:** Web and mobile banking, e-commerce, SaaS APIs
- **Defense:** Insider and supply-chain threats, hotel and travel networks
- **Other:** Healthcare portals, university systems, IoT

## Terminology: full forms

| Abbreviation | Full form | Meaning |
|---|---|---|
| IAAA | Identification, Authentication, Authorization, Accounting | Four linked processes that decide who you are, prove it, set what you may do, and record it |
| IDaaS | Identity as a Service | Cloud-subscription identity and access management |
| IAM | Identity and Access Management | Policies and tools for digital identities and permissions |
| SSO | Single Sign-On | One login for many applications |
| CSP | Cloud Service Provider | Company hosting your data or services |
| SLA | Service-Level Agreement | Contract defining service and security obligations |
| MFA | Multi-Factor Authentication | Two or more proofs of identity |
| SAML | Security Assertion Markup Language | XML standard for SSO assertions |
| OIDC | OpenID Connect | Identity layer on OAuth 2.0 for SSO |
| SCIM | System for Cross-domain Identity Management | Standard for automatic provisioning |
| ACL | Access Control List | Per-object list of who may do what |
| RBAC | Role-Based Access Control | Permissions assigned through roles |
| DAC / MAC | Discretionary / Mandatory Access Control | Owner decides vs. system-enforced labels |
| TCB | Trusted Computing Base | All components that must be trusted for security |
| RM | Reference Monitor | Abstract machine mediating all access |
| TPM | Trusted Platform Module | Hardware chip for keys and crypto |
| BIOS / UEFI | Basic Input/Output System / Unified Extensible Firmware Interface | Firmware that boots the machine |
| SRM | Security Reference Monitor | Windows kernel access-check component |
| LSM | Linux Security Modules | Kernel framework hosting SELinux, AppArmor |
| SELinux | Security-Enhanced Linux | Mandatory access control for Linux |
| SAF | System Authorization Facility | z/OS security interface |
| RACF | Resource Access Control Facility | IBM mainframe access manager |
| XNU | X is Not Unix | Apple’s macOS kernel |
| SIP | System Integrity Protection | macOS protection of system files |
| DEP | Data Execution Prevention | Blocks code running from data memory |
| ASLR | Address Space Layout Randomization | Randomises memory layout |
| IDOR | Insecure Direct Object Reference | Changing an ID to reach others’ data |
| API | Application Programming Interface | Programmatic service access |
| TOCTOU | Time of Check v. Time of Use | Race between check and use |
| MITM | Machine-in-the-Middle | Interceptor between two parties |
| DLL | Dynamic-Link Library | Windows shared code module |
| PAM | Pluggable Authentication Modules | Unix/Linux auth plugins |
| pwned | Pronounced “pawned” | Identity whose ownership was taken over |
| Multics | Multiplexed Information and Computing Service | 1960s–70s OS pioneering security kernels |
