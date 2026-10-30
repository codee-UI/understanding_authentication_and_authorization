# Third-Party Connections & Outsourced Identity (Markdown edition of the simulator)

## 1. API: Application Programming Interface
Rules and data formats that let applications talk to each other.

**Flow (e-commerce payment):**
1. Customer clicks Pay; the app builds a JSON request.
2. App sends it over HTTPS with an API key to the API endpoint.
3. API validates key, rate limit and format, then forwards to the provider.
4. Provider charges the card.
5. Provider returns approved/declined.
6. API replies 200 OK + JSON; the app shows "Order confirmed".

**Pros:** reuse, fast integration, standard and documented.
**Cons:** vendor dependency, leaked keys, breaking changes, data leaves your control.
**Used in:** Stripe/PayPal, Google Maps, Open Banking (PSD2), FHIR healthcare APIs, government/defense data-exchange services between agencies and coalition systems.

## 2. Application Extensions (plugins / add-ons)
Components that add features to an app without changing its core code.

**Flow (browser ad blocker):**
1. User installs from the store.
2. Host asks for permissions; user approves; extension loads.
3. User opens a page.
4. Extension hooks into the page via host APIs and modifies it.
5. User sees the customised result; core code is untouched.

**Pros:** flexible customisation, core untouched, rich ecosystem.
**Cons:** over-broad permissions, malicious/abandoned extensions, compatibility breaks, hard to audit.
**Used in:** Chrome/Firefox add-ons, WordPress, Shopify, VS Code, Salesforce AppExchange, Microsoft 365 add-ins; allow-listed extensions on managed government/defense networks.

## 3. Middleware (e.g., ESB)
Intermediary layer giving message queuing, data transformation and protocol translation.

**Flow (order from CRM):**
1. CRM sends an order event (REST/JSON) to the ESB.
2. ESB queues the message.
3. ESB transforms data to the formats targets expect.
4. Delivers to ERP (SOAP/XML), protocol translated.
5. Delivers to the legacy system (flat file/MQ).
6. Acknowledgements flow back.

**Pros:** connects legacy and modern systems, hides complexity, reliable delivery.
**Cons:** single point of failure, costly, sensitive data in transit, lock-in.
**Used in:** banks (core banking to mobile), retail order-to-cash (MuleSoft, IBM MQ, Kafka), HL7 engines in healthcare, legacy command/logistics integration in defense, telecom billing, energy grid control.

## 4. Identity as a Service (IDaaS)
Gartner-style definition: a predominantly cloud-based service (multi-tenant or dedicated/hosted) providing identity management, provisioning, governance and administration.

| Group | Meaning |
|---|---|
| **IGA** | Provisioning identities into cloud systems, setting/changing privileges, account maintenance, password reset |
| **Access** | Real-time user/device authentication, SSO, authorization, federation via SAML or OAuth 2.0 |
| **Intelligence** | Logging, monitoring, reporting, alerting on identity activity (the "accounting" in AAA) |

**Flow:**
1. IGA: admin adds a new hire.
2. IGA: IDaaS provisions accounts in cloud apps (and later resets passwords/de-provisions).
3. Access: user signs in once via SSO (MFA possible).
4. Access: SAML/OAuth 2.0 token lets the app authorise the user.
5. Intelligence: sign-ins and activity are logged.
6. Intelligence: alerts and reports go to the administrator.

**Pros:** quick rollout, SSO, central audit, scalable subscription.
**Cons:** provider breach affects all clients, outage locks everyone out, vague vendor terminology, accountability stays with you.
**Used in:** Okta, Microsoft Entra ID, Ping, Auth0; FedRAMP-authorised agency portals; DoD ICAM/CAC federation; campus and hospital SSO.

**Other outsourceable services:** risk planning and assessment, security assessment and testing, incident response, BC/DR support, identity management, full AAA access control, IT configuration management, audit and compliance reporting. Delivery may be on-premises staff, cloud-hosted remote management, or fly-in rapid response teams.

## 5. Outsourcing models: two questions
1. **Who employs the administrators?** In-house (employees) or third party (SLA + contract).
2. **Where are systems and AAA databases?** On-premises or cloud-hosted (public, private, hybrid).

| | On-premises | Cloud-hosted |
|---|---|---|
| **In-house** | Max control, max cost | Your staff, shared infrastructure |
| **Third party** | Outsider manages hardware at your site | Typical IDaaS: lowest effort, highest vendor reliance |

**Rule:** final responsibility for performance and security stays with the organization's leadership in every case.

## 6. Full forms
API: Application Programming Interface · ESB: Enterprise Service Bus · CRM: Customer Relationship Management · ERP: Enterprise Resource Planning · IDaaS: Identity as a Service · IAM: Identity and Access Management · IGA: Identity Governance and Administration · SSO: Single Sign-On · SAML: Security Assertion Markup Language · OAuth 2.0: Open Authorization 2.0 · AAA: Authentication, Authorization, Accounting · SLA: Service Level Agreement · MFA: Multi-Factor Authentication · JSON: JavaScript Object Notation · XML: Extensible Markup Language · SOAP: Simple Object Access Protocol · REST: Representational State Transfer · BC/DR: Business Continuity / Disaster Recovery · SaaS: Software as a Service · HL7: Health Level Seven · FHIR: Fast Healthcare Interoperability Resources · FedRAMP: Federal Risk and Authorization Management Program · ICAM: Identity, Credential and Access Management
