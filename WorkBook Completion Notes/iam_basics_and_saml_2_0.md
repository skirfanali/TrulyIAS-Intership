# IAM FUNDAMENTALS & SAML 2.0
## A practical learner workbook | Beginner to Advanced

**Purpose:** Build a clear mental model of identity and access management, then understand and inspect a complete SAML 2.0 browser single sign-on transaction.

* **Running Example:** Maya, an employee, signs in to her organisation's Identity Provider (IdP) and accesses a partner application, the Service Provider (SP).
* **Audience:** Learners beginning IAM, application support engineers, junior IAM engineers, and practitioners preparing for SAML troubleshooting.
* **How to use:** Read each concept, trace the examples, complete the exercises, and use the SAML-tracer lab only with a test account and a non-production environment.

---

## Learning Outcomes
1. Explain identity, authentication, authorization, accounting/auditing, and the IAM lifecycle.
2. Distinguish SSO, federation, identity provider, service provider, and relying party.
3. Explain browser sessions, cookies, session IDs, timeouts, and logout.
4. Describe SAML 2.0 assertions, subjects, NameID, statements, conditions, protocols, bindings, profiles, and metadata.
5. Trace SP-initiated and IdP-initiated SAML Web Browser SSO from the first browser request to the application session.
6. Use SAML-tracer to locate, decode, and interpret `AuthnRequest` and `SAMLResponse` messages.
7. Identify the important SAML validation checks and diagnose common integration failures.
8. Use official OASIS and Ping Identity references for further study.

---

## Workbook Map
| Module | Topics |
| :--- | :--- |
| **1. IAM foundations** | Digital identity, authentication, authorization, AAA, lifecycle, least privilege |
| **2. SSO and federation** | SSO, federation, IdP/SP roles, trust, identity mapping |
| **3. Sessions** | Cookies, session identifiers, browser sessions, IdP/SP sessions, logout |
| **4. SAML 2.0 foundations** | Architecture, XML, assertion structure, subject, NameID, statements |
| **5. SAML browser SSO** | SP-initiated and IdP-initiated flows, detailed steps |
| **6. Advanced SAML** | Bindings, profiles, metadata, signatures, encryption, conditions, replay, RelayState, logout |
| **7. SAML-tracer lab** | Install, capture, inspect, decode, export safely |
| **8. Troubleshooting** | Common errors, systematic diagnosis, scenarios |
| **9. Assessment** | Knowledge checks, practical tasks, glossary and official references |

---

## 1. IAM foundations

### 1.1 What is Identity and Access Management?
Identity and Access Management (IAM) is the set of people, processes, policies, and technologies used to establish digital identities and control their access to systems and data.

Think of an organisation's office building. A person has an identity (who they are), presents credentials at the entrance (authentication), and is allowed into particular floors or rooms (authorization). IAM applies the same ideas to digital resources.

| IAM capability | Question it answers | Example |
| :--- | :--- | :--- |
| **Identity management** | Who is this digital identity? | Create an employee identity for Maya. |
| **Authentication** | Can the claimant prove the identity? | Maya signs in with a password and MFA. |
| **Authorization** | What is this identity allowed to do? | Maya can view invoices but cannot approve payments. |
| **Provisioning** | Which accounts and entitlements must exist? | Create accounts in email, CRM, and payroll. |
| **Governance** | Is access appropriate and approved? | Manager reviews Maya's finance access. |
| **Auditing** | What happened, and can we prove it? | Record sign-in, role change, and access decisions. |

### 1.2 Digital identity and identity attributes
A digital identity is a collection of information representing a person, device, service, or other entity in a particular system. Attributes describe that identity; they are not necessarily unique or permanent.

| Attribute | Example | Why it matters |
| :--- | :--- | :--- |
| **Unique subject identifier** | `employee-10482` | Stable correlation within an agreed identity domain. |
| **Username** | `maya.singh` | Often used for login; may be renamed or reused. |
| **Email** | `maya.singh@example.com` | Useful for communication and sometimes account matching; may change. |
| **Department** | Finance | May influence access decisions. |
| **Employment status** | Active | May trigger provisioning or deprovisioning. |
| **Groups/roles** | Finance-Analyst | Can be used as authorization inputs. |

> **Important:** An email address or username should not automatically be treated as a globally unique, immutable identity key. Define a reliable identifier and account-linking policy.

### 1.3 Authentication vs Authorization
Authentication (AuthN) verifies a claimed identity. Authorization (AuthZ) determines what an authenticated identity may access or do. They are related but separate decisions.

| Authentication (AuthN) | Authorization (AuthZ) |
| :--- | :--- |
| Who are you? | What are you allowed to do? |
| Checks credentials or proof | Checks policy, roles, attributes, scopes, and resource context |
| Password, passkey, certificate, MFA | Role, permission, group, scope, ownership, tenant |
| **Result:** identity/authentication context | **Result:** permit, deny, or constrained access |

*Example:* Maya successfully signs in to the corporate IdP using a passkey. This proves an authentication event. The expense application then checks whether Maya has the `Finance-Approver` role and permission to approve this particular expense. That is authorization.

A successful login does not mean the user is authorized for every application, API, record, or action.

### 1.4 Authentication factors and MFA
| Factor category | Meaning | Examples |
| :--- | :--- | :--- |
| **Something you know** | Knowledge | Password, PIN |
| **Something you have** | Possession | Hardware security key, registered authenticator |
| **Something you are** | Inherence/biometric | Fingerprint or face match |

Multi-factor authentication (MFA) uses two or more distinct factor categories. Two passwords are not two factors. A password plus a one-time code from an authenticator may be MFA, although phishing resistance depends on the method. FIDO2/WebAuthn passkeys and security keys can provide phishing-resistant authentication when correctly deployed.

### 1.5 Authorization models
| Model | Decision basis | Example |
| :--- | :--- | :--- |
| **RBAC** | Roles assigned to users | `Finance-Analyst` can read invoices. |
| **ABAC** | Attributes of user, resource, and environment | Allow access if `department=Finance` and `data classification=Internal`. |
| **PBAC** | Policies evaluate contextual rules | Allow approval only below a configured amount and from a managed device. |

In real deployments, these models can be combined. A role may provide a baseline permission while attributes and contextual policies further restrict access.

### 1.6 IAM lifecycle: Joiner, Mover, Leaver
| Lifecycle event | Typical IAM action | Example |
| :--- | :--- | :--- |
| **Joiner** | Create identity, accounts, and baseline access | New employee joins Finance. |
| **Mover** | Modify access based on new job or department | Employee moves from Finance to HR; remove old access and grant new access. |
| **Leaver** | Disable access, revoke sessions/tokens and remove or retain accounts according to policy | Employee leaves; disable sign-in and initiate deprovisioning. |

Provisioning creates or changes accounts and entitlements. Deprovisioning removes or disables access. Timely leaver processing is critical because a disabled HR record does not automatically terminate every application session or API token unless connected controls perform those actions.

### 1.7 Least privilege and separation of duties
Least privilege means giving an identity only the access needed to perform its assigned tasks, for only as long as needed. Separation of duties prevents one person from controlling an entire sensitive process—for example, the same person should not both create a supplier and approve payment to that supplier.

#### Module 1 Practice
1. An employee enters a correct password but receives HTTP 403 from an application. Is this necessarily an authentication failure? Explain.
2. Give one example each of RBAC and ABAC in an expense application.
3. Describe the IAM actions for an employee who changes from Sales to Finance.
4. Why is a username a weaker long-term correlation key than a stable immutable identifier?

---

## 2. SSO and identity federation

### 2.1 What is Single Sign-On?
Single Sign-On (SSO) allows a user to authenticate once with a trusted identity system and then access multiple connected applications without entering credentials separately for each application, subject to session lifetime, policy, and risk checks.

*Example:* Maya signs in to the company portal. She selects CRM, HR, and a ticketing application. Each application trusts the configured IdP and accepts an appropriate authentication result. The applications may each create their own local session.

> **SSO does not mean one universal session cookie.** The IdP and each SP commonly maintain separate sessions. SSO reduces repeated credential prompts because applications rely on the IdP's authentication assertion or token.

### 2.2 What is identity federation?
Identity federation is an agreed trust relationship between separate security domains that allows identity information and authentication claims to be accepted across organisational or administrative boundaries.

*Example:* Company A employees use Company A's IdP to access a SaaS application operated by Company B. Company B does not need to collect or validate the employee's Company A password. Instead, the SaaS SP trusts assertions from the configured Company A IdP.

| SSO | Federation |
| :--- | :--- |
| **User experience/capability:** fewer repeated logins across applications. | **Trust arrangement:** separate domains agree how identities and authentication information are exchanged. |
| Can exist within one organisation and one administrative domain. | Often crosses organisational or security-domain boundaries. |
| May use cookies, Kerberos, SAML, OIDC, or other mechanisms. | Can be implemented using SAML, OIDC federation, or other trust protocols. |

### 2.3 IdP and SP roles
| Role | Responsibility | Typical example |
| :--- | :--- | :--- |
| **Identity Provider (IdP)** | Authenticates the user and issues identity/authentication information. | Corporate PingFederate or Entra ID acting as SAML IdP. |
| **Service Provider (SP)** | Consumes and validates the assertion, maps the identity, and creates an application session. | SaaS application or PingFederate acting as SAML SP. |
| **User Agent** | Usually the user's browser, carrying redirects and form posts. | Chrome or Edge. |

In SAML, an SP is also commonly described as a relying party because it relies on statements from an asserting party. The IdP is the asserting party in the common Web Browser SSO scenario.

### 2.4 Trust establishment
A SAML federation is not created merely because two endpoints can communicate. The parties must configure and agree on identifiers, endpoints, bindings, certificates/keys, supported profiles, NameID formats, attribute release, and validation expectations.
* The SP needs to know which IdP issuer it trusts and how to validate signatures from that IdP.
* The IdP needs to know the SP's entity ID and where to send the response, usually its Assertion Consumer Service (ACS) endpoint.
* Both parties must protect private signing keys and keep metadata/configuration accurate.
* Trust should be scoped to the intended partner and purpose; accepting any signed assertion from any issuer is unsafe.

### 2.5 SSO versus account provisioning
Federated SSO establishes an application session based on a trusted authentication exchange. It does not necessarily create the user's application account. Just-in-time (JIT) account creation may create or update an account during login, while SCIM or another provisioning mechanism manages accounts before the user signs in.

| Function | Typical protocol/approach |
| :--- | :--- |
| **Authentication and SSO** | SAML 2.0 or OpenID Connect |
| **Provisioning and lifecycle** | SCIM 2.0, APIs, directory synchronization |
| **Directory lookup** | LDAP / Active Directory |
| **Authorization** | Application policy, roles, groups, scopes, attributes |

---

## 3. Sessions, cookies, and logout

### 3.1 What is a session?
A session is server-side state associated with a user or client after an application establishes an authenticated context. Rather than asking the user to prove their identity on every request, the browser sends a session cookie or another session credential that allows the server to look up the session.

Typical sequence: user authenticates $\rightarrow$ server creates a session $\rightarrow$ server sends a session identifier in a cookie $\rightarrow$ browser sends that cookie on later requests $\rightarrow$ server checks the session and applies authorization.

### 3.2 Cookies and session identifiers
| Term | Explanation |
| :--- | :--- |
| **Cookie** | A browser-managed name/value pair sent to matching sites according to cookie scope and attributes. |
| **Session ID** | An opaque, unpredictable value used by the server to identify a session. It should not contain a password or sensitive profile data. |
| **HttpOnly** | Prevents JavaScript from reading the cookie; it does not prevent malicious scripts from making authenticated requests in the browser. |
| **Secure** | Restricts cookie transmission to secure HTTPS connections. |
| **SameSite** | Controls when cookies are sent in cross-site contexts. SSO redirects and cross-site POST flows must be tested against the chosen setting. |
| **Domain/Path** | Scope which hosts and paths receive the cookie. |

> **Security note:** Use HTTPS, strong unpredictable session identifiers, appropriate cookie attributes, session rotation after authentication/privilege changes, server-side invalidation, and sensible idle/absolute timeouts.

### 3.3 IdP session vs SP session
The IdP session represents the user's authenticated session at the identity provider. The SP session represents the user's local authenticated session at the application. The SP can accept an assertion and create its own session; it does not normally reuse the IdP's browser cookie.

*Example:* Maya signs in to the IdP, creating an IdP session. She opens the CRM SP; the SP receives and validates a SAML assertion, then creates a CRM session. She opens a second SP and may be silently redirected through the IdP because the IdP session already exists. The second SP creates its own session.

### 3.4 Session lifetime and reauthentication
* **Idle timeout:** expires a session after a period without activity.
* **Absolute timeout:** expires a session after a maximum total lifetime, even if active.
* **Reauthentication:** requires the user to authenticate again, potentially due to policy, risk, or sensitive action.
* **Session revocation:** invalidates a session before its normal expiry.
* **Remember-me/persistent login:** may use longer-lived credentials and needs additional risk controls.

### 3.5 Logout is not always global logout
Logging out of one SP may end only that SP's local session. SAML Single Logout (SLO) defines a protocol to coordinate logout across participating SAML entities, but support, configuration, and reliability vary. A user may still have sessions at applications that do not participate or cannot be reached.

A robust logout design considers local session invalidation, IdP session state, SP sessions, outstanding tokens, back-channel/front-channel behavior, timeouts, and failure handling.

---

## 4. SAML 2.0 foundations

### 4.1 What is SAML?
Security Assertion Markup Language (SAML) 2.0 is an OASIS standard for expressing and exchanging security information—especially authentication and attribute information—between security domains. SAML messages and assertions are XML-based.

The most common enterprise use case is browser-based SSO: an IdP authenticates a user and sends a SAML Response containing an assertion to an SP. The SP validates it and establishes a local session.

### 4.2 SAML building blocks
| Building block | What it defines | Simple analogy |
| :--- | :--- | :--- |
| **Assertions** | Statements about a subject, such as authentication and attributes. | The signed/validated evidence being presented. |
| **Protocols** | Request/response messages such as `AuthnRequest` and `LogoutRequest`. | What message is being asked or sent. |
| **Bindings** | How protocol messages are transported, e.g. HTTP Redirect or POST. | How the message travels. |
| **Profiles** | Rules combining assertions, protocols, and bindings for a use case. | The complete recipe for browser SSO. |
| **Metadata** | Entity IDs, endpoints, bindings, roles, and key information. | Federation partner configuration card. |
| **Authentication Context** | Information about the method/strength of authentication. | How the user authenticated. |

### 4.3 SAML entity, issuer, and entityID
A SAML entity is a system participating in SAML. Its `entityID` is a unique identifier for that entity within the federation context. It is commonly represented in metadata as the `EntityDescriptor entityID` attribute. Do not confuse `entityID` with a hostname, login URL, ACS URL, or user identifier.

The `Issuer` element in a SAML message or assertion identifies the SAML entity that issued that message/assertion. An SP validates that the issuer is the expected trusted IdP.

### 4.4 SAML assertion anatomy
An Assertion is a structured XML element. It typically contains an ID, IssueInstant, Version, Issuer, Subject, Conditions, and one or more statements.

```xml
<saml:Assertion
    ID="_a12345"
    Version="2.0"
    IssueInstant="2026-11-06T04:00:00Z">
 
  <saml:Issuer>https://idp.example.com</saml:Issuer>
 
  <saml:Subject>
    <saml:NameID Format="...:emailAddress">
      maya.singh@example.com
    </saml:NameID>
    <saml:SubjectConfirmation Method="...:bearer">
      <saml:SubjectConfirmationData
          Recipient="https://sp.example.com/acs"
          NotOnOrAfter="2026-11-06T04:05:00Z"
          InResponseTo="_request123"/>
    </saml:SubjectConfirmation>
  </saml:Subject>
 
  <saml:Conditions NotBefore="..." NotOnOrAfter="...">
    <saml:AudienceRestriction>
      <saml:Audience>https://sp.example.com/entity</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
 
  <saml:AuthnStatement AuthnInstant="...">
    <saml:AuthnContext>...</saml:AuthnContext>
  </saml:AuthnStatement>
 
  <saml:AttributeStatement>
    <!-- agreed user attributes -->
  </saml:AttributeStatement>
</saml:Assertion>
```

> *Note: This is an educational structural example, not a complete production assertion. Ellipses and omitted XML namespaces/signatures mean it must not be used as a deployable message.*

### 4.5 Subject and NameID
The Subject identifies the principal about whom the assertion makes statements. In a browser SSO assertion, the subject is commonly represented using a `NameID` and a `SubjectConfirmation` element.

`NameID` is a SAML name identifier. Its value and format are agreed by the federation partners. Common formats include:

| NameID format | Meaning / use |
| :--- | :--- |
| `emailAddress` | An email-shaped identifier; it can change and should not automatically be assumed immutable. |
| `persistent` | A stable pseudonymous identifier for the federation relationship, subject to the parties' policy. |
| `transient` | A temporary identifier, typically limited to a short-lived SSO context. |
| `unspecified` | The identifier format is not further specified by this format URI. |

A `NameID` is not necessarily the user's login username. An IdP may authenticate Maya using a username but issue a persistent pseudonymous `NameID` to the SP. The SP can map that `NameID` to its local account.

> **Subject is not the same as SubjectConfirmation.** Subject says who the assertion is about. SubjectConfirmation describes how the relying party can accept/confirm the subject in the context of the protocol; for browser SSO, bearer confirmation is common and includes important recipient, expiry, and correlation data.

### 4.6 Three SAML statement types
| Statement | Purpose | Example |
| :--- | :--- | :--- |
| **AuthnStatement** | States that the subject was authenticated and records authentication details/time/context. | Maya authenticated at 09:15 using a specified authentication context. |
| **AttributeStatement** | Carries attributes about the subject. | email, givenName, department, groups. |
| **AuthzDecisionStatement** | Expresses an authorization decision in the SAML assertion model. | Permit/deny a specified action on a resource. |

Modern application authorization is often performed by the SP using its own authorization policy and the attributes/roles it receives, rather than relying on `AuthzDecisionStatement` in typical browser SSO integrations.

### 4.7 Conditions and validity
Conditions place constraints on the use of an assertion. Common elements include:
* `NotBefore`: the assertion is not valid before this time.
* `NotOnOrAfter`: the assertion is not valid at or after this time.
* `AudienceRestriction`: limits the assertion to the intended audience, commonly the SP `entityID`.
* `OneTimeUse`: indicates that the assertion is intended for one-time use.
* `ProxyRestriction`: constrains proxying of assertions.

The SP must evaluate time conditions with a controlled clock and an explicitly configured, limited clock-skew allowance. A broad time tolerance weakens protection.

### 4.8 AuthnRequest and Response
`AuthnRequest` is the SAML protocol message an SP sends to request authentication. The IdP returns a Response, which can include one or more assertions. The Response and Assertion are separate XML structures and can have separate signatures.

| Element | Purpose |
| :--- | :--- |
| **AuthnRequest** | SP asks the IdP to authenticate the user and issue an appropriate response. |
| **Response** | IdP protocol response; includes status and may contain assertions. |
| **StatusCode** | Indicates success or failure of the protocol request. |
| **Assertion** | Carries security statements about the subject. |
| **InResponseTo** | Correlates a response/subject confirmation to the request ID in SP-initiated flows. |

---

## 5. SAML Web Browser SSO: complete flows

### 5.1 The participants and endpoints
| Term | Meaning |
| :--- | :--- |
| **SP Entity ID** | Identifier for the Service Provider; used as an audience and partner identifier where configured. |
| **IdP Entity ID** | Identifier for the Identity Provider; appears as the issuer. |
| **SSO Service URL** | IdP endpoint that receives authentication requests. |
| **ACS URL** | SP Assertion Consumer Service endpoint that receives SAML responses/assertions. |
| **SLO URL** | Endpoint used for SAML Single Logout messages, if configured. |
| **Metadata URL/file** | XML describing entity roles, endpoints, bindings, and key descriptors. |

### 5.2 SP-initiated SSO (most common)
In SP-initiated SSO, the user begins at the Service Provider by requesting a protected application resource. The SP creates an authentication request and redirects the browser to the IdP. The IdP authenticates the user and returns a SAML response to the SP's ACS.

1. User requests a protected URL at the SP, for example `https://crm.example.com/reports`.
2. The SP checks for a valid local session. If none exists, it starts SAML SSO.
3. The SP creates an `AuthnRequest` with a unique ID, Issuer (SP `entityID`), destination, issue time, and any requested options.
4. The SP may store request state and the original target URL. It sends the browser to the IdP SSO service, commonly using HTTP Redirect binding; HTTP-POST is also possible.
5. The IdP receives and validates the request, identifies the SP, and applies authentication policy.
6. If the user has no valid IdP session, the IdP prompts for authentication (password, MFA, passkey, etc.). If a valid session exists, it may reuse that session subject to policy.
7. The IdP creates a SAML Response and Assertion. It sets the issuer, subject/NameID, subject confirmation, conditions, audience, authentication statement, and agreed attributes.
8. The IdP signs the Response and/or Assertion according to the federation agreement and profile. It may encrypt the assertion for the SP if configured.
9. The IdP returns the browser to the SP ACS, commonly through an auto-submitted HTML form using HTTP-POST binding.
10. The SP validates the response and assertion: signature, trusted issuer, destination, recipient, audience, time conditions, correlation, subject confirmation, and replay controls.
11. The SP maps the federated subject to a local account, applies local authorization, and creates its own application session.
12. The SP redirects the browser to the originally requested resource.

> **Remember:** The browser transports the SAML message, but the browser is not the trust authority. The SP must validate the received message using trusted configuration and cryptographic checks.

### 5.3 IdP-initiated SSO
In IdP-initiated SSO, the user starts at the IdP—often an application portal or app launcher—and selects an application. The IdP sends an unsolicited SAML Response to the SP; there is no preceding SP `AuthnRequest` in that transaction.

1. User signs in to the IdP and opens the application catalog.
2. User selects the target SP application.
3. The IdP identifies the configured SP connection and creates a SAML Response and Assertion.
4. The IdP sends the response to the SP ACS, commonly using an HTML form with HTTP-POST binding.
5. The SP validates the response/assertion, maps the subject to a local account, and creates an SP session.
6. The SP redirects the user to a default or RelayState-selected destination, subject to its validation policy.

| SP-initiated | IdP-initiated |
| :--- | :--- |
| User starts at the SP/resource. | User starts at the IdP portal. |
| SP sends `AuthnRequest`. | No `AuthnRequest` is sent in the transaction. |
| Response can be correlated using `InResponseTo`. | No request correlation is available for the unsolicited response. |
| SP can preserve the requested target resource. | SP commonly uses a configured/default landing page or carefully validated RelayState. |

> **Security consideration:** IdP-initiated SSO lacks the SP-generated request correlation available in SP-initiated SSO. The SP still must validate issuer, signature, audience, recipient, time conditions, and replay. It should use strong assertion-consumption and account-linking controls.

### 5.4 HTTP Redirect and HTTP POST bindings
| Binding | Typical use | How it transports data |
| :--- | :--- | :--- |
| **HTTP-Redirect** | Often SP `AuthnRequest` to IdP. | SAML protocol message is DEFLATE-compressed, Base64-encoded, and URL-encoded in the redirect query string; binding-specific signature parameters may be used. |
| **HTTP-POST** | Often IdP Response to SP ACS. | Base64-encoded SAML message is placed in an HTML form field, commonly `SAMLResponse`; browser submits form to ACS. |
| **HTTP-Artifact** | Can avoid sending the full message through the browser. | Browser carries a short artifact; SP resolves it through a back-channel exchange with the IdP. |
| **SOAP** | Back-channel protocol exchanges such as artifact resolution or other SAML profiles. | SAML protocol message is carried in SOAP. |

### 5.5 RelayState
RelayState is an opaque state value carried alongside a SAML message by supported bindings. It is often used to preserve application state or the target URL across the redirect to the IdP and back.
* Treat RelayState as untrusted input when it comes back from the browser.
* Do not blindly redirect to arbitrary URLs supplied in RelayState; validate against an allowlist or store server-side state and send an opaque reference.
* Do not place secrets or sensitive personal data in RelayState.
* RelayState is not a replacement for request ID correlation or signature validation.

### 5.6 SAML Single Logout (SLO)
SAML Single Logout defines messages such as `LogoutRequest` and `LogoutResponse` to coordinate logout among participating SAML entities. SLO may be initiated by an IdP or SP. Each participant must map the NameID/session information to local sessions and process the logout request securely.

### 5.7 Artifact binding
With HTTP-Artifact binding, the browser carries a compact artifact rather than the full SAML response. The receiving SP contacts the IdP's Artifact Resolution Service over a back channel, commonly using SOAP, to retrieve the actual message.

---

## 6. Advanced SAML concepts and security

### 6.1 XML signatures: What is signed?
SAML deployments use XML Signature to provide integrity and issuer authentication for messages/assertions. The federation configuration determines whether the Response, Assertion, or both are signed and what algorithms/certificates are trusted.
* A signature is verified using the public key/certificate trusted for the configured partner.
* The corresponding private signing key must remain protected by the issuer.
* A signed assertion is not automatically encrypted; signatures provide integrity/authenticity, not confidentiality.

### 6.2 Signing vs encryption
| Signing | Encryption |
| :--- | :--- |
| Protects integrity and proves the message/assertion was signed by a holder of the trusted private key. | Protects confidentiality so only the intended recipient with the decryption private key can read the encrypted content. |
| Issuer signs using its private signing key; recipient verifies with trusted public certificate. | Issuer encrypts using recipient's public encryption key; recipient decrypts using its private key. |

### 6.3 Metadata and certificates
SAML metadata is XML describing an entity's role and federation configuration. It can include entityID, IdP SSO endpoints, SP ACS endpoints, supported bindings, NameID formats, Single Logout endpoints, and KeyDescriptor elements for signing/encryption keys.

### 6.4 Audience, Destination, Recipient, and InResponseTo
| Field | What it protects/checks |
| :--- | :--- |
| **Audience** | The assertion is intended for the configured SP/entity. |
| **Destination** | The SAML Response is addressed to the intended endpoint, commonly the ACS URL. |
| **Recipient** | `SubjectConfirmationData` is intended for the SP's ACS endpoint. |
| **InResponseTo** | The response/subject confirmation corresponds to the `AuthnRequest` ID in SP-initiated SSO. |
| **NotBefore / NotOnOrAfter** | The assertion is within its permitted validity window. |
| **Issuer** | The message/assertion came from the expected trusted IdP. |
| **Replay cache** | The same response/assertion ID is not accepted more than once. |

### 6.5 Authentication context and ForceAuthn / IsPassive
* `AuthnContext` describes the authentication method or context used by the IdP.
* `ForceAuthn=true` requests that the IdP freshly authenticate the user rather than relying solely on an existing session.
* `IsPassive=true` asks the IdP not to visibly interact with the user.

### 6.6 NameID policy and account linking
`NameIDPolicy` in an `AuthnRequest` can express the requested NameID format and related behavior. The SP must map that identifier to the correct local account using an explicit and safe account-linking policy.

### 6.7 Common advanced errors
| Symptom | Potential causes to investigate |
| :--- | :--- |
| **Invalid signature** | Wrong signing certificate, stale metadata, wrong signed element, canonicalization/algorithm mismatch. |
| **Audience mismatch** | SP entityID mismatch, wrong partner connection, assertion issued for another SP. |
| **Destination/recipient mismatch** | Incorrect ACS URL, proxy/host rewrite, wrong environment or endpoint. |
| **Not yet valid / expired** | Clock skew, time-zone handling, assertion lifetime, delayed browser transaction. |
| **InResponseTo mismatch** | Wrong request ID, multiple tabs/parallel requests, state lost, unsolicited response sent to SP-only flow. |
| **No user found** | NameID/attribute mapping mismatch, account not provisioned, wrong identifier format or tenant mapping. |

---

## 7. Hands-on lab: SAML-tracer

### 7.1 What is SAML-tracer?
SAML-tracer is a browser debugging extension that captures SAML and WS-Federation messages sent through the browser during SSO and Single Logout. It helps learners inspect redirects, form posts, `AuthnRequest`, `SAMLResponse`, and `RelayState`.

> **Safety Warning:** SAML traces can contain personally identifiable information, authentication context, email addresses, group memberships, session-related values, and bearer assertions. Treat exported traces as sensitive credentials/evidence.

---

## 8. SAML troubleshooting methodology
1. **Reproduce consistently:** Identify the user, SP, IdP, browser, time, environment, and exact start URL.
2. **Capture the browser flow:** Use SAML-tracer with a test account.
3. **Identify the last successful step:** SP request, IdP authentication, response generation, ACS receipt, assertion validation, account mapping, or local authorization.
4. **Compare captured message:** Check federation metadata and connection settings.

---

## 9. Integrated practical exercises
* **Exercise 1:** Draw the SP-initiated flow sequence diagram.
* **Exercise 2:** Compare IdP-initiated vs SP-initiated models.
* **Exercise 3:** Inspect a sample assertion structure.
* **Exercise 4:** Diagnose an audience mismatch.
* **Exercise 5:** Validate assertion with no access (authorization issue).
* **Exercise 6:** Plan a certificate rollover procedure.

---

## 10. Knowledge check & glossary
*(Refer to core summary questions inside the manual workbook for self-assessment checks regarding authentication, federation, IdP/SP roles, metadata, and security implications.)*

### Key Terms Glossary
* **ACS:** Assertion Consumer Service; SP endpoint that receives SAML responses.
* **Assertion:** XML structure containing statements about a subject.
* **AuthnRequest:** SAML request asking an IdP to authenticate a user.
* **IdP:** Identity Provider; authenticates and issues identity assertions.
* **SP:** Service Provider; consumes assertions and provides the application/service.
* **SLO:** Single Logout; protocol for coordinating logout across SAML participants.

---

## 11. Official documentation and further reading
* **OASIS SAML 2.0 Technical Overview:** [OASIS Docs](https://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html)
* **OASIS SAML 2.0 Core Specification:** [Core PDF](https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf)
* **OASIS SAML 2.0 Bindings Specification:** [Bindings PDF](https://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf)
* **OASIS SAML 2.0 Profiles Specification:** [Profiles PDF](https://docs.oasis-open.org/security/saml/v2.0/saml-profiles-2.0-os.pdf)
* **OASIS SAML 2.0 Metadata Specification:** [Metadata PDF](https://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf)