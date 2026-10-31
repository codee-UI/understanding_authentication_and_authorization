# Access Control: Concepts, Working Principles, Pros/Cons and Applications

Companion to the interactive simulator. Same content, text form.


## Authentication


### Single & Multi-Factor Authentication (SFA/MFA)

**Working principle (step by step)**

1. **U → S** (login ID): Identification: the user claims an identity (username).
2. **U → P** (password): Factor 1, something you know.
3. **P → S** (✓ factor 1): With only one factor this is SFA, the weakest form.
4. **U → T** (OTP code): Factor 2, something you have (token, smart card, phone).
5. **T → S** (✓ factor 2): Two different factor types = MFA, the best practice.
6. **U → B** (fingerprint): Factor 3, something you are (biometrics).
7. **B → S** (✓ factor 3): Every factor verified.
8. **S → U** (ACCESS GRANTED): Stolen password alone is no longer enough.

**Pros:** Stops most credential-theft attacks; Layers can be added by risk; Required by many regulations

**Cons:** Friction for users; Lost token/phone = lockout; Biometrics cannot be reset if leaked

**Applications:** Banking apps (password + SMS/app code); Military CAC smart cards + PIN; Cloud admin consoles, airport e-gates (face)


### Single Sign-On (SSO)

**Working principle (step by step)**

1. **U → A** (open App A): User has no session yet.
2. **A → I** (redirect): App A trusts the IdP and sends the user there.
3. **U → I** (login once): Credentials are entered a single time.
4. **I → A** (signed token): IdP issues a ticket/token (Kerberos ticket, OIDC ID token).
5. **U → B** (open App B): Second app, different system.
6. **B → I** (check session): App B asks the IdP instead of the user.
7. **I → B** (✓ token): Existing session is reused.
8. **B → U** (NO re-login): Fewer passwords, less password fatigue.

**Pros:** One login, fewer passwords; Central policy and logging; Instant central revocation

**Cons:** IdP is a single point of failure; One compromised session opens many apps; Legacy apps need adapters

**Applications:** Corporate Okta/Azure AD portals; Kerberos in Windows domains; Hospitals: tap-in badge to open all clinical apps; University campus services


### Device Authentication

**Working principle (step by step)**

1. **T → D** (key in chip): The TPM safely stores the device private key.
2. **D → N** (MAC + IP + cert): Device presents physical (MAC) and logical (IP) address plus a digital certificate.
3. **N → C** (validate cert): Network asks the CA whether the certificate is genuine.
4. **C → N** (✓ trusted): Certificate chain and validity confirmed.
5. **N → D** (NETWORK ACCESS): Device joins the network.
6. **R → N** (✗ unknown MAC): No valid certificate.
7. **N → R** (✗ BLOCKED): MAC addresses alone can be spoofed, so certificates plus TPM matter.

**Pros:** Blocks unmanaged devices; Works without user action; TPM makes keys hard to steal

**Cons:** MAC/IP can be spoofed alone; Certificate lifecycle workload; Hardware replacement complexity

**Applications:** 802.1X Wi-Fi in enterprises; Defence: only registered laptops on classified LANs; IoT/OT device onboarding; Windows device health attestation


### Federated Access (SAML / OAuth 2)

**Working principle (step by step)**

1. **U → S** (open service): User from partner org wants a service.
2. **S → I** (SAML request / OAuth authz): Service trusts an external IdP within the federation.
3. **I → U** (login prompt): User authenticates at their own home organization.
4. **U → I** (credentials): Service never sees the password.
5. **I → S** (assertion / token): SAML assertion (identity) or OAuth token (delegated authorization).
6. **S → U** (ACCESS): Identity shared across domains without new accounts.

**Pros:** No duplicate accounts; Password stays at home org; Great for partnerships

**Cons:** Trust agreements needed; Misconfiguration = token forgery; Complex debugging

**Applications:** Universities (eduroam, research portals); Government inter-agency portals; Sign in with Google/Microsoft; Defence coalition information sharing


## Trust


### PKI & Secure Protocols (TLS/HTTPS)

**Working principle (step by step)**

1. **S → C** (CSR + public key): Server asks the CA to certify its public key.
2. **C → S** (signed certificate): CA signs with its private key.
3. **B → S** (HTTPS hello): Client starts a TLS handshake.
4. **S → B** (certificate): Server proves its identity.
5. **B → C** (verify signature): Browser checks the CA is trusted and not revoked.
6. **B → S** (session key): A symmetric key is exchanged securely.
7. **S → K** (encrypted data): Data confidentiality and integrity achieved.

**Pros:** Authenticates servers/users; Encrypts traffic; Global, standardised

**Cons:** CA compromise breaks trust; Revocation checks are weak; Key management effort

**Applications:** Online banking and e-commerce; Government e-ID and digital signatures; Military PKI for secure email; Code signing


### Zero Trust

**Working principle (step by step)**

1. **U → P** (request): Location (inside/outside) gives no trust.
2. **P → D** (who? device? risk?): Every request is verified: identity, device health, context.
3. **D → P** (✓ allow (least privilege)): Only the minimum access is given.
4. **P → R** (short-lived session): Access is narrow and time-limited.
5. **A → R** (✗ already inside): Being on the internal network grants nothing.
6. **R → D** (continuous monitoring): Behaviour is rechecked; access is revoked on anomalies.

**Pros:** Limits lateral movement; Fits cloud and remote work; Applies to apps, data, CI/CD, supply chain

**Cons:** Complex migration; Needs good asset inventory; Can add latency/friction

**Applications:** Google BeyondCorp; US DoD zero trust strategy; Banks protecting remote staff; DevOps pipelines (signed builds)


### Trust Relationships (one-way, two-way, transitive)

**Working principle (step by step)**

1. **B → A** (one-way): A trusts B: B users may use A resources, not the reverse.
2. **A → B** (✗ reverse blocked): One-way trust is unidirectional.
3. **A → B** (two-way): Now B also trusts A, so both can access each other.
4. **B → C** (B trusts C): A second link is created.
5. **C → A** (transitive): A trusts B, B trusts C, so A trusts C automatically (AD forest).

**Pros:** Transitive trust simplifies management; One-way limits exposure; Single sign-on across domains

**Cons:** Transitive trust spreads compromise; Hard to audit chains; Unintended access

**Applications:** Active Directory forests; SAML federations; Merger/acquisition integration; Partner supply chains


### Internet, Intranet, Extranet & DMZ

**Working principle (step by step)**

1. **I → F** (HTTPS request): Public traffic arrives at the perimeter.
2. **F → W** (allow 443 only): Public services live in the DMZ buffer zone.
3. **W → N** (limited DB query): A second, strict rule reaches internals.
4. **I → N** (✗ direct access): Internet can never reach the intranet directly.
5. **P → F** (VPN login): Extranet: external partner is authenticated.
6. **F → N** (specific app only): Partner sees only agreed resources.

**Pros:** Contains breaches; Public services are separated; Controlled B2B sharing

**Cons:** Firewall rule complexity; DMZ host compromise still serious; Maintenance cost

**Applications:** E-commerce web farms; Defence cross-domain gateways; Supplier portals (automotive); Hospital telehealth gateways


### Third-Party Connections (API, Extensions, Middleware)

**Working principle (step by step)**

1. **G → A** (extends features): Application extension/plugin adds capability.
2. **A → P** (API call + token): Standard rules and data formats.
3. **P → M** (route request): Gateway checks authN/authZ and rate limits.
4. **M → X** (translate protocol): Middleware queues and transforms messages.
5. **X → M** (response): Back through the layers.
6. **P → A** (JSON result): Third parties extend systems but widen attack surface.

**Pros:** Fast integration; Reuse of external services; Legacy interoperability

**Cons:** Third-party risk; API abuse/leaks; Plugin vulnerabilities

**Applications:** Payment APIs (Stripe); Banking open-banking APIs; ERP middleware in manufacturing; Browser extensions


## IAM Lifecycle


### IAM Life Cycle (IAAA)

**Working principle (step by step)**

1. **H → F** (ID documents): Identification and proofing: verify the person is who they claim.
2. **F → V** (✓ verified): Confidence established before access.
3. **V → D** (create account): Provisioning: account, ID and rights created.
4. **D → A** (authenticate + authorize): The IAAA steps happen at each access.
5. **A → L** (accounting): Actions are logged for audit and compliance.
6. **L → V** (review: role change): Monitoring/maintenance spots mismatches.
7. **V → D** (update entitlements): Rights follow the role.
8. **H → V** (employee leaves): Deprovisioning trigger.
9. **V → D** (✗ disable account): Teardown prevents orphan accounts.

**Pros:** Automation reduces admin work; Faster onboarding; Audit-ready

**Cons:** Integration is expensive; Stale data causes errors; Needs HR process discipline

**Applications:** Enterprise SailPoint/Okta/Entra; Defence personnel clearance onboarding; Hospital staff rotations; Banks SOX compliance


### Authorization & Entitlements

**Working principle (step by step)**

1. **U → G** (member of Finance): Group membership.
2. **G → R** (inherited: read): Inherited right from group.
3. **U → R** (explicit: write): Explicit right assigned directly to this user.
4. **R → U** (effective = inherited + explicit): Policy evaluation, then enforcement.
5. **U → Q** (✗ no entitlement): Access denied and logged (accountability).

**Pros:** Groups simplify management; Clear audit trail; Least privilege possible

**Cons:** Entitlement creep over time; Inheritance can be confusing; Needs reviews

**Applications:** Windows NTFS permissions; Cloud IAM policies; SOX segregation of duties; Hospital record access


## Access Control


### Mandatory Access Control (MAC)

**Working principle (step by step)**

1. **S → K** (read Secret doc): All subjects and objects carry security labels.
2. **K → E** (✓ clearance ≥ label): Central rule allows it.
3. **S → K** (read Top Secret): Request above clearance.
4. **K → T** (✗ DENIED): Even the document owner cannot override this.
5. **S → K** (change a label): User tries to modify permissions.
6. **K → S** (✗ not allowed): Only the policy administrator may change rules.

**Pros:** Strongest control; Stops owner leaks; Predictable

**Cons:** Rigid and costly; Heavy admin; Poor flexibility

**Applications:** Military/government classified systems; SELinux; Intelligence networks; Nuclear/critical infrastructure


### Discretionary Access Control (DAC)

**Working principle (step by step)**

1. **O → F** (creates file): The creator becomes the owner.
2. **O → B** (grants Read): Owner decides at own discretion.
3. **B → F** (✓ read): B allowed.
4. **C → F** (✗ denied): C has no grant.
5. **O → C** (grants Write): Owner extends access.
6. **C → F** (✓ write): C allowed.

**Pros:** Flexible, easy; Quick sharing; Low admin

**Cons:** Owner mistakes spread access; Malware inherits user rights; Hard to audit

**Applications:** Linux/Unix file permissions; Windows file sharing; Google Drive sharing; Small business folders


### Role-Based Access Control (RBAC)

**Working principle (step by step)**

1. **A → R** (defines role perms): Permissions attach to roles.
2. **A → U** (assigns role): Users get roles by job function.
3. **U → R** (activate role): User acts through the role.
4. **R → P** (view, add notes): Role bundles permissions.
5. **P → X** (✓ view chart): Allowed.
6. **U → X** (✗ delete record): Not in role, least privilege enforced.

**Pros:** Simple, scalable; Enforces least privilege; Easy audits

**Cons:** Role explosion; Little context awareness; Roles need upkeep

**Applications:** Hospitals and EMRs; Banking teller/manager roles; AWS/Azure RBAC; Enterprise ERP


### Rule-Based Access Control (RuBAC)

**Working principle (step by step)**

1. **U → E** (request at 22:00): Request arrives.
2. **E → L** (check rules): Predefined conditions evaluated.
3. **L → E** (✗ outside hours): Rule fails.
4. **U → E** (request at 10:00): Try again in hours.
5. **L → E** (✓ rule passes): Condition met.
6. **E → X** (ACCESS): Granted by rules, not identity alone.

**Pros:** Easy to automate; Consistent decisions; Good for time/IP limits

**Cons:** Rules multiply; Rigid; Conflicts possible

**Applications:** Firewall ACLs; Banking after-hours limits; Network router policies; Parental controls


### Attribute-Based Access Control (ABAC)

**Working principle (step by step)**

1. **S → E** (request): Policy Enforcement Point intercepts.
2. **E → D** (authz query): Policy Decision Point evaluates policy.
3. **D → I** (fetch attributes): Dept, title, sensitivity, time, location.
4. **I → D** (attributes): Context returned.
5. **D → E** (✓ allow read-only): Allowed with restrictions.
6. **E → R** (ACCESS): Enforced.
7. **D → E** (✗ if location = foreign): Same user, different context = denied.

**Pros:** Very fine-grained; Dynamic, contextual; Supports zero trust

**Cons:** Complex policy design; Hard to debug; Performance cost

**Applications:** XACML systems; Defence coalition data sharing; Healthcare consent policies; Cloud ABAC (AWS tags)


## Terminology: Full Forms

| Term | Full form | Meaning |
|---|---|---|
| IAM | Identity and Access Management | Automates account/access processes |
| IAAA | Identification, Authentication, Authorization, Accounting | Core identity model |
| SFA | Single-Factor Authentication | One factor only |
| MFA | Multi-Factor Authentication | 2+ factor types |
| OTP | One-Time Password | Single-use code |
| SSO | Single Sign-On | Log in once, access many |
| IdP | Identity Provider | Authenticates users |
| SP | Service Provider | Relies on IdP |
| SAML | Security Assertion Markup Language | XML identity assertions |
| OAuth 2 | Open Authorization 2 | Delegated access tokens |
| OIDC | OpenID Connect | Identity layer on OAuth 2 |
| ADFS | Active Directory Federation Services | Microsoft federation |
| AD | Active Directory | Microsoft directory |
| Azure AD | Microsoft Azure Active Directory (Entra ID) | Cloud identity |
| MAC (net) | Media Access Control address | Hardware address |
| IP | Internet Protocol | Logical address |
| TPM | Trusted Platform Module | Hardware key chip |
| FIM | Federated Identity Management | Cross-domain identity |
| PKI | Public Key Infrastructure | Certificates and CAs |
| CA | Certificate Authority | Issues certificates |
| CSR | Certificate Signing Request | Request for a certificate |
| TLS | Transport Layer Security | Encrypts traffic |
| SSL | Secure Sockets Layer | TLS predecessor |
| HTTPS | Hypertext Transfer Protocol Secure | HTTP over TLS |
| TCP/IP | Transmission Control Protocol / Internet Protocol | Internet suite |
| DMZ | Demilitarized Zone | Buffer network |
| VPN | Virtual Private Network | Encrypted remote link |
| NAC | Network Access Control | Device admission |
| API | Application Programming Interface | Software communication rules |
| B2B | Business-to-Business | Partner interactions |
| CI/CD | Continuous Integration / Continuous Deployment | Software pipeline |
| MAC (ac) | Mandatory Access Control | Central labels |
| DAC | Discretionary Access Control | Owner decides |
| RBAC | Role-Based Access Control | Role permissions |
| RuBAC | Rule-Based Access Control | Predefined rules |
| ABAC | Attribute-Based Access Control | Attribute policies |
| PEP | Policy Enforcement Point | Enforces decision |
| PDP | Policy Decision Point | Makes decision |
| XACML | eXtensible Access Control Markup Language | ABAC policy standard |
| SOX | Sarbanes-Oxley Act | Financial controls law |