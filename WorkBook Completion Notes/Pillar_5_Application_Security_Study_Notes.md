# PILLAR 5

# Application Security, OWASP & Identity Threats

**Beginner-friendly study notes with simple explanations, real-world
examples, diagrams, flowcharts and interview revision**

Based on the uploaded *Pillar 5 \| Beginner Application Security
Workbook*.

------------------------------------------------------------------------

## How to use these notes

1.  Read the simple explanation first.
2.  Follow the flowchart and understand each step.
3.  Study the real-world and IAM-related example.
4.  Revise the important terms and comparisons.
5.  Try answering the interview questions without looking at the
    answers.

**Safety rule:** Use only dedicated training labs and fictional
examples. Never test attacks against company systems, public websites,
colleagues' accounts or real customer data without explicit written
authorization.

## Study roadmap

  -----------------------------------------------------------------------
  Topic                               Main question
  ----------------------------------- -----------------------------------
  5.1 Authentication vs Authorization Who are you, and what are you
                                      allowed to do?

  5.2 OWASP Top 10 concepts           What common application weaknesses
                                      should we recognize?

  5.3 XSS & token theft               Can unsafe browser script affect a
                                      logged-in user?

  5.4 CSRF                            Can another site trick a logged-in
                                      browser into sending a request?

  5.5 CORS                            Which website origins may read a
                                      browser response?

  5.6 Modern identity attacks         How do attackers misuse passwords
                                      and sessions?

  5.7 Zero Trust                      Should access be trusted, or
                                      verified each time?

  5.8 PAM                             How do we protect powerful
                                      administrator accounts?
  -----------------------------------------------------------------------

## Key vocabulary

  -----------------------------------------------------------------------
  Term                                Simple meaning
  ----------------------------------- -----------------------------------
  IAM                                 Identity and Access Management:
                                      managing users and their access.

  SSO                                 Single Sign-On: one trusted login
                                      used to access multiple
                                      applications.

  API                                 A way for software systems to
                                      communicate.

  Session                             Information that helps a website
                                      remember a logged-in user.

  Least privilege                     Give only the access required to do
                                      a job.

  MFA                                 Multi-Factor Authentication: use
                                      more than one type of proof.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 5.1 Authentication vs Authorization

These two ideas are related, but they answer different questions. A user
may prove their identity and still not have permission to perform a
particular action.

## Simple explanation

-   **Authentication (AuthN)** checks **who you are**.
-   **Authorization (AuthZ)** checks **what you are allowed to do**.

Imagine an office: the guard checks your ID at the entrance
(authentication). Your ID card then decides whether you can enter the
cafeteria or server room (authorization).

  -----------------------------------------------------------------------
  Concept                 Question                IAM / application
                                                  example
  ----------------------- ----------------------- -----------------------
  Authentication          Who are you?            PingFederate verifies a
                                                  user's login or accepts
                                                  a trusted Identity
                                                  Provider response.

  Authorization           What can you do?        An application checks
                                                  whether the user can
                                                  view a payroll record
                                                  or approve an invoice.
  -----------------------------------------------------------------------

## Authentication factors

  Factor       Meaning              Example
  ------------ -------------------- --------------------------------
  Knowledge    Something you know   Password, PIN
  Possession   Something you have   Registered phone, security key
  Inherence    Something you are    Fingerprint, face verification

## RBAC vs ABAC

  -----------------------------------------------------------------------
  Model                   How access is decided   Example
  ----------------------- ----------------------- -----------------------
  **RBAC -- Role-Based    Based on assigned role  A Finance Approver role
  Access Control**                                can approve invoices.

  **ABAC --               Based on attributes of  Allow only if
  Attribute-Based Access  user, resource or       department is Finance,
  Control**               situation               invoice is below
                                                  approval limit and
                                                  tenant IDs match.
  -----------------------------------------------------------------------

## Authentication and API access flow

``` text
1. User enters credentials
          |
          v
2. Identity Provider verifies identity
          |
          v
3. Application receives trusted login result / token
          |
          v
4. User requests a protected action
          |
          v
5. Application checks role, attributes and resource
          |
          v
6. ALLOW or DENY the action
```

**Example:** Ravi logs in successfully. That proves his identity. If
Ravi is not a Finance Approver, the system must still deny his
invoice-approval request.

## OIDC and OAuth connection

An **OIDC ID Token** contains authentication-related claims for the
client. An **OAuth access token** is used to access a protected API.
They are not interchangeable.

## Interview questions

-   **Q: Can authentication succeed while authorization fails?**\
    **A:** Yes. The user is valid but does not have permission.
-   **Q: What is least privilege?**\
    **A:** Give a user only the permissions needed for their work.
-   **Q: Should an IAM administrator automatically approve invoices?**\
    **A:** No. Admin duties and finance approval duties should be
    separated unless explicitly required and approved.

------------------------------------------------------------------------

# 5.2 OWASP Top 10 -- Core Concepts

OWASP is a community that publishes practical guidance to improve
application security. The OWASP Top 10 is an awareness list of important
web application risk categories; it is not a complete security standard.

## Four focus areas in this workbook

  -----------------------------------------------------------------------
  Risk area               Simple explanation      IAM example / impact
  ----------------------- ----------------------- -----------------------
  Broken Access Control   A user can see or       Employee A changes an
                          change something they   invoice ID and sees
                          should not.             Employee B's invoice.

  Identification &        The application does    Weak login protections
  Authentication Failures not reliably verify     allow repeated password
                          identity or protect     guesses.
                          login/session           
                          processes.              

  Cryptographic Failures  Sensitive data is not   Login details travel
                          protected properly.     without appropriate
                                                  transport protection.

  Security                The system is deployed  An admin console is
  Misconfiguration        with unsafe settings.   public or exposes
                                                  default
                                                  credentials/debug
                                                  details.
  -----------------------------------------------------------------------

## How an access-control weakness happens

``` text
User signs in successfully
          |
          v
User changes a record ID in the URL or API request
          |
          v
Server fails to check record ownership / permission
          |
          v
Another user's private record is returned
          |
          v
BROKEN ACCESS CONTROL
```

> **Critical security rule:** Hiding an Admin button in the frontend is
> not security. The backend/API must check permission on every protected
> action.

## Practical examples and controls

  -----------------------------------------------------------------------
  Scenario                Risk                    Useful control
  ----------------------- ----------------------- -----------------------
  Changing `/invoice/100` Broken Access Control   Check the logged-in
  to `/invoice/101` shows                         user's permission and
  another customer's                              ownership on the
  invoice                                         server.

  Unlimited password      Authentication failure  Rate limiting,
  attempts                                        throttling, monitoring
                                                  and suitable MFA.

  Login details sent over Cryptographic failure   Use HTTPS/TLS and
  an unencrypted                                  secure configuration.
  connection                                      

  Public admin console    Security                Restrict admin access,
  with default password   misconfiguration        change defaults, harden
                                                  and review
                                                  configuration.
  -----------------------------------------------------------------------

## Interview questions

-   **Q: What is OWASP Top 10?**\
    **A:** An awareness list of important web application security risk
    categories.
-   **Q: What is Broken Access Control?**\
    **A:** A user can access data or perform actions beyond their
    permission.
-   **Q: Why is a hidden button not enough?**\
    **A:** A user can call the API directly; the server must enforce
    authorization.

------------------------------------------------------------------------

# 5.3 Cross-Site Scripting (XSS) & Token Theft

XSS happens when a website handles untrusted input unsafely and the
browser treats it as executable script instead of ordinary text.

## Simple example

Imagine a feedback page that should display a visitor's comment. If the
website inserts that comment as executable HTML/script instead of safe
text, a malicious comment may run in the browser of anyone who views the
page.

## Types of XSS

  -----------------------------------------------------------------------
  Type                    What happens            Example
  ----------------------- ----------------------- -----------------------
  Stored XSS              Unsafe content is saved A saved comment runs
                          and shown to users      when another person
                          later.                  opens the page.

  Reflected XSS           Unsafe input is         A search term is
                          immediately included in displayed unsafely on
                          a response.             the results page.

  DOM-based XSS           Browser-side code puts  A page reads URL data
                          untrusted data into an  and inserts it as HTML.
                          unsafe page location.   
  -----------------------------------------------------------------------

## XSS impact on IAM

-   A script running inside a trusted application page may act as the
    logged-in user.
-   If a bearer token is stored where JavaScript can read it, XSS may
    expose that token.
-   **HttpOnly** prevents JavaScript from reading a cookie value, but it
    does not stop every action that injected script may perform in the
    page.
-   Use safe output handling, context-aware encoding, safe DOM APIs,
    suitable Content Security Policy (CSP) and secure session design.

## Conceptual XSS impact flow

``` text
Attacker-controlled input enters the application
          |
          v
Application handles input unsafely
          |
          v
Browser executes injected script in the trusted page
          |
          v
Script may act as the logged-in user or read
JavaScript-accessible tokens
          |
          v
Sensitive session or account actions may be at risk
```

> **HttpOnly in simple words:** HttpOnly makes a cookie harder for
> JavaScript to read. It is a useful layer of protection, not a complete
> fix for XSS.

## Defensive checklist

  -----------------------------------------------------------------------
  Do                                  Avoid
  ----------------------------------- -----------------------------------
  Display user comments as text where Treat untrusted user input as
  appropriate.                        trusted HTML.

  Encode output for its context.      Assume input validation alone
                                      prevents every XSS case.

  Use safe DOM APIs and a suitable    Assume HttpOnly alone fixes XSS.
  CSP.                                

  Protect sessions and minimize token Store sensitive bearer tokens where
  exposure.                           unnecessary scripts can read them.
  -----------------------------------------------------------------------

## Interview questions

-   **Q: What does XSS stand for?**\
    **A:** Cross-Site Scripting.
-   **Q: Name the three types.**\
    **A:** Stored, Reflected and DOM-based XSS.
-   **Q: Does HttpOnly fix XSS?**\
    **A:** No. It reduces JavaScript access to the cookie value, but
    injected script may still perform actions in the page.

------------------------------------------------------------------------

# 5.4 Cross-Site Request Forgery (CSRF)

CSRF is when a victim's browser is tricked into sending an unwanted
request to a website where the victim is already logged in. The browser
may automatically attach the site's session cookie.

## Real-world example

You are signed in to your banking website. You visit a malicious page in
another tab. That page attempts to make your browser send a transfer
request to the bank. If the bank trusts only the session cookie and does
not verify that the request was intentionally made by you, the request
may succeed.

## CSRF protection methods

  -----------------------------------------------------------------------
  Control                             Simple explanation
  ----------------------------------- -----------------------------------
  Anti-CSRF token                     A random value associated with the
                                      user's session. The server checks
                                      it on a state-changing request.

  SameSite cookie                     A cookie setting that restricts
                                      when cookies are sent in cross-site
                                      situations.

  Origin checking                     The server checks the origin of a
                                      request as an additional defense.

  OAuth state                         A random value linking an
                                      authorization response to the login
                                      request started by the client.

  SAML RelayState                     Carries state through a SAML flow;
                                      the application must validate it
                                      safely.
  -----------------------------------------------------------------------

## CSRF request flow

``` text
User logs in to trusted website
          |
          v
Browser stores / sends session cookie
          |
          v
User visits an untrusted website
          |
          v
Untrusted page causes a request to the trusted website
          |
          v
Server checks cookie but misses CSRF validation
          |
          v
Unwanted state-changing action may happen
```

## Expected request vs forged request

  -----------------------------------------------------------------------
  Expected user action                Possible CSRF
  ----------------------------------- -----------------------------------
  User clicks the official Transfer   A different website silently causes
  button after login. The server      the browser to send a transfer
  validates a correct anti-CSRF       request. The application accepts it
  token.                              based only on the session cookie.

  -----------------------------------------------------------------------

> **Remember:** OAuth state and SAML RelayState are not magic security
> controls. The application must generate, bind and validate the values
> correctly.

## Interview questions

-   **Q: Why are cookies relevant to CSRF?**\
    **A:** Browsers may automatically attach session cookies to
    requests.
-   **Q: What is an anti-CSRF token?**\
    **A:** A value the server checks to confirm a state-changing request
    is associated with the user's legitimate session.
-   **Q: What if OAuth state does not match?**\
    **A:** Reject the response; do not continue the login flow.

------------------------------------------------------------------------

# 5.5 CORS -- Cross-Origin Resource Sharing

A website loaded from one origin may try to read data from another
origin. Browsers apply the **Same-Origin Policy (SOP)** to limit this.
CORS lets a server tell the browser which other origins may read its
response.

## What is an origin?

An origin is made from three parts:

**Scheme + Host + Port**

If any one of these differs, the origin is different.

  -----------------------------------------------------------------------
  URL                                 Why it is different
  ----------------------------------- -----------------------------------
  `https://portal.example.com`        Reference origin.

  `https://api.example.com`           Different host.

  `http://portal.example.com`         Different scheme (HTTP instead of
                                      HTTPS).

  `https://portal.example.com:8443`   Different port.
  -----------------------------------------------------------------------

> **CORS is not authentication:** CORS controls whether browser
> JavaScript may read a cross-origin response. It does not prove who the
> user is and does not decide business permissions on the API.

## CORS preflight flow

``` text
Browser wants to send a cross-origin request
          |
          v
Browser sends OPTIONS preflight for some requests
          |
          v
Server responds with allowed origin, methods and headers
          |
          v
Browser checks the CORS response
          |
          v
If permitted, browser sends the actual request
          |
          v
API still performs authentication and authorization
```

## Important CORS response headers

  ------------------------------------------------------------------------
  Header                               Meaning
  ------------------------------------ -----------------------------------
  `Origin`                             Which website initiated the browser
                                       request.

  `Access-Control-Allow-Origin`        Which origin is allowed to read the
                                       response.

  `Access-Control-Allow-Methods`       Which HTTP methods are permitted.

  `Access-Control-Allow-Headers`       Which request headers are
                                       permitted.

  `Access-Control-Allow-Credentials`   Whether browser credentials may be
                                       included and response exposed.
                                       Wildcard origin cannot be used with
                                       credentials.
  ------------------------------------------------------------------------

## Workbook example

Frontend: `https://portal.example.com` calls API:
`https://api.example.com`.

The browser sends an OPTIONS request with Origin set to the portal and
asks to use GET with the Authorization header. A response allowing
`https://portal.example.com`, GET and Authorization matches the request.
A response allowing `https://another-site.example` does not.

> **Postman vs browser:** Postman may call an API successfully while a
> browser blocks JavaScript from reading the response because of CORS.
> The API must still authenticate and authorize every request.

## Interview questions

-   **Q: What does SOP do?**\
    **A:** It limits how one origin can interact with/read data from
    another origin in the browser.
-   **Q: Which method is used for preflight?**\
    **A:** OPTIONS.
-   **Q: Is CORS an API permission system?**\
    **A:** No. It is a browser cross-origin response-sharing mechanism.

------------------------------------------------------------------------

# 5.6 Modern Identity Attacks

Identity attacks target passwords, login processes and authenticated
sessions. Learn the pattern each attack uses and the warning signs
defenders may see.

## Common identity attacks

  -----------------------------------------------------------------------
  Attack                  Beginner explanation    Possible warning sign
  ----------------------- ----------------------- -----------------------
  Credential stuffing     Trying                  Many accounts
                          username/password pairs attempted; some
                          leaked from other       successful logins from
                          services.               unusual contexts.

  Password spraying       Trying a small number   Many usernames receive
                          of common passwords     failures from one
                          against many accounts.  source in a short time.

  Brute force             Trying many password    Repeated failures
                          guesses against one     against a target
                          account or a small set  account.
                          of accounts.            

  Session hijacking /     Stealing and reusing a  A valid session appears
  pass-the-cookie         valid session cookie or from an unusual device
                          token.                  or context.

  AiTM phishing           A phishing proxy relays Unusual sign-in context
                          traffic between a user  or session use after a
                          and a real service and  phishing event.
                          may capture session     
                          material.               
  -----------------------------------------------------------------------

## How to distinguish password attacks

``` text
CREDENTIAL STUFFING
Many leaked username + password pairs
                  |
                  v
PASSWORD SPRAYING
Few common passwords across many users
                  |
                  v
BRUTE FORCE
Many guesses against one / few accounts
```

## Fictional log investigation

  Time    Event
  ------- ------------------------------------------------
  09:01   `login_failed user=asha source=203.0.113.10`
  09:02   `login_failed user=ben source=203.0.113.10`
  09:03   `login_failed user=chris source=203.0.113.10`
  09:04   `login_failed user=deepa source=203.0.113.10`
  09:05   `login_failed user=asha source=203.0.113.10`
  09:08   `login_success user=asha source=198.51.100.20`
  09:09   `session_created user=asha device=unknown`

## What can we infer?

-   Four different usernames were attempted from the same source
    address.
-   This pattern may indicate password spraying. It is an indicator, not
    proof.
-   The successful login from another address and the unknown device
    should be investigated.
-   Check device history, MFA events, location/context, session details,
    user confirmation and related alerts before declaring compromise.

## Response and prevention

  -----------------------------------------------------------------------
  If compromise is confirmed          Defensive measures
  ----------------------------------- -----------------------------------
  Revoke active sessions/tokens;      MFA; phishing-resistant
  reset affected credentials; notify  FIDO2/WebAuthn keys or passkeys;
  the security/IAM team and user.     rate limiting; detection rules;
                                      sign-in risk monitoring.

  -----------------------------------------------------------------------

> **MFA reminder:** MFA helps reduce account compromise.
> Phishing-resistant authentication such as FIDO2/WebAuthn security keys
> or passkeys is stronger against real-time credential phishing than
> codes that can be relayed.

## Interview questions

-   **Q: Credential stuffing vs password spraying?**\
    **A:** Stuffing uses leaked pairs; spraying tries a small number of
    passwords across many accounts.
-   **Q: What is pass-the-cookie?**\
    **A:** Reusing a stolen valid session cookie/token.
-   **Q: What does AiTM mean?**\
    **A:** Adversary-in-the-Middle phishing.

------------------------------------------------------------------------

# 5.7 Zero Trust Architecture

Zero Trust is often summarized as **Never Trust, Always Verify**. Access
should not be granted simply because a person is inside the office
network or has logged in once.

## What should be checked?

The system evaluates identity, device, requested resource, requested
action and risk. Access may need to be reassessed when context or risk
changes.

## Four Zero Trust principles

  -----------------------------------------------------------------------
  Principle               Plain-language meaning  Example
  ----------------------- ----------------------- -----------------------
  Explicit verification   Check identity and      Verify user and device
                          relevant context before before opening a
                          granting access.        sensitive application.

  Least privilege         Give only permissions   Employee can view their
                          needed.                 own payslip, not every
                                                  employee's payslip.

  Assume breach           Design as if an         Segment access and
                          attacker may already be monitor unusual admin
                          inside; limit damage    activity.
                          and monitor activity.   

  Continuous evaluation   Reassess access when    Require another check
                          risk or context         if device risk changes.
                          changes, where          
                          supported.              
  -----------------------------------------------------------------------

## Zero Trust access decision flow

``` text
User requests a resource/action
          |
          v
Verify identity and account status
          |
          v
Check device health and context
          |
          v
Check role, attributes and resource sensitivity
          |
          v
Evaluate risk and policy
          |
          v
PERMIT / DENY / REQUIRE ADDITIONAL VERIFICATION
          |
          v
Monitor and reassess when context changes
```

## Example decisions

  -----------------------------------------------------------------------
  Case                    Context                 Suggested decision
  ----------------------- ----------------------- -----------------------
  A                       Authenticated employee, Permit, if policy
                          managed device, low     checks pass.
                          risk, own payslip.      

  B                       Authenticated employee  Deny.
                          requests another        
                          tenant's customer       
                          record.                 

  C                       Finance approver        Require additional
                          requests high-value     verification or deny
                          approval from unmanaged according to policy.
                          device.                 

  D                       Disabled account        Deny.
                          requests access from    
                          office network.         

  E                       Administrator requests  Grant time-limited,
                          production access for a approved access with
                          short approved          monitoring.
                          maintenance task.       
  -----------------------------------------------------------------------

> **Remember:** Being on the office network does not automatically make
> a user trustworthy. Network location is only one possible signal.

## Interview questions

-   **Q: Is Zero Trust a single product?**\
    **A:** No. It is an architecture/security approach.
-   **Q: Name three access signals.**\
    **A:** Identity/account status, device condition, resource
    sensitivity, requested action and risk context.
-   **Q: What does Assume Breach mean?**\
    **A:** Design controls as if an attacker may already be inside, so
    access is limited and activity is monitored.

------------------------------------------------------------------------

# 5.8 Privileged Access Management (PAM)

A privileged account can make powerful changes, such as creating users,
changing security settings or administering servers. PAM protects these
high-impact accounts and the sessions that use them.

## Main PAM controls

  -----------------------------------------------------------------------
  PAM control             Simple meaning          Example
  ----------------------- ----------------------- -----------------------
  Vaulting                Keep privileged         An administrator checks
                          passwords/keys in a     out a server credential
                          controlled secure store from an approved vault.
                          and manage their use.   

  Just-In-Time (JIT)      Give elevated access    A database admin
  access                  only when needed and    receives production
                          only for a limited      access for a 30-minute
                          time.                   approved task.

  Session recording /     Record or monitor       A privileged session is
  monitoring              privileged activity for logged for later
                          audit and               review.
                          investigation, subject  
                          to policy and law.      

  Break-glass account     Emergency access used   A controlled emergency
                          only when normal access admin account is used
                          paths are unavailable;  during a critical
                          tightly controlled and  identity outage.
                          reviewed.               
  -----------------------------------------------------------------------

## Safe privileged access flow

``` text
Administrator requests elevated access
          |
          v
Request is approved for a specific task
          |
          v
Credential is obtained through secure vault
          |
          v
JIT access is enabled for limited time
          |
          v
Privileged session is monitored / recorded
          |
          v
Access expires; credential/session is rotated or revoked as required
          |
          v
Activity is reviewed and audited
```

## Break-glass process -- emergency only

  -----------------------------------------------------------------------
  Step                                Safe process to define
  ----------------------------------- -----------------------------------
  1\. Qualify emergency               Define genuine situations, such as
                                      normal access paths being
                                      unavailable during a critical
                                      incident.

  2\. Approval / witness              Identify who can approve or witness
                                      emergency use, with a documented
                                      exception process.

  3\. Secure storage                  Keep emergency credentials in an
                                      approved secure vault with
                                      restricted access.

  4\. Alerting                        Notify the security/IAM team
                                      immediately when the account is
                                      used.

  5\. After use                       Review logs, rotate/reset
                                      credentials, revoke unnecessary
                                      access and document the reason.

  6\. Prevent everyday use            Test periodically under policy,
                                      monitor every use and prohibit
                                      routine administration through the
                                      emergency account.
  -----------------------------------------------------------------------

> **PAM in one sentence:** Vault the secret, grant access only when
> needed, monitor the session, and review emergency use.

## Interview questions

-   **Q: What is a privileged account?**\
    **A:** An account with powerful permissions to administer systems or
    sensitive resources.
-   **Q: What is JIT?**\
    **A:** Just-In-Time access: temporary elevated permission for an
    approved task.
-   **Q: What must happen after break-glass use?**\
    **A:** Alert, investigate/review, document, and rotate or revoke
    credentials/access as appropriate.

------------------------------------------------------------------------

# Capstone: Suspicious Finance Login

## Scenario

An employee says they did not approve an invoice. The IAM team finds
several failed logins against different users, then a successful login
for the employee from an unfamiliar device. The finance application
records an invoice approval request.

## Investigate step by step

``` text
1. Review failed login pattern:
   stuffing, spraying or brute force are possibilities
          |
          v
2. Investigate successful login and unfamiliar device
          |
          v
3. Confirm user identity and review MFA / session evidence
          |
          v
4. Check whether invoice approval was permitted for this user
          |
          v
5. Check CSRF protection on the approval action
          |
          v
6. Check whether XSS could have put the logged-in session at risk
          |
          v
7. Apply Zero Trust checks and contain confirmed compromise
          |
          v
8. If production admin access is needed, use approved PAM controls
```

## Questions to ask

1.  Which attack patterns could explain the login failures? List
    possibilities, not conclusions.
2.  Why should the successful login and new device be investigated?
3.  What evidence proves that the invoice approval was authorized?
4.  What CSRF protection should exist for a state-changing approval
    action?
5.  How could XSS put a logged-in session at risk?
6.  Which Zero Trust signals should be checked?
7.  If a privileged administrator must investigate production, how
    should access be granted?

## Incident summary template

  -----------------------------------------------------------------------
  Section                             Record
  ----------------------------------- -----------------------------------
  What happened?                      Describe the reported event and
                                      timeline.

  What evidence do we have?           Logs, authentication events,
                                      device/session data, application
                                      audit trail.

  Confirmed vs suspected              Separate facts from hypotheses.

  Who/what may be affected?           Users, records, applications,
                                      tenants and systems.

  Immediate containment               Revoke sessions, reset credentials,
                                      block suspicious access as
                                      justified.

  Long-term remediation               Fix root causes, improve controls
                                      and detection.

  How will we verify the fix?         Repeat safe tests, review logs and
                                      confirm policy enforcement.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Fast Revision Sheet

  -----------------------------------------------------------------------
  Term                                Remember this
  ----------------------------------- -----------------------------------
  Authentication                      Who are you?

  Authorization                       What are you allowed to do?

  RBAC                                Access based on role.

  ABAC                                Access based on attributes and
                                      context.

  OWASP                               Awareness of common application
                                      security risks.

  XSS                                 Untrusted input is executed as
                                      script in a trusted page.

  HttpOnly                            Stops JavaScript from reading the
                                      cookie value; not a complete XSS
                                      fix.

  CSRF                                Tricks a logged-in browser into
                                      sending an unwanted request.

  Anti-CSRF token                     Server-checked value for
                                      state-changing requests.

  OAuth state                         Binds the authorization response to
                                      the login attempt.

  SAML RelayState                     Carries state; must be safely
                                      validated.

  Origin                              Scheme + host + port.

  CORS                                Controls browser access to
                                      cross-origin responses.

  Preflight                           OPTIONS check before some
                                      cross-origin requests.

  Credential stuffing                 Uses leaked username/password
                                      pairs.

  Password spraying                   Few passwords across many
                                      usernames.

  Brute force                         Many guesses against one/few
                                      accounts.

  Session hijacking                   Stealing/reusing a valid session
                                      token or cookie.

  AiTM                                Adversary-in-the-Middle phishing.

  Zero Trust                          Never trust, always verify.

  PAM                                 Protect powerful accounts and
                                      privileged sessions.

  Vaulting                            Securely store/manage privileged
                                      credentials.

  JIT                                 Temporary elevated access.

  Break-glass                         Tightly controlled emergency
                                      access.
  -----------------------------------------------------------------------

## Interview-ready comparisons

  -----------------------------------------------------------------------
  Compare                             Difference
  ----------------------------------- -----------------------------------
  AuthN vs AuthZ                      AuthN proves identity; AuthZ
                                      decides permission.

  RBAC vs ABAC                        RBAC uses assigned roles; ABAC
                                      evaluates attributes of
                                      user/resource/context.

  XSS vs CSRF                         XSS runs unsafe script in a trusted
                                      page; CSRF tricks a logged-in
                                      browser into sending a request.

  SOP vs CORS                         SOP is the browser restriction;
                                      CORS is the server-controlled
                                      mechanism for allowing selected
                                      cross-origin response access.

  Credential stuffing vs spraying     Stuffing uses leaked pairs;
                                      spraying tries a few passwords
                                      across many accounts.

  JIT vs break-glass                  JIT is planned, temporary
                                      elevation; break-glass is
                                      exceptional emergency access.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Practice Questions with Short Answers

  ------------------------------------------------------------------------
                            \# Question              Short answer
  ---------------------------- --------------------- ---------------------
                             1 Differentiate         Authentication
                               authentication and    verifies identity;
                               authorization.        authorization checks
                                                     permissions.

                             2 Why can login succeed The user is
                               but the API return    authenticated but
                               403 Forbidden?        lacks permission for
                                                     that resource/action.

                             3 Explain RBAC with an  A Finance Approver
                               example.              role can approve
                                                     invoices.

                             4 What is Broken Access A user can access
                               Control?              data/actions outside
                                                     their permission.

                             5 Name the four OWASP   Broken Access
                               focus areas in this   Control;
                               workbook.             Identification &
                                                     Authentication
                                                     Failures;
                                                     Cryptographic
                                                     Failures; Security
                                                     Misconfiguration.

                             6 Name the three XSS    Stored, Reflected and
                               types.                DOM-based.

                             7 What does HttpOnly    It prevents
                               protect against?      JavaScript from
                                                     reading the cookie
                                                     value.

                             8 Explain CSRF.         A malicious site
                                                     tricks a logged-in
                                                     browser into sending
                                                     an unwanted request.

                             9 Purpose of an         The server checks it
                               anti-CSRF token?      before processing a
                                                     state-changing
                                                     request.

                            10 What is OAuth state   It links the
                               used for?             authorization
                                                     response to the login
                                                     attempt and should be
                                                     validated.

                            11 What is a browser     Scheme, host and
                               origin?               port.

                            12 What is a CORS        An OPTIONS request
                               preflight?            that checks whether a
                                                     cross-origin request
                                                     is permitted.

                            13 Credential stuffing   Leaked
                               vs password spraying? username/password
                                                     pairs vs a few
                                                     passwords tried
                                                     across many users.

                            14 What is session       Stealing and reusing
                               hijacking?            a valid session
                                                     cookie or token.

                            15 State two Zero Trust  Explicit verification
                               principles.           and least privilege
                                                     (also assume breach
                                                     and continuous
                                                     evaluation).

                            16 Explain vaulting, JIT Secure credential
                               and break-glass.      storage; temporary
                                                     elevation; tightly
                                                     controlled emergency
                                                     access.
  ------------------------------------------------------------------------

## Final takeaway

Secure applications do more than check passwords. They enforce
permissions on the server, protect browser sessions, validate requests,
monitor identity threats, continuously evaluate access and tightly
govern administrator privileges.

------------------------------------------------------------------------

**Source note:** These notes are based on the uploaded 14-page *Pillar 5
\| Beginner Application Security Workbook*. The original workbook's
eight topics, examples, lab scenarios and learning outcomes have been
retained and reorganized for easier study. Explanations and flow
diagrams are teaching aids; follow your organization's policies and
authorized lab rules.
