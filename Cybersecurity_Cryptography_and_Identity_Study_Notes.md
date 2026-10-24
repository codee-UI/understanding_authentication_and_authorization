# Cybersecurity Study Notes: Identity, Secure Communication, Encryption, Hashing, and Digital Signatures

> Beginner-friendly notes based on the supplied course readings.
> Examples use a fictional business, **Sam's Scoops**.
>
> **Important accuracy notes:** These notes preserve the topics in the
> readings while clarifying a few common misconceptions. SSL is
> obsolete; modern websites use TLS. OAuth 2.0 is primarily an
> authorization framework, while OpenID Connect (OIDC) adds
> authentication. A digital signature does not, by itself, hide the
> message.

## Contents

1.  [Authentication and
    authorization](#1-authentication-and-authorization)
2.  [Centralized identity systems and
    protocols](#2-centralized-identity-systems-and-protocols)
3.  [Digital certificates, SSL, and
    TLS](#3-digital-certificates-ssl-and-tls)
4.  [HTTPS: enabling secure website
    communication](#4-https-enabling-secure-website-communication)
5.  [Symmetric encryption and AES](#5-symmetric-encryption-and-aes)
6.  [Hashing algorithms](#6-hashing-algorithms)
7.  [Digital signatures: RSA and
    ECDSA](#7-digital-signatures-rsa-and-ecdsa)
8.  [How the technologies work
    together](#8-how-the-technologies-work-together)
9.  [Sam's Scoops worked examples](#9-sams-scoops-worked-examples)
10. [Common mistakes to avoid](#10-common-mistakes-to-avoid)
11. [Quick revision table](#11-quick-revision-table)
12. [Practice questions](#12-practice-questions)
13. [Glossary](#13-glossary)

------------------------------------------------------------------------

## 1. Authentication and authorization

These two terms sound similar, but they answer different questions.

-   **Authentication (AuthN):** "Who are you?" It verifies identity.
-   **Authorization (AuthZ):** "What are you allowed to do?" It checks
    permissions.

### Simple example

A Sam's Scoops employee signs in to the business system.

1.  The system checks the employee's password and a second factor. This
    is **authentication**.
2.  The system checks whether the employee is allowed to issue refunds.
    This is **authorization**.

A person can be successfully authenticated but still not have permission
to access a particular file or perform an administrative task.

### Why centralize identity management?

Without centralization, every application may have its own user
accounts, passwords, and access rules. A centralized system lets
administrators manage identities and policies in one place.

**Benefits** - One place to create, update, disable, and review
accounts. - Easier enforcement of multi-factor authentication (MFA) and
access policies. - Single sign-on (SSO) can reduce repeated logins. -
Access can be revoked more consistently when a worker leaves.

**Risks and safeguards** - If a central identity service is unavailable,
many services may be affected. - If an administrator account is
compromised, the impact can be large. - Use MFA, least privilege, secure
backups, monitoring, and recovery plans.

## 2. Centralized identity systems and protocols

### 2.1 Kerberos and the Key Distribution Center (KDC)

**Kerberos** is a network authentication protocol. Its **Key
Distribution Center (KDC)** is a trusted service that helps clients
prove their identity and obtain tickets to use network services.

A KDC commonly includes: - **Authentication Server (AS):** checks the
user's initial authentication and issues a Ticket-Granting Ticket. -
**Ticket-Granting Server (TGS):** issues tickets for particular
services.

#### Kerberos flow in plain language

1.  **Initial sign-in:** The client asks the AS for a Ticket-Granting
    Ticket (TGT).
2.  **TGT issued:** After authentication, the client receives a TGT and
    information needed to communicate securely with the TGS.
3.  **Request a service ticket:** When the client needs a service---such
    as a file server---it sends the TGT and a request to the TGS.
4.  **Service ticket issued:** The TGS validates the request and issues
    a ticket for the requested service.
5.  **Access the service:** The client presents the service ticket to
    the server. The server validates it and can establish that the
    request comes from an authenticated client.

The ticket and session-key details are protected with cryptographic
keys. Timestamps and authenticators help reduce replay attacks.

**Example:** An employee signs in once to a company computer and then
accesses an authorized file server without entering the password again
for every service. This is a common form of SSO.

### 2.2 Microsoft identity platform and IDaaS

**Identity as a Service (IDaaS)** means identity and access management
delivered as a cloud service. Microsoft's cloud identity offering is one
example.

Common capabilities include:

-   **SSO:** one sign-in can provide access to multiple connected
    applications.
-   **MFA:** uses more than one kind of evidence to verify a user.
-   **Provisioning/deprovisioning:** creates, updates, or disables
    accounts and access.
-   **Identity governance:** reviews who has access, applies policies,
    and supports audits.
-   **Integration:** connects cloud and, in some configurations,
    on-premises applications.
-   **Scalability:** the service can support changing numbers of users
    and applications.

**Example:** When a Sam's Scoops employee leaves, an administrator
disables the employee's central account and reviews connected
application access. This is safer than relying on someone to remember
every separate account.

> **Practical point:** SSO improves convenience, but it does not replace
> MFA, least privilege, or careful account protection.

### 2.3 RADIUS

**RADIUS** stands for *Remote Authentication Dial-In User Service*. It
is a client-server protocol commonly used to centralize authentication,
authorization-related policy, and accounting for network access.

A network device---such as a wireless access point or VPN gateway---can
send a user's access request to a RADIUS server.

**Example:** An employee connects to company Wi-Fi using their
individual credentials. The access point consults the central
authentication service rather than keeping a separate password list for
every employee.

The "accounting" part can record session information, such as when a
user connected or disconnected.

### 2.4 SAML, OAuth 2.0, and OpenID Connect

These are related but **not interchangeable**.

  -------------------------------------------------------------------------
  Technology              Main purpose              Easy example
  ----------------------- ------------------------- -----------------------
  **SAML**                Exchanges security        An employee signs in to
                          assertions, often for     a company identity
                          enterprise SSO            provider and opens a
                                                    connected web
                                                    application

  **OAuth 2.0**           Delegated authorization:  A calendar app receives
                          allows an application to  permission to read
                          access specific resources selected calendar data
                          with permission           without receiving the
                                                    user's password

  **OpenID Connect        Adds an                   A website lets a user
  (OIDC)**                identity/authentication   sign in with an
                          layer on top of OAuth 2.0 identity provider and
                                                    receives identity
                                                    information
  -------------------------------------------------------------------------

#### Remember the distinction

-   **SAML:** commonly used for enterprise web SSO.
-   **OAuth 2.0:** "What data or action may this application access?"
-   **OIDC:** "Who signed in?"

OAuth 2.0 alone should not be treated as a login/authentication
protocol. In modern systems, applications should use established
authorization-code flows and appropriate security protections rather
than inventing their own token handling.

## 3. Digital certificates, SSL, and TLS

### 3.1 What is a digital certificate?

A **digital certificate** is an electronic document that binds an
identity---such as a website domain---to a public key. A trusted
**Certificate Authority (CA)** signs the certificate so browsers can
check whether they trust that binding.

A certificate commonly contains: - the domain name or subject
identity, - the public key, - the issuer (CA), - validity dates, -
permitted uses and other certificate information, - the CA's digital
signature.

A certificate is **not the same thing as the private key**. The private
key must remain secret on the server or in an appropriately protected
key store.

### 3.2 SSL versus TLS

-   **SSL (Secure Sockets Layer)** is obsolete and should not be
    enabled.
-   **TLS (Transport Layer Security)** is the modern protocol used to
    protect connections.
-   People sometimes say "SSL certificate," but they usually mean a
    certificate used with HTTPS/TLS.

TLS can provide: - **Confidentiality:** outsiders cannot easily read
captured traffic. - **Integrity:** unauthorized changes to protected
traffic are detected. - **Authentication:** the browser can verify the
server's identity using its certificate. Client certificates can also be
used when mutual authentication is required.

### 3.3 Certificate validation levels

  -----------------------------------------------------------------------
  Type                    What is checked         What it does not prove
  ----------------------- ----------------------- -----------------------
  **Domain Validation     Control of the domain   It does not, by itself,
  (DV)**                                          establish that the site
                                                  operator is a
                                                  trustworthy business

  **Organization          Domain control plus     It is not a guarantee
  Validation (OV)**       specified organization  that the organization
                          checks                  behaves honestly

  **Extended Validation   More extensive checks   It does not mean the
  (EV)**                  of the organization's   website is immune to
                          identity                hacking or fraud
  -----------------------------------------------------------------------

A padlock/HTTPS indicator means the connection is encrypted and the
certificate checks passed under the browser's rules; **it does not prove
that the website's content or business is honest**. Modern browsers do
not generally show the old "green address bar" indicator described in
some older material.

### 3.4 What happens during a TLS handshake?

At a high level:

1.  **Client hello:** The browser proposes supported TLS settings and
    provides handshake information.
2.  **Server hello:** The server selects compatible settings.
3.  **Server authentication:** The server supplies its certificate and
    proves possession of the corresponding private key as required by
    the protocol.
4.  **Key agreement:** The browser and server establish shared session
    keys. Modern TLS commonly uses ephemeral key agreement rather than
    the old simplified "encrypt a pre-master secret with the server's
    public key" explanation.
5.  **Handshake verification:** Both sides verify the handshake messages
    and establish that the connection parameters agree.
6.  **Encrypted traffic:** Application data is sent using the negotiated
    keys and algorithms.

Exact messages differ between TLS versions. TLS 1.3 has a different
handshake from older TLS versions, so a diagram of an older handshake
should not be assumed to describe every current connection.

## 4. HTTPS: enabling secure website communication

**HTTPS = HTTP carried over TLS.** It protects communication between a
browser and a web server.

### 4.1 Typical deployment steps

1.  **Control the domain and choose a trusted CA.** Let's Encrypt is one
    example of a CA; commercial CAs are also available.
2.  **Generate or arrange the private key and Certificate Signing
    Request (CSR).** A CSR contains the public key and requested
    identity information. Never send the private key to the CA.
3.  **Complete domain/organization validation** as required by the
    certificate type.
4.  **Install the issued certificate and certificate chain** on the web
    server or hosting platform.
5.  **Configure HTTPS** and enable supported modern TLS versions,
    normally TLS 1.2 and/or TLS 1.3.
6.  **Redirect HTTP to HTTPS** so visitors use the secure endpoint.
7.  **Test the deployment** and monitor certificate expiry,
    configuration, and renewal.

HTTPS normally uses TCP port **443**. Plain HTTP commonly uses port
**80**.

### 4.2 Security configuration checklist

-   [ ] Use TLS 1.2 or TLS 1.3; disable SSL and obsolete TLS versions.
-   [ ] Use modern, secure cipher suites and key-exchange settings.
-   [ ] Keep the private key secret and restrict access to it.
-   [ ] Redirect HTTP to HTTPS.
-   [ ] Set cookies containing session identifiers with the `Secure`
    attribute; also consider `HttpOnly` and an appropriate `SameSite`
    setting.
-   [ ] Consider **HTTP Strict Transport Security (HSTS)** after HTTPS
    is working correctly. HSTS tells browsers to use HTTPS for future
    connections.
-   [ ] Renew certificates before they expire; automate renewal where
    practical.
-   [ ] Test the configuration using a reputable TLS testing tool and
    review the results.
-   [ ] Keep the web server and its software updated.

**Important:** Do not focus only on "encryption length." Security
depends on the whole configuration: protocol version, cipher suite, key
management, certificate validation, software updates, and application
security.

## 5. Symmetric encryption and AES

### 5.1 What is encryption?

Encryption transforms readable **plaintext** into unreadable-looking
**ciphertext** using a cryptographic algorithm and key. Decryption uses
the appropriate key to recover the plaintext.

**Symmetric encryption** uses the same secret key (or the same shared
secret key material) to encrypt and decrypt.

``` text
Plaintext + secret key
        |
        v
   Encryption
        |
        v
    Ciphertext
        |
        | same shared secret key
        v
    Decryption
        |
        v
    Plaintext
```

The key must be shared and stored securely. If an attacker obtains it,
the protected data may be exposed.

### 5.2 From DES to AES

-   **DES (Data Encryption Standard):** an older standard whose key size
    is no longer adequate for modern security.
-   **AES (Advanced Encryption Standard):** a widely used symmetric
    block cipher standardized by NIST in 2001, based on the Rijndael
    design by Joan Daemen and Vincent Rijmen.

AES always uses a **128-bit block size**. Its supported key sizes are:

  AES variant     Key size
  ------------- ----------
  AES-128         128 bits
  AES-192         192 bits
  AES-256         256 bits

A longer key offers a larger brute-force search space, but secure
implementation and key management remain essential.

### 5.3 How AES works conceptually

AES transforms data through repeated rounds. Its main operations
include:

-   **SubBytes:** substitutes bytes using a nonlinear substitution
    table.
-   **ShiftRows:** shifts rows of the internal state.
-   **MixColumns:** mixes values within columns (not used in the final
    round).
-   **AddRoundKey:** combines the state with a round key.
-   **Key expansion/key schedule:** derives round keys from the original
    key.

You do not need to memorize every mathematical operation at first. The
key idea is that multiple rounds transform the data in a carefully
designed way.

### 5.4 AES modes of operation

AES is a block cipher; a **mode of operation** defines how it is applied
to data longer than one block.

  -----------------------------------------------------------------------
  Mode                    Basic idea              Practical guidance
  ----------------------- ----------------------- -----------------------
  **ECB**                 Encrypts each block     Usually avoid for
                          independently           general data because
                                                  repeated plaintext
                                                  blocks can reveal
                                                  patterns

  **CBC**                 Chains blocks together  Requires careful IV
                          and uses an             handling and separate
                          initialization vector   integrity protection;
                          (IV)                    padding and
                                                  implementation errors
                                                  can be dangerous

  **CTR**                 Uses a counter to       Never reuse the same
                          generate a stream-like  key/nonce combination
                          keystream               

  **GCM**                 Provides encryption     A strong common choice
                          plus authentication     when implemented
                          (integrity)             correctly; never reuse
                                                  a nonce with the same
                                                  key
  -----------------------------------------------------------------------

An **IV/nonce** is an input used by a mode. Its requirements depend on
the mode: it may need to be unpredictable or unique. It usually does not
need to be secret, but it must be generated and handled correctly.

For most new application designs, use a well-reviewed
authenticated-encryption library/API (for example, AES-GCM) rather than
designing a mode yourself.

### 5.5 Attacks and safe use

-   **Brute-force attack:** tries many possible keys. Strong keys make
    this impractical.
-   **Side-channel attack:** uses information leaked by an
    implementation---such as timing, power use, or electromagnetic
    emissions---to infer secrets.
-   **Chosen-plaintext attack:** an attacker can choose inputs and
    observe their encryption. This is a standard threat model; it does
    **not** automatically mean AES itself is broken. Secure modes and
    correct implementations are designed to withstand relevant attacks.
-   **Tampering:** encryption alone does not always detect changes. Use
    authenticated encryption or a properly designed integrity mechanism.

Do not rely on "whitening" as a general fix for chosen-plaintext
attacks. Use standard, reviewed cryptographic constructions and
libraries.

### 5.6 Where AES is used

AES is used in many systems, including disk encryption, VPNs, Wi-Fi
security, and secure web connections. In HTTPS, the TLS handshake
establishes keys and the negotiated record-protection algorithm encrypts
and authenticates traffic.

## 6. Hashing algorithms

### 6.1 What is a cryptographic hash?

A cryptographic hash function takes input of almost any length and
produces a fixed-length output called a **hash** or **digest**.

``` text
"Sam's Scoops invoice #104"
             |
             v
     Cryptographic hash
             |
             v
     Fixed-length digest
```

A secure hash function is designed so that it is difficult to: - recover
the original input from its hash (**preimage resistance**), - find
another input with the same hash (**second-preimage resistance**), -
find any two different inputs with the same hash (**collision
resistance**).

Hashing is **not encryption**: there is no decryption key that reverses
a normal cryptographic hash.

### 6.2 Common hash families

  ------------------------------------------------------------------------
  Algorithm                   Output size (examples) Current guidance
  --------------------- ---------------------------- ---------------------
  **MD5**                                   128 bits Broken for collision
                                                     resistance; do not
                                                     use for
                                                     security-sensitive
                                                     integrity or
                                                     signatures

  **SHA-1**                                 160 bits Collision-broken; do
                                                     not use for new
                                                     security-sensitive
                                                     designs

  **SHA-2**              224, 256, 384, or 512 bits, Widely used; SHA-256
                                depending on variant is a common choice

  **SHA-3**              224, 256, 384, or 512 bits, Modern standardized
                                depending on variant alternative with a
                                                     different internal
                                                     construction

  **RIPEMD**               Several variants, such as Legacy/specialized
                                          RIPEMD-160 use; choose a
                                                     currently approved
                                                     hash for new designs

  **BLAKE2**              Common variants provide up Fast modern
                                  to 256 or 512 bits cryptographic hash
                                                     family

  **Tiger**                                 192 bits Older/specialized;
                                                     not a default
                                                     recommendation for
                                                     new designs
  ------------------------------------------------------------------------

**Correction to a common misconception:** SHA-2 and SHA-3 are not
automatically appropriate for password storage just because they are
secure hash functions. Password storage needs a **salt** and a
deliberately slow password-hashing function such as Argon2id, scrypt, or
bcrypt (or an approved alternative appropriate to the environment).

### 6.3 Hashes and integrity checks

Suppose Sam sends an invoice and publishes its expected SHA-256 digest
through a trusted channel. Alex calculates the digest of the received
file.

-   If the digests differ, the file is not identical to the expected
    file.
-   If they match, this supports the conclusion that the file matches
    the expected content, assuming the expected digest itself is
    trustworthy.

A hash by itself does **not** prove who created the file. An attacker
who can replace both the file and the displayed hash may defeat this
simple check. Use a digital signature or authenticated channel when the
source must also be verified.

### 6.4 Password storage

A website should not store users' passwords in plaintext. A secure
password-storage process uses a password-hashing scheme designed for
passwords, with a unique salt per password and suitable work parameters.

``` text
Password + unique salt
          |
          v
 Password-hashing function
          |
          v
Stored salt + password hash + parameters
```

When the user signs in, the system repeats the process and compares the
result. A salt is not a secret; its purpose includes preventing
identical passwords from having identical stored hashes and frustrating
precomputed attacks.

## 7. Digital signatures: RSA and ECDSA

### 7.1 What does a digital signature provide?

A digital signature helps a receiver verify: - **Integrity:** the signed
content has not changed since it was signed. - **Origin/authenticity:**
the signature verifies under the signer's public key. - **Support for
non-repudiation:** evidence can help establish who signed, subject to
key control, identity verification, and legal/process context.

A digital signature does **not** hide the message. Use encryption as
well if confidentiality is required.

### 7.2 How a digital signature works

A simplified view:

1.  The sender creates a cryptographic hash of the message.
2.  A digital-signature algorithm uses the sender's **private key** to
    produce a signature associated with that message hash.
3.  The sender sends the message and signature.
4.  The receiver uses the corresponding **public key** and signature
    algorithm to verify the signature against the received message.
5.  Verification succeeds only if the signature is valid for that
    message and key.

``` text
SENDER
Message -> Hash + private key -> Digital signature
   |                                  |
   +---------- message + signature ---+
                                      |
                                      v
RECEIVER
Received message + signature + trusted public key
                    |
                    v
             Verify signature
                    |
           Valid / Invalid result
```

The phrase "encrypting the hash with the private key" is a simplified
historical explanation. Real digital signatures use specified signature
schemes (for example, RSA-PSS or ECDSA); they are not simply
general-purpose encryption in reverse.

### 7.3 RSA

**RSA** is a public-key cryptographic algorithm based on properties of
large prime numbers and modular arithmetic. It uses a mathematically
related key pair:

-   **Private key:** kept secret; used to sign.
-   **Public key:** shared; used to verify.

For a digital signature, the sender signs with the private key and the
receiver verifies with the public key. For confidentiality using RSA key
transport in supported legacy designs, the direction of key use is
different: a sender would use the recipient's public key and the
recipient's private key would be needed to decrypt. Modern secure
protocols commonly use hybrid encryption and key agreement rather than
RSA directly encrypting large messages.

**Advantages** - Widely supported and well studied. - Can be used for
digital signatures and, in appropriate schemes, public-key encryption.

**Limitations** - Key sizes and operations can be relatively large and
computationally expensive compared with some alternatives. - Private-key
management is essential. - Security depends on correct padding,
parameters, software, and key protection---not merely on using the name
"RSA."

### 7.4 ECDSA

**ECDSA** stands for *Elliptic Curve Digital Signature Algorithm*. It is
a digital-signature algorithm based on elliptic-curve mathematics.

-   It provides strong security with smaller keys than traditional RSA
    keys at comparable security levels.
-   Smaller keys can reduce storage and communication overhead.
-   It is used in some web, software-signing, and other cryptographic
    systems.

RSA and ECDSA are both signature algorithms, but they use different
mathematics and have different implementation requirements. Use a
trusted cryptographic library; do not write your own signature
algorithm.

### 7.5 Certificates and signatures: how they relate

A certificate can be digitally signed by a CA. The CA's signature lets
software verify that the certificate was issued by that CA and has not
been altered. The certificate then binds a public key to a domain or
other identity, subject to validation and trust rules.

For a signed document, the receiver must also obtain the **correct,
trusted public key** for the claimed signer. A signature that verifies
under an unknown key does not automatically establish the real-world
identity of the signer.

## 8. How the technologies work together

Consider a user opening `https://example.com`, signing in, and uploading
an invoice.

1.  **DNS and network connection** help the browser find and contact the
    server.
2.  **TLS certificate validation** helps the browser check that it is
    communicating with the intended server.
3.  **TLS session keys** protect the connection's traffic against
    eavesdropping and tampering.
4.  **Authentication** verifies the user's identity, perhaps using a
    password and MFA.
5.  **Authorization** determines whether the user may upload or view
    that invoice.
6.  **Hashing** can produce a digest to help detect changes to the
    invoice.
7.  **Digital signatures** can help prove that a particular signing key
    signed the invoice and that the signed content has not changed.
8.  **Symmetric encryption**, such as AES in a secure authenticated
    mode, can protect stored or transmitted data.

No single item replaces all the others. They solve different security
problems.

## 9. Sam's Scoops worked examples

### Example A: Employee Wi-Fi

**Need:** Staff should use individual accounts rather than a shared
Wi-Fi password.

**Possible design:** The wireless infrastructure uses centralized
authentication, such as RADIUS with an appropriate enterprise Wi-Fi
configuration.

**Result:** Administrators can manage access centrally and disable a
former employee's access more effectively. The exact design depends on
the equipment and identity system.

### Example B: Secure online ordering

**Need:** Customers submit orders to Sam's Scoops website.

**Possible design:** Configure HTTPS using a trusted certificate, modern
TLS, HTTP-to-HTTPS redirects, secure session cookies, and regular
certificate renewal.

**Result:** Data in transit is protected and the browser can validate
the website's certificate. HTTPS does not by itself protect a
compromised server or guarantee that the business is legitimate.

### Example C: Protecting business records

**Need:** Protect exported financial records stored on a laptop or
backup drive.

**Possible design:** Use a reputable full-disk or file-encryption tool
with strong key management. For application-level encryption, use a
reviewed authenticated-encryption API such as AES-GCM where suitable.

**Result:** Someone who obtains the encrypted storage without the
required key should not be able to read the protected data. Losing the
key can also make the data unrecoverable, so backups and key recovery
must be planned.

### Example D: Verify an invoice

**Need:** Alex wants evidence that an invoice has not been changed and
was signed by Sam.

**Possible design:** Sam signs the invoice using a trusted
digital-signature tool. Alex verifies the signature and checks that the
public key or certificate belongs to Sam.

**Result:** A valid signature supports integrity and signer
authenticity. It does not encrypt the invoice; use encryption separately
if the invoice must remain confidential.

## 10. Common mistakes to avoid

1.  **Confusing authentication and authorization:** proving who someone
    is does not automatically grant every permission.
2.  **Calling OAuth 2.0 an authentication protocol:** OAuth is for
    delegated authorization; OIDC is the identity layer commonly used
    for sign-in.
3.  **Treating SSL as current:** use modern TLS; SSL is obsolete.
4.  **Assuming HTTPS means a site is honest:** HTTPS protects the
    connection, not the trustworthiness of the site's owner or content.
5.  **Thinking a hash is encryption:** hashes are not designed to be
    decrypted.
6.  **Using MD5 or SHA-1 for security-sensitive signatures:** their
    collision weaknesses make them unsuitable for new designs.
7.  **Using plain SHA-256 to store passwords:** use a password-hashing
    scheme with a salt and suitable work factor.
8.  **Assuming encryption always detects tampering:** use authenticated
    encryption or an appropriate integrity mechanism.
9.  **Reusing AES-GCM nonces with the same key:** nonce reuse can
    seriously undermine security.
10. **Thinking digital signatures provide secrecy:** signatures help
    with integrity and authenticity; encryption provides
    confidentiality.
11. **Calling a certificate a private key:** the certificate contains a
    public key and identity information; the associated private key must
    be protected separately.
12. **Building cryptography yourself:** prefer well-maintained, reviewed
    libraries and standard protocols.

## 11. Quick revision table

  -----------------------------------------------------------------------
  Concept                 Main question it        Key idea
                          answers                 
  ----------------------- ----------------------- -----------------------
  Authentication          Who are you?            Verify identity

  Authorization           What can you do?        Check permissions

  Kerberos                How can a user prove    KDC and tickets
                          identity to network     
                          services?               

  RADIUS                  How can network access  Network device consults
                          use central policy?     an authentication
                                                  server

  SAML                    How can enterprise apps Commonly used for web
                          exchange sign-in        SSO
                          assertions?             

  OAuth 2.0               What can an app access  Delegated authorization
                          with permission?        

  OpenID Connect          Who signed in?          Identity layer over
                                                  OAuth 2.0

  Digital certificate     Which identity is       CA-signed
                          associated with this    identity/public-key
                          public key?             binding

  TLS/HTTPS               How can browser-server  Authenticated,
                          traffic be protected?   encrypted connection

  AES                     How can data be         Symmetric block cipher
                          encrypted efficiently   
                          with a shared secret?   

  Hash                    Does this content match Fixed-length digest;
                          an expected digest?     not reversible
                                                  encryption

  Digital signature       Is the signature valid  Private-key signing,
                          for this content and    public-key verification
                          key?                    
  -----------------------------------------------------------------------

## 12. Practice questions

Try answering before checking the answer key.

1.  An employee enters a password and MFA code. Is this authentication
    or authorization?
2.  The system allows that employee to view invoices but not approve
    refunds. What is being checked?
3.  Which Kerberos component issues a service ticket: AS or TGS?
4.  Which protocol is commonly used for enterprise SSO assertions: SAML,
    OAuth 2.0, or OIDC?
5.  What is the primary purpose of OAuth 2.0?
6.  What is the difference between a certificate and its private key?
7.  What is the modern replacement for SSL?
8.  Does HTTPS guarantee that a website is honest?
9.  Does AES use a public/private key pair?
10. Which AES mode provides encryption and integrity together when
    correctly implemented?
11. Why should MD5 not be used for new digital-signature designs?
12. Why is a normal cryptographic hash not suitable by itself for
    password storage?
13. Does a digital signature encrypt the signed document?
14. Which key does a signer use to create a signature, and which key
    does a receiver use to verify it?
15. Name one control that helps protect a centralized identity system.

### Answer key

1.  Authentication.
2.  Authorization.
3.  TGS.
4.  SAML is commonly used for enterprise SSO assertions.
5.  Delegated authorization: granting an application scoped access to
    resources without sharing the user's password.
6.  A certificate contains identity/public-key information and a CA
    signature; the private key is secret and must be protected
    separately.
7.  TLS.
8.  No. HTTPS protects the connection and verifies the certificate under
    browser trust rules; it does not guarantee honesty.
9.  No. AES is symmetric and uses a shared secret key.
10. AES-GCM.
11. MD5 has known collision weaknesses.
12. Passwords need a dedicated password-hashing scheme, a salt, and
    suitable work parameters.
13. No. A signature supports integrity/authenticity; encryption is
    separate.
14. The signer uses the private key; the receiver verifies with the
    corresponding public key.
15. Examples: MFA, least privilege, protected admin accounts,
    logging/monitoring, secure backups, or a tested recovery plan.

## 13. Glossary

-   **AES:** Advanced Encryption Standard, a symmetric block cipher.
-   **Authentication:** Checking a claimed identity.
-   **Authorization:** Checking permitted actions or resources.
-   **CA (Certificate Authority):** An entity trusted to issue and sign
    certificates.
-   **Certificate:** A signed document binding an identity to a public
    key.
-   **Ciphertext:** Encrypted data.
-   **Cryptographic hash:** A function that produces a fixed-size digest
    from input data.
-   **Digital signature:** A cryptographic value used to verify a
    message or document's integrity and association with a signing key.
-   **ECDSA:** Elliptic Curve Digital Signature Algorithm.
-   **HSTS:** A browser policy that instructs a browser to use HTTPS for
    a site.
-   **IDaaS:** Identity and Access Management delivered as a cloud
    service.
-   **IV/nonce:** An input used by a cryptographic mode; requirements
    vary by mode.
-   **KDC:** Kerberos Key Distribution Center.
-   **MFA:** Multi-factor authentication.
-   **OAuth 2.0:** A framework for delegated authorization.
-   **OIDC:** OpenID Connect, an identity layer on OAuth 2.0.
-   **Plaintext:** Readable/original data before encryption.
-   **Private key:** Secret key used for operations such as signing or
    decryption, depending on the algorithm.
-   **Public key:** Key that can be shared and used for operations such
    as signature verification.
-   **RADIUS:** Protocol commonly used to centralize network-access
    authentication and related policy/accounting.
-   **RSA:** Public-key cryptographic algorithm used for signatures and
    certain encryption schemes.
-   **SAML:** Security Assertion Markup Language, commonly used for
    enterprise SSO.
-   **Salt:** A per-password random value used by password-hashing
    schemes.
-   **TLS:** Transport Layer Security, the modern protocol used by
    HTTPS.
-   **TGT:** Ticket-Granting Ticket in Kerberos.

------------------------------------------------------------------------

## Final takeaway

Think of cybersecurity as several complementary layers:

-   **Identity systems** decide who is signing in and what they can
    access.
-   **Certificates and TLS** help establish trusted, protected
    connections.
-   **AES** encrypts data using a shared secret.
-   **Hashes** help detect whether content differs from an expected
    value.
-   **Digital signatures** help verify content integrity and association
    with a signing key.

Choose standard, maintained implementations, protect keys and accounts,
and keep systems updated.
