# IAM: Identity, Entitlement, Monitoring, Reporting and Maintenance

## 1. Life cycle (the simulation's flow)
Create identity → Provision access → Authenticate → **Maintain** (monitor, report, adjust) → Retire (revoke).
Maintenance loops back to provisioning whenever roles or standards change.

## 2. Entities needing digital identities
People, Devices, Organizations, Code, Agents.
Example: Aisling (field technician) has several personal identities; her laptop (MAC/serial), OS processes, applications and monitoring agents (EDR, antimalware) are separate entities. An **agent** acts on another's behalf using its own identity plus the principal's details. Linking identities to entities and permissions to identities builds the chain of trust.

## 3. IAM process: working principle, step by step
Parties: Claimant (user) → Requesting system → Authenticating system.
1. Claimant claims an identity (username/ID).
2. Credential is presented over TLS.
3. Requesting system asks the authenticating system to verify.
4. Credential checked (hash + salt comparison; MFA if required).
5. Grant or deny; event logged; entitlements then apply.

**Identity** = who/what you are. **Credential** = proof (know / have / are).

### Credential management
Authentication · Storage (hash + salt, key protection) · Life cycle (create, issue, expire, revoke, retire) · Secure transmission (TLS/SSL) · Monitoring and auditing · Compliance (PCI DSS, GDPR, HIPAA).

## 4. Entitlements and inherited rights
An entitlement is a specific permission on a resource. Basis: role, group membership, attributes (ABAC), access control policies, business need.
**Inheritance:** users inherit rights from roles/groups, so one change propagates to all members. Efficient, consistent, scalable.
Flow: User → Groups → Roles/attributes → Policy check → Effective entitlements.

## 5. Monitoring, reporting and maintenance
### Role changes
1. Identify the change (promotion, transfer, demotion, new duties).
2. Review access; tear down old rights, then provision for the new role.
3. Update RBAC policies and roles.
4. Notify HR, IT, managers and the user.

### New security standards
1. Assess applicability and impact.
2. Update policies and procedures.
3. Implement controls (stronger authentication, stricter access).
4. Train staff.
5. Monitor continuously, audit, report, fix gaps.

## 6. Terminology
| Term | Full form |
|---|---|
| IAM | Identity and Access Management |
| IAAA | Identification, Authentication, Authorization, Accounting |
| RBAC | Role-Based Access Control |
| ABAC | Attribute-Based Access Control |
| MFA | Multifactor Authentication |
| OTP | One-Time Password |
| PIN | Personal Identification Number |
| TLS / SSL | Transport Layer Security / Secure Sockets Layer |
| MAC | Media Access Control |
| VoIP | Voice over Internet Protocol |
| EDR | Endpoint Detection and Response |
| IdP | Identity Provider |
| SSO | Single Sign-On |
| PAM | Privileged Access Management |
| PCI DSS | Payment Card Industry Data Security Standard |
| GDPR | General Data Protection Regulation |
| HIPAA | Health Insurance Portability and Accountability Act |

## 7. Where it is deployed
- **Commercial/banking:** MFA, SSO, PCI DSS compliance, regular access reviews.
- **Defence/government:** smart cards and PKI certificates (e.g. CAC/PIV), clearance and need-to-know attributes.
- **Healthcare:** role/attribute access to patient records (HIPAA), logged emergency access.
- **Cloud/enterprise:** Microsoft Entra ID, Okta, AWS IAM.
- **Telecom/utilities:** privileged access to network and control systems.
- **Education:** federated identity such as eduroam.

## 8. Pros and cons
**Pros:** central consistent control; fast onboarding/offboarding; least privilege limits breach impact; audit trails; scales via groups and roles.
**Cons:** setup and integration cost; role explosion and privilege creep; IAM is a high-value target; MFA friction; needs continuous reviews.
