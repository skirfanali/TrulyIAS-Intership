# OAuth 2.0 & OpenID Connect (OIDC) Comprehensive Technical Notes

A detailed guide covering foundational principles, authorization grant types, protocol mechanics, extensions, token validation, security controls, and practical testing workflows.

---

## 1. OAuth 2.0 Foundations

### 1.1 What OAuth Solves
OAuth 2.0 is an **authorization framework** (RFC 6749) that allows a third-party application (the Client) to obtain limited access to an HTTP resource (the Resource Server) on behalf of a resource owner. 

- **Core Advantage**: It decouples authorization from authentication and eliminates the need for the resource owner to share their raw credentials (e.g., username/password) with third-party applications.
- **Important Distinction**: Standard OAuth 2.0 provides delegated access authorization; it **does not** by itself provide user authentication or identity assertions.

### 1.2 Core Roles

| Role | Description | Example |
| :--- | :--- | :--- |
| **Resource Owner** | The entity capable of granting access to a protected resource (typically an end user). | An employee granting an app access to their calendar. |
| **Client** | The application requesting access to a protected resource on behalf of the resource owner or on its own behalf. | Web application, Mobile app, Single Page Application (SPA), CLI tool, microservice. |
| **Authorization Server (AS)** | The server that authenticates the resource owner, obtains consent, validates grant requests, and issues tokens. | Okta, PingFederate, Auth0, Keycloak, Entra ID. |
| **Resource Server (RS)** | The server hosting the protected resources/APIs capable of accepting and responding to protected resource requests using access tokens. | Customer API, Payment API, User Info API. |

### 1.3 Essential Endpoints

- **Authorization Endpoint**: Used by interactive clients to obtain authorization from the resource owner via user-agent redirection (`GET` request).
- **Token Endpoint**: Used by the client to exchange an authorization grant, assertion, or refresh token for access tokens (`POST` request with `application/x-www-form-urlencoded`).
- **Revocation Endpoint (RFC 7009)**: Used to invalidate an active access token or refresh token.
- **Introspection Endpoint (RFC 7662)**: Allows resource servers or authorized clients to query the state and metadata of an opaque token.
- **Metadata Endpoint (RFC 8414 / OIDC Discovery)**: Exposes the AS configurations, capabilities, supported algorithms, and endpoints (`/.well-known/oauth-authorization-server` or `/.well-known/openid-configuration`).
- **JWKS URI**: Exposes the public keys (JSON Web Key Set) used to verify signed tokens (JWTs) issued by the AS.

### 1.4 Client Types & Authentication
1. **Confidential Clients**: Applications capable of maintaining the confidentiality of their credentials (e.g., server-side web apps, backend daemons running on secure infrastructure).
2. **Public Clients**: Applications incapable of maintaining credential confidentiality (e.g., Single-Page Applications running in browsers, native mobile/desktop apps, CLI tools).

> **Key Rule**: Embedding a static client secret (`client_secret`) inside a public client binary or JavaScript file does **not** make it secure. Public clients must use Proof Key for Code Exchange (PKCE) and must not rely on client secrets.

---

## 2. Decision Matrix & Grant Overview

| Grant / Pattern | Interactive User? | Client Type | Typical Use Case | Security Status / Guidance |
| :--- | :--- | :--- | :--- | :--- |
| **Authorization Code + PKCE** | Yes | Public or Confidential | Web Apps, SPAs, Mobile Apps, Desktop Apps | **Preferred Standard** for all user-delegated flows. PKCE mandatory for public clients, highly recommended for confidential. |
| **Client Credentials** | No | Confidential | Machine-to-machine (M2M), batch processing, microservices | Standard for workload/machine identity. Requires strong client authentication. |
| **Device Authorization** | Yes (2nd Device) | Public / Input-Constrained | Smart TVs, IoT devices, CLI tools | Recommended when the initiating device lacks an adequate browser or input method (RFC 8628). |
| **Refresh Token Grant** | No | Public or Confidential | Renewing expired Access Tokens without re-prompting user | Requires token rotation or sender-constraining for public clients. |
| **Implicit Grant** | Yes | Public (Historical) | Legacy SPAs | **Deprecated (RFC 9700)**. Do not use for new implementations. |
| **Resource Owner Password (ROPC)** | Yes (direct pass) | Public / Confidential | Legacy migrations only | **Prohibited (RFC 9700 - MUST NOT BE USED)**. Defeats identity separation. |
| **JWT Bearer Grant (RFC 7523)** | Varies | Confidential / Trusted | Federated workload identity, cross-domain authorization | Extension grant; widely used in enterprise integrations. |
| **SAML 2.0 Bearer (RFC 7522)** | Varies | Confidential / Trusted | SAML assertion exchange for OAuth tokens | Extension grant; bridges SAML assertions to OAuth access. |
| **Token Exchange (RFC 8693)** | Varies | Trusted Clients | On-behalf-of (OBO), token transformation, audience restriction | Extension grant for middle-tier API delegation. |

---

## 3. Deep Dive: Authorization Code Grant with PKCE

The primary interactive flow for modern web, mobile, and SPA applications. PKCE (Proof Key for Code Exchange - RFC 7636) prevents authorization code injection and interception attacks.

### 3.1 PKCE Execution Sequence
1. **Verifier Generation**: Client generates a cryptographically random string `code_verifier` (min 43 chars, max 128 chars from unreserved URL characters).
2. **Challenge Derivation**: Client computes:
   $$\text{code\_challenge} = \text{BASE64URL}(\text{SHA256}(\text{ASCII}(\text{code\_verifier})))$$
3. **Authorization Request**: Client redirects the browser to the Authorization Endpoint containing `code_challenge` and `code_challenge_method=S256`.
4. **User Auth & Authorization**: Authorization Server authenticates the user and processes consent.
5. **Code Delivery**: AS redirects browser back to the client’s `redirect_uri` with a temporary `code` and `state`.
6. **Token Redemption**: Client makes a back-channel HTTP `POST` request to the Token Endpoint containing the original `code_verifier` along with the `code`.
7. **Verification**: AS hashes the received `code_verifier` using S256 and compares it against the stored `code_challenge`. If they match, the tokens are issued.

### 3.2 Key Request Parameters

#### Authorization Request (`GET` / `POST` to Authorization Endpoint)
- `response_type`: MUST be `code`.
- `client_id`: Registered client identifier.
- `redirect_uri`: Target callback URL (must strictly match registration).
- `scope`: Space-delimited permissions requested.
- `state`: High-entropy opaque string for anti-CSRF correlation.
- `code_challenge`: SHA-256 base64url-encoded string derived from verifier.
- `code_challenge_method`: MUST be `S256` (`plain` is deprecated/prohibited in modern profiles).

#### Token Request (`POST` to Token Endpoint)
```http
POST /oauth2/token HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=CODE_FROM_CALLBACK
&redirect_uri=https%3A%2F%2Fapp.example.com%2Fcallback
&client_id=demo-client
&code_verifier=ORIGINAL_UNHASHED_VERIFIER
```

---

## 4. Workload & Specialized Grants

### 4.1 Client Credentials Grant (RFC 6749 Section 4.4)
Used for machine-to-machine (M2M) communication where no human context is involved.

```http
POST /oauth2/token HTTP/1.1
Host: as.example.com
Authorization: Basic Base64(client_id:client_secret)
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&scope=inventory.read%20inventory.write
```
- **Rules**:
  - Requires strong client authentication (Basic, `private_key_jwt`, or mTLS).
  - Refresh tokens are typically **not** issued; the client simply re-authenticates when access tokens expire.

### 4.2 Device Authorization Grant (RFC 8628)
Designed for browserless or input-constrained devices (Smart TVs, CLI tools, hardware devices).

1. **Device Request**: Device calls `/device_authorization` with `client_id` and requested scopes.
2. **AS Response**: Returns `device_code`, `user_code`, `verification_uri`, `expires_in`, and polling `interval`.
3. **User Prompt**: Device displays `verification_uri` and `user_code` to the user.
4. **Polling**: Device repeatedly polls `/token` with `grant_type=urn:ietf:params:oauth:grant-type:device_code` and `device_code`.
5. **Handling Errors**:
   - `authorization_pending`: Keep polling at specified interval.
   - `slow_down`: Increase polling interval by at least 5 seconds.
   - `expired_token` / `access_denied`: Stop polling immediately.

### 4.3 Refresh Token Grant & Rotation
Refresh tokens allow clients to obtain new access tokens without user interaction.

```http
POST /oauth2/token HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=EXISTING_REFRESH_TOKEN
&scope=api.read
```

#### Security Controls: Refresh Token Rotation
- Every time a refresh token is exchanged, the AS returns a **new access token AND a new refresh token**.
- The previously used refresh token is immediately invalidated.
- **Reuse Detection**: If an invalidated refresh token is presented to the AS, it indicates potential credential theft. The AS MUST immediately revoke the entire token family associated with that grant.

---

## 5. Modern Extension Grants & Advanced Patterns

### 5.1 Extension Grants Matrix

```
                 +--------------------------------------------------------+
                 |                    Token Endpoint                      |
                 +--------------------------------------------------------+
                                             ^
                                             |
     +---------------------------------------+---------------------------------------+
     |                                       |                                       |
+----+-------------------+       +-----------+------------+       +------------------+----+
|  JWT Bearer (RFC 7523) |       | SAML Bearer (RFC 7522) |       | Token Exchange (RFC 8693)|
| Exchanges signed JWT   |       | Exchanges SAML assertion |       | Exchanges Token for      |
| assertion for Token    |       | for OAuth Token          |       | targeted Downstream Token|
+------------------------+       +------------------------+       +-----------------------+
```

### 5.2 OAuth 2.0 Token Exchange (RFC 8693)
Used by middle-tier microservices to exchange an incoming subject token for an audience-restricted token targeted for a downstream API (On-Behalf-Of pattern).

```http
POST /oauth2/token HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&subject_token=INCOMING_USER_ACCESS_TOKEN
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
&requested_token_type=urn:ietf:params:oauth:token-type:access_token
&audience=https%3A%2F%2Fdownstream-service.example.com
```

---

## 6. OAuth 2.0 vs. OpenID Connect (OIDC)

| Dimension | OAuth 2.0 | OpenID Connect (OIDC 1.0) |
| :--- | :--- | :--- |
| **Primary Domain** | Delegated **Authorization** | User **Authentication** & Identity |
| **Artifact Issued** | **Access Token** (Opaque or JWT) | **ID Token** (Always JWT) + Access Token |
| **Intended Consumer** | Resource Server (API) | Relying Party (Client Application) |
| **Core Standards** | RFC 6749, RFC 6750 | OpenID Connect Core 1.0 |
| **Primary Scope** | Domain specific (`api.read`, `orders:write`) | `openid` (Mandatory), `profile`, `email` |
| **UserInfo Endpoint** | Non-standard | Standardized `/userinfo` endpoint |

### 6.1 ID Token vs. Access Token Validation

| Validation Check | ID Token | Access Token |
| :--- | :--- | :--- |
| **Recipient (`aud`)** | Must match the **Client ID** (`client_id`). | Must match the **Resource Server/API Audience**. |
| **Validation Location** | Evaluated locally by the **Client App**. | Evaluated by the **Resource Server / API Gateway**. |
| **Usage** | Read profile claims; state tracking. **Never pass to APIs**. | Included in HTTP `Authorization: Bearer` header for API access. |
| **Required Claims** | `iss`, `sub`, `aud`, `exp`, `iat`, `nonce` (if requested). | `iss`, `aud`, `exp`, `scope` / `scp`, `client_id`, `sub`. |

---

## 7. Security Architecture & Threat Mitigation (RFC 9700)

### 7.1 Threat Vector & Mitigation Table

| Threat Vector | Mechanism | Mandated Mitigations |
| :--- | :--- | :--- |
| **Authorization Code Interception** | Malicious apps registered on local protocol handlers intercept codes. | Use **PKCE with S256 challenge**. Enforce exact matching on `redirect_uri`. |
| **Cross-Site Request Forgery (CSRF)** | Attacker injects their authorization code into a victim's session. | Enforce unique, crypto-random `state` parameter bound to session cookies; or PKCE. |
| **Token Leakage via URLs** | Tokens in URL query parameters leak via `Referer` headers, proxy logs, and browser history. | **Deprecate Implicit Flow**. Never return tokens directly in URL responses. |
| **Bearer Token Replay** | Attacker intercepts a bearer token and replays it from a different machine. | Use **Sender-Constrained Tokens**: **DPoP (RFC 9449)** or **mTLS (RFC 8705)**. |
| **Mix-Up Attacks** | Client tricks user into authenticating against an attacker-controlled AS. | Validate the `iss` parameter returned in authorization responses (RFC 9207). |
| **JWT Signature Abuse (`alg: none`)** | Attacker removes signature and changes algorithm header to `none`. | Strict allow-list of algorithms on RS (`RS256`, `ES256`). Reject `none` explicitly. |
| **Credential Exposure (ROPC)** | Password capture by client applications; inability to enforce MFA. | **Prohibit ROPC**. Force interactive authentication through browser redirects. |

### 7.2 Sender-Constrained Token Mechanisms
1. **mTLS Token Binding (RFC 8705)**:
   - Access tokens are bound to the client’s X.509 certificate presented during Mutual TLS handshake.
   - The access token contains a hash of the certificate in its confirmation (`cnf`) claim.
2. **DPoP - Demonstrating Proof of Possession (RFC 9449)**:
   - Application-level binding.
   - The client generates an ephemeral public/private key pair and sends a signed DPoP Proof JWT in the `DPoP` HTTP request header alongside the token.

---

## 8. Practical Testing & Diagnostic Cookbook

### 8.1 cURL Request Cookbook

#### Authorization Code Token Exchange (Confidential Client)
```bash
curl -X POST "https://as.example.com/oauth2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -u "demo-client:CLIENT_SECRET_HERE" \
  --data-urlencode "grant_type=authorization_code" \
  --data-urlencode "code=AUTH_CODE_FROM_CALLBACK" \
  --data-urlencode "redirect_uri=https://app.example.com/callback" \
  --data-urlencode "code_verifier=CRYPTOGRAPHIC_CODE_VERIFIER"
```

#### Token Introspection (RFC 7662)
```bash
curl -X POST "https://as.example.com/oauth2/introspect" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -u "resource-server-id:resource-server-secret" \
  --data-urlencode "token=ACCESS_TOKEN_TO_INSPECT"
```

#### Token Revocation (RFC 7009)
```bash
curl -X POST "https://as.example.com/oauth2/revoke" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -u "demo-client:CLIENT_SECRET_HERE" \
  --data-urlencode "token=REFRESH_TOKEN_TO_REVOKE" \
  --data-urlencode "token_type_hint=refresh_token"
```

### 8.2 Common Error Diagnostics Matrix

| Error Response | Cause | Resolution Path |
| :--- | :--- | :--- |
| `invalid_client` | Failed client authentication (bad secret, unknown `client_id`, certificate mismatch). | Verify HTTP `Authorization: Basic` header or body parameters against AS registration. |
| `invalid_grant` | Code expired, single-use code re-used, bad PKCE verifier, or redirect URI mismatch. | Trace execution timeline. Ensure code redemption happens < 60 seconds; check PKCE transformation logic. |
| `invalid_scope` | Requested scope is unrecognized or not granted to client profile. | Verify requested scopes against AS allowed client configuration. |
| `redirect_uri_mismatch` | `redirect_uri` in token exchange does not match authorization request URI *character for character*. | Compare schemes, ports, hostnames, trailing slashes, and URL encodings. |
| `401 Unauthorized` (API) | Token signature invalid, expired (`exp`), incorrect audience (`aud`), or unreadable key. | Decode JWT. Check header `kid` matches JWKS; verify system clock skew; check `aud` claim. |
| `403 Forbidden` (API) | Token valid, but lacks mandatory scope (`scp`/`scope`) or RBAC permissions for the endpoint. | Verify scopes in decoded token payload match endpoint policy requirements. |

---

## 9. Comprehensive Self-Assessment & Knowledge Check

1. **Q: Why must a Public Client (SPA/Mobile App) avoid using a client secret?**
   - *Answer*: Public client code executes on end-user hardware. Binaries can be decompiled and JS source files inspected, making stored secrets public knowledge.

2. **Q: How does PKCE prevent authorization code injection?**
   - *Answer*: An attacker who intercepts an authorization code cannot redeem it at the `/token` endpoint because they lack the high-entropy `code_verifier` kept securely in the original client's memory.

3. **Q: What is the structural difference between `state` and `nonce`?**
   - *Answer*: `state` correlates the authorization request/callback to prevent CSRF attacks on the client. `nonce` binds an OIDC ID Token directly to the client's request to prevent replay attacks.

4. **Q: Can an Access Token be an opaque string?**
   - *Answer*: Yes. OAuth 2.0 does not specify a token format. Opaque tokens require the Resource Server to perform active token introspection (RFC 7662) at the AS.

5. **Q: Why does RFC 9700 strictly prohibit the Resource Owner Password Credentials (ROPC) grant?**
   - *Answer*: ROPC requires users to share primary passwords directly with third-party applications, bypasses modern interactive MFA policies, and violates security isolation boundaries.
