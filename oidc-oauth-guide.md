# OAuth 2.0 and OpenID Connect (OIDC): Guide

**OAuth 2.0** answers *"what may this app access?"* (authorization).
**OIDC** is an identity layer on top of OAuth 2.0 that answers *"who is this user?"* (authentication). It uses JSON and JSON Web Tokens (JWT), is developer-friendly, and is widely used for single sign-on (SSO). Mobile network operators (MNOs) have adopted it too; the GSMA (750+ operators, 400+ manufacturers) promotes operator-based identity.

| | OAuth 2.0 | OpenID Connect |
|---|---|---|
| Purpose | Delegated access | Login and identity |
| Key token | Access token | ID token (signed JWT) + access token |
| Trigger | Scopes (e.g. `files.read`) | Scope `openid` (+ `profile`, `email`) |
| Answers | What can the app do? | Who logged in, when, how? |

## Terminology

| Term | Full form | Meaning |
|---|---|---|
| OAuth 2.0 | Open Authorization 2.0 | Delegated access framework |
| OIDC | OpenID Connect | Identity layer on OAuth 2.0 |
| IdP / OP | Identity Provider / OpenID Provider | Authenticates users, issues ID tokens |
| RP | Relying Party | App relying on the IdP (the client) |
| AS | Authorization Server | Issues access tokens after consent |
| RS | Resource Server | API holding protected data |
| JWT | JSON Web Token | header.payload.signature |
| JWS / JWK / JWKS | JSON Web Signature / Key / Key Set | Signature format and IdP public keys |
| JSON | JavaScript Object Notation | Light data format |
| SSO | Single Sign-On | One login, many apps |
| MFA | Multi-Factor Authentication | Two or more proofs |
| PKCE | Proof Key for Code Exchange | Binds the code to the requesting app |
| Claim | | Statement in a token (sub, name, email) |
| sub / iss / aud | Subject / Issuer / Audience | User ID / token issuer / intended app |
| exp / iat | Expiration / Issued At | Validity times |
| Scope | | Permissions requested |
| nonce | Number used once | Stops ID-token replay |
| state | | Stops CSRF on redirect |
| CSRF | Cross-Site Request Forgery | Unwanted browser request |
| SAML | Security Assertion Markup Language | Older XML SSO standard |
| TLS / HTTPS | Transport Layer Security | Encrypts token traffic |
| MNO | Mobile Network Operator | Telecom carrier |
| GSMA | GSM Association | Mobile industry body |
| FAPI | Financial-grade API | Hardened OAuth/OIDC profile |

## Working principle (Authorization Code flow with PKCE)

The app never sees the user's password. It redirects the browser to the IdP, receives a short-lived **authorization code**, then swaps it for tokens over a direct server-to-server call. The IdP signs the ID token so the app can trust it.

**Actors:** User + Browser → Client App (RP) → Identity Provider (AS) → API (RS)

1. **User starts sign-in.** User clicks Login in the app.
2. **Redirect to IdP.** App sends the browser to `/authorize` with `response_type=code`, `client_id`, `redirect_uri`, `scope=openid profile email`, `state`, `nonce`, `code_challenge`. *(OAuth-only: scope is e.g. `files.read`, no `openid`, no `nonce`.)*
3. **User authenticates.** Password, MFA or social login at the IdP. Only the IdP sees credentials.
4. **Consent.** IdP shows what is requested; the user allows or denies and can revoke later.
5. **Code returned.** IdP redirects to `redirect_uri?code=...&state=...`; app checks `state`.
6. **Token request.** App POSTs `code` + `code_verifier` to `/token` (back-channel, TLS).
7. **Tokens issued.** OIDC: ID token + access token (+ refresh token). OAuth-only: access token (+ refresh token), no ID token.
8. **Validate ID token (OIDC).** Verify signature via JWKS, then `iss`, `aud`, `exp`, `nonce`.
9. **Call API.** `Authorization: Bearer <access_token>` to `/userinfo` or a resource API.
10. **API responds.** Claims (OIDC) or the resource (OAuth) are returned if the token and scope are valid.
11. **Session and SSO.** App creates a session; the IdP session lets the user open other apps without re-entering credentials. Refresh tokens obtain new access tokens; consent can be revoked.

**ID token example (payload):** `iss`, `sub`, `aud`, `exp`, `iat`, `nonce`, `name`, `email`, `amr` (how the user authenticated).

## Pros

- Passwords are held by one IdP, not every app.
- SSO and central MFA, policy and logout.
- Standard JSON/JWT with many libraries across web, mobile and API.
- Signed ID tokens can be verified offline.
- Scoped, revocable user consent.
- Interoperable across vendors.

## Cons and risks

- IdP is a single point of failure and a prime target.
- Misconfiguration: loose redirect URIs, missing `state`, `nonce` or PKCE, wrong `aud`/`iss` checks.
- Bearer tokens work for whoever holds them.
- Many flows and endpoints; learning curve.
- Self-contained JWTs are hard to revoke instantly.
- The IdP can see which apps a user signs into; minimise claims.
- Legacy apps may only support SAML or LDAP.

## Applications

Examples of well-known, publicly documented use; deployments vary and change over time.

| Domain | Examples | Why it fits |
|---|---|---|
| Commercial / consumer | Sign in with Google, Microsoft, Apple; GitHub; Okta, Auth0, Amazon Cognito | Reuse an existing account |
| Enterprise SSO | Microsoft Entra ID, Okta, Keycloak | Central MFA and offboarding |
| Banking / fintech | Open Banking with FAPI profiles | Share data without sharing bank passwords |
| Telecom | GSMA Mobile Connect (operator as OIDC IdP) | SIM-backed identity |
| Government | GOV.UK One Login, Login.gov | One trusted login for many agencies |
| Defense / secure DevSecOps | Keycloak-based SSO (e.g. US DoD Platform One); smartcard/CAC-backed IdPs | Central identity and hardware MFA, with accredited hardening |
| Healthcare | SMART on FHIR | Patient-approved, scoped record access |
| Cloud / DevOps | Kubernetes login, CI/CD workload identity federation | Short-lived tokens replace static secrets |
| Education | University portals, learning platforms, eduGAIN (mostly SAML, OIDC growing) | Campus login across services |
