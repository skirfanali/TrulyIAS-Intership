# Workbook 3 --- Web Protocols, APIs, Form Mechanics & Sessions for IAM

## Study Notes, Commands and Hands-on Labs (Modules 1--10)

**Level:** Engineering Foundation\
**Basis:** Supplied Workbook 3. These notes retain its module order and
explain the concepts with practical commands. Use only test accounts and
systems you are authorized to access. Never publish real passwords,
tokens, private keys, or session cookies.

------------------------------------------------------------------------

# Module 1 --- Web Application Architecture & the Dual Channel Model

## Core concepts

A typical identity-enabled web application involves: - **User
browser/agent:** The end user's browser. - **Identity Provider (IdP):**
Authenticates the user. - **Service Provider (SP):** The application the
user wants to access. - **Application backend:** Communicates with the
IdP and handles server-side operations.

**Front-channel:** Communication that passes through the user's browser,
commonly redirects and browser requests. It can be inspected in browser
Developer Tools.

**Back-channel:** Direct server-to-server communication, commonly HTTPS
requests. For example, an application backend exchanges an authorization
code for tokens. This exchange normally does not appear in the browser's
Network tab.

### Simplified authorization-code flow

1.  Browser is redirected to the IdP.
2.  User authenticates.
3.  IdP redirects the browser to the application's callback with an
    authorization code.
4.  Application backend exchanges the code with the IdP over a
    back-channel.
5.  Application establishes a session or uses the returned tokens.

### Lab 1A --- Trace front-channel redirects

1.  Open Chrome and press `F12`.
2.  Select **Network** and enable **Preserve log**.
3.  Visit a login page for a test account you own.
4.  Sign in and inspect the sequence of browser requests.
5.  Look for redirects such as `302` and callback requests containing a
    `code` parameter.
6.  Identify which activity is visible in the browser and which token
    exchange is performed server-to-server.

**Useful DevTools shortcuts** - `F12` --- open Developer Tools. -
`Ctrl + R` --- reload. - Network → Preserve log --- keep requests across
redirects. - Network → select request → Headers --- inspect headers.

------------------------------------------------------------------------

# Module 2 --- HTTP Request & Response Anatomy in IAM

## HTTP request

An HTTP request typically contains: - **Method:** GET, POST, PUT, PATCH,
DELETE. - **URL/path:** The resource being requested. - **Headers:**
Metadata such as `Host`, `Accept`, `Content-Type`, `Authorization`. -
**Body:** Optional data, often sent with POST, PUT, or PATCH.

## HTTP response

An HTTP response typically contains: - **Status code:** Result of the
request. - **Headers:** Metadata such as `Content-Type`, `Set-Cookie`,
and `Location`. - **Body:** Returned content, often JSON or an error
message.

## Lab 2A --- cURL header inspection

Run in Windows PowerShell:

``` powershell
curl.exe -Iv "https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"
```

-   `curl.exe` --- runs cURL in Windows PowerShell.
-   `-I` --- requests response headers only.
-   `-v` --- verbose connection and header details.
-   Lines beginning with `>` show client request information.
-   Lines beginning with `<` show server response information.

Retrieve the JSON body:

``` powershell
curl.exe -sS "https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"
```

Save it to a file:

``` powershell
curl.exe -sS "https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration" -o oidc-discovery.json
```

## Important HTTP status codes

  Code   Meaning                 IAM/API interpretation
  ------ ----------------------- ---------------------------------------------
  200    OK                      Request succeeded
  201    Created                 Resource created
  302    Found                   Temporary redirect
  303    See Other               Follow-up request uses GET
  307    Temporary Redirect      Preserves request method
  400    Bad Request             Invalid request
  401    Unauthorized            Missing, invalid, or expired authentication
  403    Forbidden               Caller lacks permission
  404    Not Found               Route/resource not found
  500    Internal Server Error   Server-side failure

**401 vs 403** - **401:** "Who are you?" Authentication failed or was
not supplied. - **403:** "I know who you are, but you are not allowed."
Authentication may have succeeded, but permission is insufficient.

A 403 can also result from security policy or configuration. Check
server logs and response details rather than assuming one cause.

------------------------------------------------------------------------

# Module 3 --- HTTP Redirection Mechanics & State Preservation

A redirect tells a browser/client to make another request to a different
URL. The response commonly contains a `Location` header.

  Code   Name                 Typical method behavior
  ------ -------------------- ---------------------------------------------------
  301    Moved Permanently    Browsers commonly change POST to GET
  302    Found                Historically, clients commonly change POST to GET
  303    See Other            Explicitly follow with GET
  307    Temporary Redirect   Preserve original method and body

### Lab 3A --- Inspect multi-hop redirects

1.  Open Chrome DevTools → Network.
2.  Enable Preserve log.
3.  Visit an authorized login portal.
4.  Inspect requests with status `301`, `302`, `303`, or `307`.
5.  Open a redirect response and inspect its `Location` header.
6.  Follow the request sequence to understand where the browser is sent.

### cURL redirect commands

Show headers without automatically following redirects:

``` powershell
curl.exe -I "https://example.com"
```

Follow redirects and show final response headers:

``` powershell
curl.exe -IL "https://example.com"
```

Show verbose redirect information:

``` powershell
curl.exe -v -L "https://example.com"
```

-   `-L` --- follow redirects.
-   `-I` --- request headers only.
-   `-v` --- verbose output.

------------------------------------------------------------------------

# Module 4 --- Sessions, Cookies & Browser Storage Security

## What is a session?

A session is server-managed state associated with a user's interaction
with an application. The browser often carries a session identifier in a
cookie, and the server uses it to find the session.

## What is a cookie?

A cookie is a small piece of data stored by the browser and sent with
matching requests. Cookies are commonly used for session identifiers and
preferences.

## Important cookie attributes

  -----------------------------------------------------------------------
  Attribute                           Security role
  ----------------------------------- -----------------------------------
  `Secure`                            Cookie is sent only over secure
                                      HTTPS connections

  `HttpOnly`                          JavaScript cannot read it through
                                      `document.cookie`

  `SameSite=Strict`                   Restricts cookie to same-site
                                      contexts

  `SameSite=Lax`                      Allows some top-level cross-site
                                      navigations, commonly GET

  `SameSite=None`                     Allows cross-site cookie use;
                                      requires `Secure` in modern
                                      browsers
  -----------------------------------------------------------------------

Cookie behavior depends on browser rules and request context. SameSite
settings can affect SSO flows, including SAML POST flows.

### Lab 4A --- Inspect cookies

1.  Sign in to a test website you own or are authorized to use.
2.  Open Chrome DevTools (`F12`).
3.  Select **Application** → Storage → Cookies.
4.  Select the site's domain.
5.  Inspect Secure, HttpOnly, and SameSite attributes.
6.  Never copy or share live session-cookie values.

### Lab 4B --- Demonstrate HttpOnly

Open the DevTools Console and run:

``` javascript
console.log(document.cookie);
```

**Observation:** Cookies marked `HttpOnly` are not exposed through
`document.cookie`. This helps prevent JavaScript from directly reading
session cookies if an XSS flaw occurs, although it does not eliminate
every impact of XSS.

Other useful browser commands:

``` javascript
console.log(window.location.href);   // Current page URL
console.log(window.location.origin); // Current page origin
```

------------------------------------------------------------------------

# Module 5 --- RESTful API Architecture & CRUD Mechanics

## What is an API?

An Application Programming Interface allows software systems to
communicate through defined requests and responses.

## What is REST?

REST is an architectural style for designing network APIs around
resources. REST APIs commonly use HTTP methods and JSON representations.

## CRUD mapped to HTTP methods

  CRUD      HTTP method   Example               Purpose
  --------- ------------- --------------------- ----------------------------
  Create    POST          `POST /Users`         Create a user
  Read      GET           `GET /Users/123`      Read a user
  Replace   PUT           `PUT /Users/123`      Replace a resource
  Update    PATCH         `PATCH /Users/123`    Update selected attributes
  Delete    DELETE        `DELETE /Users/123`   Delete/deprovision a user

Example JSON:

``` json
{
  "name": "Rahul Sharma",
  "email": "rahul@example.com",
  "active": true
}
```

## cURL examples

The following use a placeholder URL. Replace it with an API you own or
are authorized to test.

**GET**

``` powershell
curl.exe -i "https://api.example.com/Users/123"
```

**POST**

``` powershell
curl.exe -i -X POST "https://api.example.com/Users" `
  -H "Content-Type: application/json" `
  -d '{"name":"Rahul Sharma","email":"rahul@example.com"}'
```

**PUT**

``` powershell
curl.exe -i -X PUT "https://api.example.com/Users/123" `
  -H "Content-Type: application/json" `
  -d '{"name":"Rahul Sharma","email":"rahul@example.com","active":true}'
```

**PATCH**

``` powershell
curl.exe -i -X PATCH "https://api.example.com/Users/123" `
  -H "Content-Type: application/json" `
  -d '{"active":false}'
```

**DELETE**

``` powershell
curl.exe -i -X DELETE "https://api.example.com/Users/123"
```

`api.example.com` is a placeholder, not a working API for these
operations.

## Connection to SCIM

SCIM (System for Cross-domain Identity Management) standardizes identity
provisioning. Its user operations map naturally to POST, GET, PUT,
PATCH, and DELETE.

------------------------------------------------------------------------

# Module 6 --- Data Formats: JSON vs XML

## JSON

JSON (JavaScript Object Notation) is a lightweight text format for
structured data. It supports objects, arrays, strings, numbers,
booleans, and null.

``` json
{
  "name": "John Doe",
  "email": "john.doe@company.com",
  "active": true,
  "groups": ["Engineering", "Security"]
}
```

## XML

XML (Extensible Markup Language) represents structured data using tags
and attributes.

``` xml
<user>
  <name>John Doe</name>
  <email>john.doe@company.com</email>
  <active>true</active>
</user>
```

## JSON vs XML

  JSON                            XML
  ------------------------------- ------------------------------------
  Lightweight syntax              Tag-based syntax
  Common in REST and OAuth/OIDC   Common in SAML and SOAP
  Uses objects and arrays         Uses elements and attributes
  Easily parsed in JavaScript     Supports namespaces and attributes

### Lab 6A --- Convert XML to JSON in CyberChef

1.  Open https://gchq.github.io/CyberChef/
2.  Paste the XML below into Input.
3.  Search for an XML-to-JSON operation available in the current
    CyberChef version.
4.  Add it to the Recipe and inspect the output.

Input:

``` xml
<saml:Attribute Name="email">
  <saml:AttributeValue>john.doe@company.com</saml:AttributeValue>
</saml:Attribute>
```

Illustrative JSON representation:

``` json
{
  "saml:Attribute": {
    "@Name": "email",
    "saml:AttributeValue": "john.doe@company.com"
  }
}
```

The exact output shape depends on the converter and its options.

### Validate JSON in PowerShell

``` powershell
Get-Content .\sample.json -Raw | ConvertFrom-Json
```

Convert a PowerShell object to JSON:

``` powershell
$user = @{
  name = "John Doe"
  email = "john.doe@company.com"
  active = $true
}

$user | ConvertTo-Json -Depth 5
```

------------------------------------------------------------------------

# Module 7 --- SOAP vs REST & Legacy Enterprise Identity

## What is SOAP?

SOAP (Simple Object Access Protocol) is a messaging protocol that
commonly uses XML envelopes to structure requests and responses.

## What is WS-Security?

WS-Security is a set of SOAP message security extensions. A SOAP message
can include security information in its header, such as a UsernameToken.

### Example SOAP envelope

Use fictional test values only:

``` xml
<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Header>
    <sec:Security>
      <sec:UsernameToken>
        <sec:Username>iam_service</sec:Username>
        <sec:Password>REPLACE_WITH_TEST_SECRET</sec:Password>
      </sec:UsernameToken>
    </sec:Security>
  </soapenv:Header>
  <soapenv:Body>
    <GetUserBalance account="TEST_ACCOUNT"/>
  </soapenv:Body>
</soapenv:Envelope>
```

## REST bearer-token example

A REST API commonly carries an access token in the HTTP Authorization
header:

``` http
Authorization: Bearer YOUR_ACCESS_TOKEN
```

## SOAP vs REST

  -----------------------------------------------------------------------
  SOAP                                REST
  ----------------------------------- -----------------------------------
  Protocol                            Architectural style

  Commonly uses XML envelopes         Commonly uses JSON, but can support
                                      other formats

  Defines structured message          Uses resources and HTTP methods
  conventions                         

  WS-Security may be used             HTTPS, OAuth 2.0, JWT and other
                                      mechanisms may be used
  -----------------------------------------------------------------------

**IAM relevance:** Legacy enterprise systems may use SOAP and
WS-Security. Modern web identity integrations commonly use REST APIs and
bearer tokens.

------------------------------------------------------------------------

# Module 8 --- Postman for Identity Engineers

## What is Postman?

Postman sends API requests, displays responses, organizes collections,
and runs automated tests.

## Create a login request

Example configuration: - Method: `POST` - URL:
`{{base_url}}/api/login` - Body: raw → JSON - Header:
`Content-Type: application/json`

Example body:

``` json
{
  "email": "rahul@example.com",
  "password": "YOUR_TEST_PASSWORD"
}
```

The route and field names must match your actual backend.

## Create environment variables

In Postman, create an environment:

  Variable         Example
  ---------------- -------------------------
  `base_url`       `http://localhost:8080`
  `access_token`   Leave empty initially

Use `{{base_url}}` in request URLs.

## Automatically capture a token

In the login request's **Scripts → Post-response** (called the Tests tab
in some older Postman versions), use:

``` javascript
const responseData = pm.response.json();

pm.test("Status code is 200 OK", function () {
    pm.response.to.have.status(200);
});

if (responseData.token) {
    pm.environment.set("access_token", responseData.token);
    console.log("Access token captured.");
}
```

If your backend returns `accessToken` instead of `token`, use:

``` javascript
const responseData = pm.response.json();

if (responseData.accessToken) {
    pm.environment.set("access_token", responseData.accessToken);
    console.log("Access token captured.");
}
```

**Security:** Do not print real tokens to shared logs, screenshots, or
public workspaces. A bearer token acts like a credential.

## Reuse the token

For a protected request: 1. Open Authorization. 2. Select **Bearer
Token**. 3. Enter `{{access_token}}`. 4. Send the request.

Equivalent header:

``` http
Authorization: Bearer {{access_token}}
```

## Suggested Postman collection flow

1.  Register a test user, if registration is supported.
2.  Login with that user.
3.  Capture the returned access token.
4.  Call a protected endpoint such as `GET /users`.
5.  Test missing, invalid, and expired tokens.
6.  Inspect status codes and response bodies.

**Note:** `/api/login` and the response property `token` are workbook
examples. Confirm the real route and JSON field in your own backend.

------------------------------------------------------------------------

# Module 9 --- HTML Form POST Mechanics & SAML Auto-POST Binding

## What is an HTML form POST?

An HTML form can send data to a server using HTTP POST. Form fields can
be visible or hidden.

## What is SAML HTTP-POST binding?

The Identity Provider returns an HTML page containing a form. The form
includes a hidden `SAMLResponse` field and may include `RelayState`.
JavaScript can automatically submit the form to the Service Provider's
Assertion Consumer Service (ACS) endpoint.

### Example auto-submit form

The values below are placeholders, not a real SAML assertion:

``` html
<form method="POST" action="https://app.example.com/saml/consume">
  <input
    type="hidden"
    name="SAMLResponse"
    value="PLACEHOLDER_BASE64_SAML_RESPONSE"
  />

  <input
    type="hidden"
    name="RelayState"
    value="/dashboard"
  />

  <noscript>
    <input type="submit" value="Click to Proceed" />
  </noscript>
</form>

<script>
  window.onload = function () {
    document.forms[0].submit();
  };
</script>
```

## Explanation

-   `method="POST"` --- submits the form using HTTP POST.
-   `action` --- destination ACS endpoint.
-   `SAMLResponse` --- carries the encoded SAML response.
-   `RelayState` --- carries application state or a return-location
    reference.
-   `type="hidden"` --- hides the field from normal page display.
-   `document.forms[0].submit()` --- automatically submits the first
    form.
-   `<noscript>` --- provides a manual fallback if JavaScript is
    disabled.

### Lab 9A --- Inspect a form POST

1.  Use a local HTML test page with dummy values.
2.  Open it in Chrome.
3.  Press F12 → Network and enable Preserve log.
4.  Submit the form.
5.  Inspect the POST request, form fields, and destination.
6.  Do not use real SAML assertions or production session data in public
    test pages.

------------------------------------------------------------------------

# Module 10 --- Single-Page Applications (SPA) vs Server-Side Applications

## What is an SPA?

A Single-Page Application is a web application where much of the
application logic runs in the browser. React and Angular are common
examples.

## What is a server-side application?

A traditional server-side application handles more application logic on
the server and commonly renders HTML responses for the browser.

  -----------------------------------------------------------------------
  SPA                                 Server-side application
  ----------------------------------- -----------------------------------
  Much application logic runs in      More application logic runs on
  browser                             server

  Commonly uses JavaScript frameworks Commonly renders pages on server

  Browser code is visible to users    Server-side code is not sent to
                                      browser

  Cannot safely keep a client secret  Can protect a confidential client
                                      secret on the server
  -----------------------------------------------------------------------

## What is a public client?

A public OAuth client cannot reliably keep a client secret confidential.
A browser-based SPA is treated as a public client because users can
inspect its code and runtime in Developer Tools.

**Never put a confidential OAuth `client_secret` in frontend
JavaScript.**

## What is PKCE?

PKCE means **Proof Key for Code Exchange**. It strengthens the OAuth
authorization-code flow for public clients.

-   `code_verifier` --- a high-entropy, one-time secret generated by the
    client.
-   `code_challenge` --- a transformed value derived from the verifier.
-   `S256` --- SHA-256 method used to derive the challenge.

### Simplified PKCE flow

1.  SPA generates a random `code_verifier`.
2.  SPA calculates the SHA-256 hash of the verifier.
3.  SPA Base64URL-encodes the hash to create the `code_challenge`.
4.  SPA sends the challenge in the authorization request.
5.  During token exchange, the client sends the original verifier.
6.  Authorization server checks that the verifier matches the original
    challenge.

## PowerShell PKCE demonstration

These commands generate a verifier and S256 challenge locally. They do
**not** perform a complete OAuth login.

``` powershell
# Generate 32 random bytes and encode as Base64URL.
$bytes = New-Object byte[] 32
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rng.GetBytes($bytes)

$codeVerifier = [Convert]::ToBase64String($bytes).TrimEnd('=').Replace('+','-').Replace('/','_')

# Hash the verifier with SHA-256.
$sha = [System.Security.Cryptography.SHA256]::Create()
$hash = $sha.ComputeHash([Text.Encoding]::ASCII.GetBytes($codeVerifier))

# Base64URL-encode the hash to create the challenge.
$codeChallenge = [Convert]::ToBase64String($hash).TrimEnd('=').Replace('+','-').Replace('/','_')

# Display values for this local learning exercise.
"code_verifier: $codeVerifier"
"code_challenge: $codeChallenge"
"code_challenge_method: S256"
```

Keep the verifier private in a real authorization flow. Do not publish
it or reuse it.

------------------------------------------------------------------------

# Quick Revision --- Modules 1 to 10

  Module   Topic                          Must remember
  -------- ------------------------------ --------------------------------------------
  1        Web application architecture   Front-channel vs back-channel
  2        HTTP anatomy                   Request, response, headers, 401 vs 403
  3        Redirects                      301, 302, 303, 307 and Location header
  4        Cookies and sessions           Secure, HttpOnly, SameSite
  5        REST and CRUD                  GET, POST, PUT, PATCH, DELETE
  6        JSON and XML                   Data formats in modern and legacy IAM
  7        SOAP vs REST                   XML envelope and WS-Security
  8        Postman                        Login, token capture, bearer authorization
  9        SAML POST binding              Hidden form and automatic POST
  10       SPA and PKCE                   Public clients, verifier and challenge

# Command Cheat Sheet

## Windows PowerShell / cURL

``` powershell
# Inspect HTTP response headers and connection details
curl.exe -Iv "https://example.com"

# Follow redirects
curl.exe -IL "https://example.com"

# Retrieve OIDC discovery metadata
curl.exe -sS "https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"

# Save JSON response to a file
curl.exe -sS "https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration" -o oidc-discovery.json

# Parse JSON in PowerShell
Get-Content .\oidc-discovery.json -Raw | ConvertFrom-Json
```

## Chrome DevTools

``` text
F12                         Open Developer Tools
Network                     Inspect HTTP requests
Preserve log                Keep requests across redirects
Application > Cookies       Inspect cookie attributes
Console                     Run JavaScript checks
```

Console practice:

``` javascript
console.log(document.cookie);
console.log(window.location.href);
console.log(window.location.origin);
```

## Postman

``` text
Method: POST
Body: raw → JSON
Header: Content-Type: application/json

Authorization:
Type: Bearer Token
Token: {{access_token}}
```

------------------------------------------------------------------------

# Self-Assessment --- 15 Questions

1.  What is the difference between front-channel and back-channel
    communication?
2.  What is the difference between HTTP 401 and 403?
3.  What does the HTTP `Location` header do?
4.  What is the difference between 302 and 307 redirects?
5.  Why is the `HttpOnly` cookie attribute useful?
6.  What is the difference between `SameSite=Lax` and `SameSite=Strict`?
7.  Which HTTP method is commonly used to create a resource?
8.  What is the difference between PUT and PATCH?
9.  What is the difference between JSON and XML?
10. What is a SOAP UsernameToken?
11. How does Postman store and reuse an access token?
12. What is the purpose of `SAMLResponse` in an HTML POST form?
13. Why does a SAML form use hidden fields?
14. Why is a browser-based SPA a public OAuth client?
15. What problem does PKCE solve?

## Short Answer Key

1.  Front-channel passes through the browser; back-channel is direct
    server-to-server communication.
2.  401 indicates authentication failure; 403 indicates insufficient
    permission.
3.  It tells the client the URL to visit after a redirect.
4.  302 may change POST to GET; 307 preserves the method.
5.  It prevents JavaScript from reading the cookie through
    `document.cookie`.
6.  Strict is more restrictive; Lax permits some top-level cross-site
    navigations.
7.  POST.
8.  PUT replaces a resource; PATCH updates selected attributes.
9.  JSON uses objects and arrays; XML uses elements and attributes.
10. A SOAP security header element carrying username-based credentials.
11. A Postman script can save a token to an environment variable; later
    requests reuse it.
12. It carries the SAML response to the Service Provider.
13. They carry data without displaying it as ordinary form fields.
14. Browser code and runtime are accessible to the user, so a secret
    cannot be kept confidential.
15. It protects the authorization-code exchange against code
    interception by requiring proof of possession of the verifier.

------------------------------------------------------------------------

**End of study notes --- Workbook 3, Modules 1--10.**

**Practice advice:** Perform each lab using a test account, record the
request and response, and write down what you observed. Never paste real
passwords, private keys, bearer tokens, or session cookies into public
notes.
