# System Security Models: Study Guide

Companion to `security-models-simulator.html`.

## System Security Models: Bell-LaPadula and Biba

Formal models give a theoretical framework for access control and each stresses a different property. Bell-LaPadula protects confidentiality; Biba protects integrity. The two are mirror images, so try both in the simulator.

### Working principle

1. A subject operating at a security level (e.g. Secret) wants to act on an object with its own level.
2. It requests read or write.
3. Bell-LaPadula (confidentiality): no read up, and no write down so data cannot leak to a lower level. This stops your bank’s system from showing you someone else’s private account.
4. Biba (integrity): no write up (simple integrity or “star” property) and no read down, so low-integrity data cannot corrupt high-integrity data. A public-disclosure process cannot write into safety-critical data.
5. Only requests that satisfy the model’s rules reach the object.

### Pros

- Mathematically precise, provable rules
- Bell-LaPadula stops information leaks
- Biba stops corruption of trusted data

### Cons

- Bell-LaPadula ignores integrity; Biba ignores confidentiality
- Rigid and hard to use in everyday business
- Needs every subject and object labelled

### Where it is deployed

- **Commercial:** Banking data separation, integrity levels in Windows (Biba-like Mandatory Integrity Control)
- **Defense:** Classified multilevel systems (Bell-LaPadula origin: US DoD)
- **Other:** Avionics and medical device software (integrity), regulated records

## Discretionary Access Control (DAC)

The owner of an object decides who may access it, usually through an access control list (ACL). Flexible and fine-grained, but only as safe as each owner’s diligence.

### Working principle

1. The owner creates a file and sets its ACL: which users or groups get read, write or execute.
2. A subject (e.g. Steve) requests an action on the file.
3. The OS looks up the subject–object table. Column = the object’s ACL; row = the subject’s capabilities list.
4. Permitted actions proceed; others are denied.
5. The owner may later change the ACL, add users or pass permissions on to others, which is the “discretion”.

### Pros

- Flexible and user-friendly
- Fine-grained per user or group
- Fast lookups, built into Windows and Unix

### Cons

- Inconsistent or insecure if owners are careless
- No central management or auditing
- Hard to scale and to trace problems
- Permissions can be passed on (leakage)

### Where it is deployed

- **Commercial:** Unix/Linux permissions (drwxr-xr-x), Windows NTFS ACLs, cloud-drive sharing
- **Defense:** Used for unclassified data, constrained by MAC (DoD TCSEC 5200.28-STD)
- **Other:** Personal computers, university file servers, collaboration tools

## Mandatory Access Control (MAC)

Access is decided by a central policy, not the owner. Subjects and objects carry labels (Top Secret, Secret, Confidential, Unclassified); access needs the subject’s label to dominate the object’s. DAC permission does not override it.

### Working principle

1. A security administrator defines the policy and assigns labels to every subject and object.
2. A subject requests access.
3. The OS compares the two labels against the policy rules.
4. If the subject’s label dominates (equal or higher), access is allowed; otherwise it is denied even if DAC would allow it.
5. Information can only flow from lower to higher levels, preventing disclosure, modification or destruction.

### Pros

- Strongest consistency and enforceability
- Owners cannot leak data by mistake
- Central, auditable policy

### Cons

- Heavy administration and labelling
- Inflexible; hard for collaboration
- Needs OS support and trained staff

### Where it is deployed

- **Commercial:** SELinux on Red Hat and Android, regulated finance and healthcare
- **Defense:** Classified government and military networks (Top Secret to Unclassified)
- **Other:** Critical infrastructure, hardened servers

## Subject-Based and Object-Based Access Control

SBAC decides by attributes of the user (identity, group, role, clearance, department, location, time, device). OBAC decides by attributes of the resource (type, classification, sensitivity, owner, location).

### Working principle

1. A subject asks for a resource.
2. SBAC: the engine evaluates user attributes such as group, clearance or job title, statically or in real time (location, time, device).
3. OBAC: it evaluates object attributes, such as classification public, internal, confidential or highly sensitive, set by the data owner.
4. Policies combine both and grant or deny; ACLs or security labels enforce.

### Pros

- Granular and tailored
- Static or dynamic decisions
- Supports compliance and reduces breach risk

### Cons

- High administrative overhead as attributes multiply
- Labels and attributes must stay accurate
- Policy complexity

### Where it is deployed

- **Commercial:** Document sensitivity labels (public/internal/confidential), data-loss prevention
- **Defense:** Clearance plus classification markings
- **Other:** Healthcare and legal records, regulated industries

## Role-Based Access Control (RBAC)

Permissions are attached to roles, and users are assigned roles that match their job. The most widely used model in modern IT, simple, manageable and scalable, and it enforces least privilege.

### Working principle

1. Identify roles from job functions, granular enough to be accurate but avoiding role proliferation.
2. Give each role only the permissions it needs (least privilege); data owners choose which roles may access objects.
3. Place each user in one or more roles.
4. On a request the system checks the user’s roles, not the individual.
5. Regularly review assignments, revoke extra privileges and update roles as jobs change.

### Pros

- Simple, scalable, flexible
- Enforces least privilege; reduces insider and breach risk
- Lower admin effort: manage roles, not individuals

### Cons

- Defining roles accurately is difficult
- Role explosion: too many granular roles
- Needs regular review

### Where it is deployed

- **Commercial:** Active Directory groups, AWS/Azure roles, ERP systems, banks
- **Defense:** Role-separated duties in command and logistics systems
- **Other:** Hospitals (doctor, nurse, admin), universities, government portals

## Privileged Access Management (PAM)

Processes and technology to monitor and control privileged accounts (system, network, database, security administrators). Controls limit damage from a malicious or careless admin and from a stolen admin account. RBAC can implement it.

### Working principle

1. Identify and define privileged roles with least-privilege permissions.
2. Privilege escalation control: an authorised person must approve elevation.
3. Multifactor authentication protects the privileged login.
4. Just-in-time: temporary privileges are granted only for the task.
5. Session monitoring and logging record commands and changes; segregation of duties stops one person holding too much power.
6. Privileges are revoked when the task ends; periodic audits confirm rights are correct.

### Pros

- Limits insider and attacker damage
- Full accountability through session logs
- Reduces standing privileges

### Cons

- Friction for admins; approval delays
- Costly to deploy and operate
- The vault itself is a high-value target

### Where it is deployed

- **Commercial:** Banks and cloud root/admin accounts, vaults like CyberArk and BeyondTrust
- **Defense:** Contractor and administrator control in classified environments
- **Other:** Healthcare IT, utilities, managed service providers

## Rule-Based Access Control (RuBAC)

Access depends on predefined rules about conditions: group membership, time, location or other context. The system evaluates the rules dynamically on each request.

### Working principle

1. Administrators define rules such as “only from the office, 08:00–18:00”.
2. A user or group attempts access.
3. The system evaluates each rule against current conditions.
4. All rules satisfied: grant. Any rule failed: deny.

### Pros

- Simple, dynamic and context-aware
- Easy to automate and audit
- Consistent enforcement

### Cons

- Rules can conflict or grow unwieldy
- Coarser than full ABAC
- Requires ongoing rule maintenance

### Where it is deployed

- **Commercial:** Firewall and router rules, bank transaction limits, time-based logins
- **Defense:** Network filtering, duty-hours access to facilities
- **Other:** School networks, parental controls, building badge schedules

## Attribute-Based Access Control (ABAC)

A centralised form of rule-based control. At runtime it evaluates attributes of the user, the resource and the environment (where, when, who, role, action, resource, threat, recent actions) against policies.

### Working principle

1. Users, remote or local, make access attempts.
2. The ABAC engine gathers attributes for subject, object and environment.
3. Multiple go/no-go tests run. Example: a cardiology nurse may open only cardiology patient records; a masked connection (e.g. TOR) is flagged, as PayPal does.
4. All pass: access approved, possibly with restrictions.
5. Any fail: deny, trigger a further challenge such as MFA, or alert security.

### Pros

- Very fine-grained and context-aware
- Handles complex policy
- Dynamic, runtime decisions

### Cons

- Complex to design and test
- Needs accurate, well-governed attributes
- Performance and audit complexity

### Where it is deployed

- **Commercial:** PayPal-style fraud flagging, adaptive cloud access policies
- **Defense:** Coalition data sharing by clearance, nationality and mission
- **Other:** Hospitals (department matching), government data portals

## Risk-Based Access Control

A near real-time process where a senior risk manager adjusts organisation-wide risk tolerance using “throttles” tied to families of administrative, technical and physical controls. It acts like ABAC with higher-level attributes.

### Working principle

1. Precursors appear: expected traffic surge, rising attacker interest, or a physical threat.
2. The risk manager moves a throttle that adjusts a set of controls together, not device by device.
3. SOAR (Security Orchestration, Automation and Response) pushes the changes across tools.
4. Traffic surge: reduce friction. Attack risk: harden network/app/web and warn staff about phishing. Physical risk: raise physical security.

### Pros

- Fast organisation-wide response
- Avoids tuning every appliance
- Balances security and usability

### Cons

- Needs integrated, mature tooling
- Depends on good risk judgement
- Over- or under-tightening has business cost

### Where it is deployed

- **Commercial:** Adaptive/conditional access, SOCs, online retail during sales peaks
- **Defense:** Threat-level (alert-state) posture changes
- **Other:** Critical infrastructure, financial exchanges, healthcare during outbreaks

## Terminology: full forms

| Abbreviation | Full form | Meaning |
|---|---|---|
| DAC | Discretionary Access Control | Owner decides who gets access |
| MAC | Mandatory Access Control | Central policy and labels decide |
| RBAC | Role-Based Access Control | Access via job roles |
| PAM | Privileged Access Management | Control of admin-level accounts |
| RuBAC | Rule-Based Access Control | Access via predefined conditions |
| ABAC | Attribute-Based Access Control | Access via user, resource and environment attributes |
| SBAC | Subject-Based Access Control | Decisions from user attributes |
| OBAC | Object-Based Access Control | Decisions from resource attributes |
| BLP | Bell-LaPadula | Confidentiality model: no read up, no write down |
| Biba | Biba integrity model (K. Biba) | Integrity model: no write up, no read down |
| ACL | Access Control List | Per-object list of permissions |
| SoD | Segregation (Separation) of Duties | No one person holds too much power |
| JIT | Just-in-Time access | Temporary privileges for a task |
| MFA | Multifactor Authentication | Two or more identity proofs |
| SOAR | Security Orchestration, Automation and Response | Automates coordinated security actions |
| VPN | Virtual Private Network | Encrypted tunnel that can mask IP |
| TOR | The Onion Router | Anonymising network with hopping connections |
| TCSEC | Trusted Computer System Evaluation Criteria | US DoD 5200.28-STD |
| SELinux | Security-Enhanced Linux | MAC implementation for Linux |
| DoD | Department of Defense | US defense department |
| R / W / X | Read / Write / eXecute | Basic file permissions |
| IAM | Identity and Access Management | Managing identities and permissions |
