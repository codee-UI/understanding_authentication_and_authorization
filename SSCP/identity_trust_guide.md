# Identity & Trust Guide: SAML, OAuth 2.0 and Trust Relationships

Companion to the interactive *Identity & Trust Lab*. Same concepts, in text form.

## Glossary

| Term | Full form | Meaning |
|---|---|---|
| SAML | Security Assertion Markup Language | XML standard for exchanging authentication and authorization data |
| OAuth 2.0 | Open Authorization 2.0 | Delegated authorization framework using access tokens |
| OIDC | OpenID Connect | Identity layer on OAuth 2.0 (ID tokens) |
| SSO | Single Sign-On | Authenticate once, access many services |
| FIM | Federated Identity Management | Sharing identity across trusted organisations |
| IdP | Identity Provider | Authenticates users, issues assertions or tokens |
| SP | Service Provider | Hosts the resource, trusts the IdP |
| ACS | Assertion Consumer Service | SP endpoint that receives assertions |
| XML / XHTML | Extensible Markup Language / Extensible HyperText Markup Language | Message and form formats |
| SOAP | Simple Object Access Protocol | XML messaging protocol (a SAML binding) |
| JWT | JSON Web Token | Signed token format used for ID tokens |
| PKCE | Proof Key for Code Exchange | Protects the authorization code flow |
| TLS / PKI / CA | Transport Layer Security / Public Key Infrastructure / Certificate Authority | Encryption and certificate-based trust |
| MAC | Media Access Control (address) | Device hardware address (also: Mandatory Access Control) |
| NIST | National Institute of Standards and Technology | Defines "trust relationship" and zero trust guidance |
| IAM | Identity and Access Management | Managing identities and permissions |
| RASP | Runtime Application Self-Protection | App defends itself at runtime |
| CI/CD | Continuous Integration / Continuous Deployment | Automated build and release pipeline |

---

## 1. SAML

**Idea:** log in once at the IdP; the IdP sends the SP a signed XML assertion (identity, authentication status, attributes). The SP trusts the IdP and decides access from roles and attributes.

**Components:** Assertions (XML statements), Protocols (request/response messages, HTTP and SOAP), Bindings (how messages travel), Profiles (rule sets for a use case such as Web Browser SSO).

### SP-initiated web SSO (the diagram you attached)
1. Browser requests the target resource from the SP.
2. SP discovers the IdP, then redirects the browser to the SSO service with an AuthnRequest.
3. Browser requests the SSO service at the IdP.
4. IdP identifies (authenticates) the user, then responds with an XHTML form containing the signed assertion.
5. Browser requests the SP's Assertion Consumer Service.
6. SP validates the signature and conditions, then redirects to the target resource.
7. Browser requests the target resource again.
8. SP responds with the requested resource.

**Pros:** true cross-domain SSO; mature and widely supported; credentials never reach the SP; rich attributes for fine-grained access control; central deprovisioning.
**Cons:** verbose XML; poor fit for mobile apps and APIs; signature-validation mistakes are dangerous; metadata and certificate management; single logout is hard.
**Used in:** Microsoft 365, Google Workspace, Salesforce, Workday, ServiceNow, AWS IAM Identity Center (commercial); InCommon, eduGAIN (education and research); inter-agency and coalition-partner federations (government and defence); healthcare and finance partner portals.

---

## 2. OAuth 2.0 and OpenID Connect

**Idea:** OAuth 2.0 delegates *authorization*: a user lets an app access resources without sharing a password. OIDC adds *authentication* with an ID token (JWT). OAuth alone is not SSO; with OIDC it is.

**Roles:** resource owner (user), client (app), authorization server, resource server. **Scopes** define permissions.

### Authorization code + OIDC
1. User clicks "Sign in with...".
2. Client redirects to the authorization server (client_id, scopes, redirect_uri, state).
3. User authenticates and consents.
4. Server returns a short-lived authorization code.
5. Client exchanges the code for tokens (back channel).
6. Server returns an access token and an ID token.
7. Client calls the API with the access token.
8. Resource server validates the token and returns data.

### Service to service (client credentials)
Service A authenticates, gets a scoped token, calls Service B, which validates it. Used in microservices.

**Federation use:** after OIDC authentication, the ID token can be exchanged for an access token to reach resources at another domain, with trust set by federation configuration.

**Pros:** no credential sharing; granular, revocable scopes; web, mobile and microservices; lightweight JSON/JWT.
**Cons:** not authentication by itself; stolen bearer tokens are usable (use short lifetimes, PKCE, TLS); many options invite misconfiguration; consent fatigue.
**Used in:** Google/Facebook/Apple sign-in, GitHub and Spotify APIs (commercial); open banking, PSD2, FAPI (finance); SMART on FHIR (healthcare); defence DevSecOps platforms and internal APIs; service meshes and API gateways.

---

## 3. Trust relationships

NIST: an alliance between system elements based on criteria for secure interaction, behaviour and outcomes for protecting assets.

### Authentication types
- **Reverse:** the respondent (e.g. a bank site) proves itself with a certificate containing its public key. Client validates it, then encrypts a session key.
- **Mutual:** both sides exchange certificates (mutual TLS).
- **Forward (node authentication):** the calling device presents its MAC or IP address. Weak alone because addresses can be spoofed.

### Trust models
- **One-way:** Domain A's users reach Domain B's resources, not the reverse.
- **Two-way:** both directions.
- **Transitive:** if A trusts B and B trusts C, A may trust C; also subdirectories inheriting from a parent. Non-transitive trust stops at the first hop.

### Zero trust
No entity is trusted by default. Principles: verify every request; least privilege; micro-segmentation; continuous monitoring; encryption in transit and at rest.
Applies to: IAM, endpoint security, application security (secure by design, RASP), data protection, cloud, DevOps and CI/CD, supply chain.

**Pros:** shared resources without duplicate accounts; assurance against spoofed sites; limits breach impact and lateral movement.
**Cons:** transitive trust may over-extend access; one compromised domain can expose partners; PKI lifecycle effort; zero trust is complex and costly.
**Used in:** TLS/PKI on every HTTPS site, Active Directory forest trusts, Google BeyondCorp (commercial); smart-card PKI with mutual TLS, NIST SP 800-207, DoD zero trust strategy (defence); mutual TLS between banks and payment APIs (finance); signed software and verified vendors (supply chain).

---

## Quick comparison

| | SAML | OAuth 2.0 | OIDC |
|---|---|---|---|
| Main job | Authentication + SSO | Delegated authorization | Authentication on OAuth |
| Format | XML assertion | Access token | JWT ID token |
| Best for | Enterprise web SSO | APIs, mobile, microservices | Modern SSO |
