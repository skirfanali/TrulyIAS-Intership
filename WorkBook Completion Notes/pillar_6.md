# PILLAR 6: Developer Foundations, SCIM & Hands-on Tooling
## Complete Learning Notes with Diagrams, Practical Code & YouTube Demonstration Guide

---

# 1. Introduction to Pillar 6

Pillar 6 focuses on the development concepts and tools that an Identity and Access Management (IAM) engineer should understand.

An IAM engineer does not necessarily need to become a full-stack developer, but should understand how applications communicate, how users are provisioned, how authentication works, and how to manage development environments.

### Main topics

1. HTML Form POST and SAML HTTP-POST Binding
2. SPA vs Traditional Server-Side Applications
3. OAuth 2.0 and PKCE
4. SCIM 2.0 Provisioning
5. JavaScript and Python for Identity
6. Git and GitHub
7. Docker and Containerization

### How everything connects to IAM

```text
                  USER
                   |
                   v
            WEB APPLICATION
                   |
          +--------+--------+
          |                 |
          v                 v
       Browser          Backend
          |                 |
          v                 v
         IdP             REST APIs
          |                 |
          v                 v
    Authentication     SCIM Provisioning
          |                 |
          v                 v
       Tokens          User / Groups
          |                 |
          +--------+--------+
                   |
                   v
             IAM SYSTEM
```

An IAM engineer works with all these components to manage user identities, authentication, authorization and provisioning.

---

# 2. HTML Form POST & SAML HTTP-POST Binding

## 2.1 What is an HTML Form?

An HTML form is used to collect information and send it to a server.

For example, when you enter your username and password on a login page and click Submit, the browser sends the entered information to a backend server.

An HTML form has three important attributes:

- `action`: Specifies the destination URL.
- `method`: Specifies how the data is sent.
- `input`: Contains the information being submitted.

### GET vs POST

| GET | POST |
|---|---|
| Sends data in the URL query string | Sends data in the request body |
| Suitable for retrieving information | Commonly used to submit information |
| Data can appear in browser history | Request body is not normally displayed in the URL |
| URL length limitations apply | Supports larger request bodies |

Important: POST does not automatically encrypt information. HTTPS provides transport encryption.

## 2.2 What is a hidden input?

A hidden input is an HTML field that stores a value without displaying it as a normal visible field.

Example:

```html
<input
  type="hidden"
  name="SAMLResponse"
  value="SAMPLE_RESPONSE"
/>
```

The user cannot see this field in the normal page interface, but the browser includes its value when the form is submitted.

A hidden field is not a security mechanism. Users can inspect and modify it using browser developer tools.

## 2.3 What is SAML HTTP-POST Binding?

SAML stands for Security Assertion Markup Language.

It is an XML-based standard used to exchange authentication information between an Identity Provider (IdP) and a Service Provider (SP).

Example:

- IdP: PingFederate
- SP: A company application
- User: Employee trying to access the application

### SAML HTTP-POST flow diagram

```text
       1. User opens application
                 |
                 v
          Service Provider
                 |
          Redirect to IdP
                 |
                 v
          Identity Provider
                 |
          User authenticates
                 |
                 v
        IdP creates signed
           SAML Response
                 |
                 v
       HTML page is generated
       with hidden SAMLResponse
                 |
                 v
          User's Browser
                 |
          HTTP POST request
                 |
                 v
          SP ACS Endpoint
                 |
          Validate SAML Response
                 |
                 v
          Create User Session
                 |
                 v
          Application Access
```

### What is ACS?

ACS stands for Assertion Consumer Service.

It is an endpoint provided by the Service Provider that receives the SAML response from the browser.

For example:

```text
https://application.example.com/saml/acs
```

### What happens during validation?

The SP must validate:

1. Digital signature.
2. Issuer.
3. Audience.
4. Destination and recipient.
5. Assertion validity and time conditions.
6. Request correlation and replay protections, where applicable.

The browser only transports the response. It does not validate the SAML assertion.

## 2.4 Practical Demonstration: HTML Form POST

**Tools required:**
- JSFiddle: https://jsfiddle.net/
- HTTPBin: https://httpbin.org/post

Paste this code into the HTML panel:

```html
<form action="https://httpbin.org/post" method="POST">

  <input
    type="hidden"
    name="SAMLResponse"
    value="RHVtbXktU0FNTC1SZXNwb25zZQ=="
  >

  <button type="submit">
    Submit Sample Response
  </button>

</form>
```

### What to demonstrate

1. Open JSFiddle.
2. Paste the HTML code.
3. Click Run.
4. Click Submit Sample Response.
5. Observe the HTTPBin response.
6. Find the `form` section in the response.
7. Identify the `SAMLResponse` field.

Expected result:

```json
{
  "form": {
    "SAMLResponse": "RHVtbXktU0FNTC1SZXNwb25zZQ=="
  }
}
```

### Demonstrate automatic submission

Add the following JavaScript:

```html
<script>
  document.forms[0].submit();
</script>
```

Now the browser automatically submits the form when the page loads.

**Important:** This is only a simulated SAML form POST. HTTPBin does not validate the SAML response or perform authentication.

### What to explain in your video

"In SAML HTTP-POST binding, the Identity Provider generates a signed SAML response and places it inside a hidden HTML form field. The browser submits that form to the Service Provider's ACS endpoint. The Service Provider then validates the signature and other SAML conditions before creating a session."

---

# 3. SPA vs Traditional Server-Side Applications

## 3.1 What is a Single-Page Application (SPA)?

A Single-Page Application is a web application in which most of the user interface is loaded and updated using JavaScript without completely reloading the page for every navigation.

Examples:
- React applications.
- Angular applications.
- Vue applications.

In a SPA, the browser runs most of the frontend application logic and communicates with backend APIs.

### SPA architecture

```text
          USER
            |
            v
       WEB BROWSER
            |
            v
      React / Angular
        Application
            |
            | API Request
            v
       Backend API
            |
            v
        Database
```

Example: A user opens a React application. The browser loads the frontend and then sends API requests to the backend to retrieve user information.

## 3.2 What is a Traditional Server-Side Application?

A traditional server-side application handles much of its application logic on the backend server.

The server processes requests, communicates with databases and often generates HTML pages that are returned to the browser.

Examples:
- Java JSP and Servlet applications.
- Spring MVC applications.
- Server-rendered Node.js applications.
- ASP.NET MVC applications.

### Server-side architecture

```text
          USER
            |
            v
         BROWSER
            |
       HTTP Request
            |
            v
      BACKEND SERVER
            |
       Application Logic
            |
            v
         DATABASE
            |
            v
       Generate HTML
            |
            v
          BROWSER
```

## 3.3 SPA vs Server-Side Application

| Feature | SPA | Server-side |
|---|---|---|
| Main execution | Browser | Backend server |
| Frontend | JavaScript-driven | Server-rendered HTML, often with JavaScript |
| API communication | Usually direct from browser | Often handled by backend |
| Client secret | Cannot be kept confidential | Can be protected server-side |
| Example | React | JSP / Spring MVC |

Remember: Modern applications can combine both architectures.

## 3.4 Public vs Confidential OAuth Clients

OAuth 2.0 defines two important client categories.

**Public client:**

A public client cannot securely protect a client secret.

Example: A React application running inside a user's browser.

Why?

Because users can inspect browser-delivered JavaScript, HTML and configuration.

**Confidential client:**

A confidential client can protect its credentials on a controlled backend server.

Example: A Spring Boot application that communicates with an Identity Provider from its backend.

### Architecture comparison

```text
SPA / PUBLIC CLIENT

Browser
   |
   | OAuth Request
   v
Identity Provider
   |
   v
Authorization Code
   |
   v
Browser Client
   |
   v
Token Endpoint (PKCE)


SERVER-SIDE / CONFIDENTIAL CLIENT

Browser
   |
   v
Backend Server
   |
   | Client Authentication
   v
Identity Provider
   |
   v
Token Endpoint
   |
   v
Backend Stores Tokens
```

## 3.5 What is PKCE?

PKCE stands for Proof Key for Code Exchange.

PKCE is an OAuth 2.0 security extension that protects the authorization code exchange, especially for public clients.

It prevents an attacker who intercepts an authorization code from easily redeeming it without the original code verifier.

### PKCE flow

```text
          CLIENT
             |
             v
    Generate code_verifier
             |
             v
    Calculate SHA256 Hash
             |
             v
      code_challenge
             |
             v
       Authorization
          Request
             |
             v
     Identity Provider
             |
             v
     Authorization Code
             |
             v
       Token Request
             |
       code_verifier
             |
             v
       IdP verifies
        the match
             |
             v
        Access Token
```

### Important terms

- `code_verifier`: Random secret generated by the client.
- `code_challenge`: Derived value sent during authorization.
- `S256`: SHA-256 transformation used to generate the challenge.

## 3.6 Practical Demonstration: Generate a PKCE Challenge

Open JSFiddle and paste the following into its JavaScript panel:

```javascript
async function generatePKCE() {

  // Generate a random verifier
  const randomBytes = crypto.getRandomValues(
    new Uint8Array(32)
  );

  const verifier = btoa(
    String.fromCharCode(...randomBytes)
  )
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=/g, "");

  // Convert verifier to bytes
  const data = new TextEncoder().encode(verifier);

  // Generate SHA-256 challenge
  const hash = await crypto.subtle.digest(
    "SHA-256",
    data
  );

  const challenge = btoa(
    String.fromCharCode(
      ...new Uint8Array(hash)
    )
  )
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=/g, "");

  console.log("Code Verifier:", verifier);
  console.log("Code Challenge:", challenge);
}

generatePKCE();
```

### Expected output

```text
Code Verifier: <random-value>
Code Challenge: <random-value>
```

Both values will be different-looking strings.

### What to explain

"Here I am generating a random code verifier. Then I use SHA-256 to create a code challenge. The authorization request carries the challenge, and the token request carries the original verifier. The Identity Provider checks whether they match."

Note: This demonstrates PKCE value generation, not a complete OAuth login or token exchange.

---

# 4. SCIM 2.0 Protocol Fundamentals

## 4.1 What is SCIM?

SCIM stands for System for Cross-domain Identity Management.

SCIM is a standard protocol used to automate the provisioning and management of users and groups between an Identity Provider and an application.

For example, suppose a company has 500 employees and uses multiple applications.

Without SCIM, an administrator may need to manually create accounts in each application.

With SCIM, the IAM system can automatically send user and group information to supported applications.

## 4.2 Why is SCIM needed?

Consider this scenario:

A new employee named Alex Taylor joins the Finance department.

The organization needs to:

1. Create Alex's account.
2. Assign Alex to Finance.
3. Update Alex's details if the department changes.
4. Disable Alex's account when Alex leaves.

SCIM standardizes the API communication used for these provisioning operations.

## 4.3 SCIM architecture

```text
          HR SYSTEM
              |
              v
        IDENTITY PROVIDER
              |
              |
       SCIM 2.0 REST API
              |
              v
       SaaS APPLICATION
              |
       +------+------+
       |             |
       v             v
    USERS          GROUPS
       |             |
       v             v
    Alex Taylor    Finance
```

Example IAM products that support SCIM integrations include PingFederate and other identity platforms, depending on the connector and configuration.

## 4.4 SCIM User Resource

A User resource represents an individual account.

Example:

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "userName": "alex.taylor@example.test",
  "name": {
    "givenName": "Alex",
    "familyName": "Taylor"
  },
  "active": true,
  "emails": [
    {
      "value": "alex.taylor@example.test",
      "primary": true
    }
  ]
}
```

### Explanation of attributes

| Attribute | Meaning |
|---|---|
| schemas | Identifies the SCIM schema |
| userName | User's login name |
| givenName | First name |
| familyName | Last name |
| active | Indicates account status |
| emails | Email addresses |

## 4.5 SCIM Group Resource

A Group resource represents a collection of users.

Example:

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:Group"
  ],
  "displayName": "Finance",
  "members": [
    {
      "value": "user-101",
      "type": "User"
    }
  ]
}
```

Here, the Finance group contains a user with the ID `user-101`.

## 4.6 Important SCIM endpoints

| HTTP method | Endpoint | Purpose |
|---|---|---|
| GET | /Users | Retrieve users |
| GET | /Users/{id} | Retrieve one user |
| POST | /Users | Create a user |
| PATCH | /Users/{id} | Update selected attributes |
| PUT | /Users/{id} | Replace the resource |
| DELETE | /Users/{id} | Delete a user |
| GET | /Groups | Retrieve groups |

### What is PATCH vs PUT?

PATCH modifies selected attributes.

PUT replaces the resource representation.

For example, if Alex changes departments, PATCH may update only the department-related attribute.

## 4.7 SCIM Joiner, Mover and Leaver

This is an important IAM lifecycle concept.

**Joiner:** A new employee joins the organization. The IAM system creates the user's application account.

**Mover:** An employee changes departments. The IAM system updates their attributes and group memberships.

**Leaver:** An employee leaves the organization. The IAM system disables the account or removes access according to company policy.

### Lifecycle diagram

```text
       NEW EMPLOYEE
             |
             v
          JOINER
       Create Account
             |
             v
          EMPLOYEE
             |
        Department Change
             |
             v
           MOVER
      Update Attributes
      Update Group Access
             |
             v
       EMPLOYEE LEAVES
             |
             v
          LEAVER
       Disable Account
       Remove Access
```

## 4.8 Practical Demonstration: SCIM Playground

Tools:

- SCIM Playground: https://scim.dev/playground/
- SCIM Guide: https://scim.dev/guide/api/

### Step 1: Open the playground

Open the SCIM playground and inspect its interface and usage instructions.

### Step 2: Check ServiceProviderConfig

Look for the request:

```http
GET /ServiceProviderConfig
```

This endpoint describes the capabilities supported by the SCIM service, such as filtering, patching, bulk operations and supported authentication schemes.

### Step 3: Retrieve users

```http
GET /Users
```

Observe the returned user records.

Identify:

- id
- userName
- active
- emails

### Step 4: Create a user

Use a POST request to:

```http
POST /Users
```

Sample JSON:

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "userName": "sam.lee@example.test",
  "name": {
    "givenName": "Sam",
    "familyName": "Lee"
  },
  "active": true,
  "emails": [
    {
      "value": "sam.lee@example.test",
      "primary": true
    }
  ]
}
```

Expected result: A successful creation commonly returns:

```text
HTTP/1.1 201 Created
```

The response normally includes the created SCIM resource and its server-assigned ID.

### Step 5: Update a user

```http
PATCH /Users/{id}
```

Example PATCH body:

```json
{
  "schemas": [
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations": [
    {
      "op": "replace",
      "path": "active",
      "value": false
    }
  ]
}
```

This example changes the user's active status to false.

### Step 6: Retrieve groups

```http
GET /Groups
```

Identify the group names and their members.

Important: Use only the playground's dummy environment. Its data and persistence behavior can change.

### What to explain in your video

"SCIM is used for automated identity provisioning. For example, when an employee joins a company, the Identity Provider can send a POST request to the application's /Users endpoint to create the account. If the employee leaves, the IAM system can deactivate the account through a PATCH operation."

---

# 5. Scripting for Identity: JavaScript and Python

## 5.1 Why do IAM engineers need scripting?

IAM engineers often work with APIs, JSON responses, user attributes and identity data.

Scripting helps automate repetitive activities such as:

- Reading user information.
- Processing API responses.
- Checking account status.
- Generating reports.
- Inspecting JWT payloads.
- Automating provisioning-related tasks.

You do not need advanced programming knowledge to understand these tasks.

## 5.2 What is JSON?

JSON stands for JavaScript Object Notation.

It is a lightweight data format used to exchange information between applications and APIs.

Example:

```json
{
  "userName": "alex.taylor",
  "department": "Finance",
  "active": true
}
```

Here:

- userName is a string.
- department is a string.
- active is a Boolean value.

## 5.3 JavaScript JSON parsing

JSON.parse() converts JSON text into a JavaScript value.

### Practical code

```javascript
const responseText =
'{"userName":"alex.taylor","active":true}';

const user = JSON.parse(responseText);

console.log(user.userName);
console.log(user.active);
```

### Expected output

```text
alex.taylor
true
```

### Explanation

1. responseText stores JSON as a string.
2. JSON.parse() converts it into a JavaScript object.
3. user.userName accesses the username.
4. user.active accesses the account status.

### Modify the example

Change:

```javascript
"userName": "alex.taylor"
```

To:

```javascript
"userName": "sam.lee"
```

Change:

```javascript
"active": true
```

To:

```javascript
"active": false
```

Run the code again and observe the output.

## 5.4 Python JSON parsing

Python provides the built-in json module.

### Practical code

```python
import json

response_text = '''
{
    "userName": "alex.taylor",
    "active": true
}
'''

user = json.loads(response_text)

print(user["userName"])
print(user["active"])
```

### Expected output

```text
alex.taylor
True
```

### Important difference

| JavaScript | Python |
|---|---|
| JSON.parse() | json.loads() |
| Object | Dictionary |
| console.log() | print() |
| user.userName | user["userName"] |

## 5.5 What is an HTTP Request and Response?

A request is a message sent by a client to a server.

A response is the server's reply.

Example:

```text
        CLIENT
           |
           | GET /Users
           v
        SERVER
           |
           | HTTP 200 OK
           v
      JSON RESPONSE
```

Common HTTP status codes:

| Status | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |

## 5.6 What is Base64URL?

Base64URL is a URL-safe variation of Base64 encoding.

It is commonly used in JWTs.

A JWT contains three parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example:

```text
eyJhbGciOiJIUzI1NiJ9
.
eyJzdWIiOiIxMjMiLCJyb2xlIjoiVmlld2VyIn0
.
SIGNATURE
```

The payload contains JSON claims encoded using Base64URL.

Important: Decoding a JWT payload does not verify the signature or prove that the token is valid.

## 5.7 Practical Demonstration: Decode a JWT Payload

Open JSFiddle and paste:

```javascript
const samplePayload =
  "eyJzdWIiOiIxMjMiLCJyb2xlIjoiVmlld2VyIn0";

// Convert Base64URL to Base64
const base64 = samplePayload
  .replace(/-/g, "+")
  .replace(/_/g, "/");

// Add required padding
const padded = base64 +
  "=".repeat((4 - base64.length % 4) % 4);

// Decode Base64
const jsonText = atob(padded);

// Convert JSON into JavaScript object
const payload = JSON.parse(jsonText);

console.log(payload);
console.log(payload.sub);
console.log(payload.role);
```

### Expected output

```text
{ sub: "123", role: "Viewer" }

123
Viewer
```

### Explain each step

1. The samplePayload stores the JWT payload.
2. replace() converts URL-safe Base64 characters.
3. Padding is added because Base64 requires the correct length.
4. atob() decodes the Base64 string.
5. JSON.parse() converts the JSON text into an object.
6. The program reads the subject and role.

**Security reminder:** This is only a payload-decoding demonstration. It does not validate the JWT signature, issuer, audience or expiration. Never paste real tokens into public editors.

---

# 6. Git and GitHub for IAM Engineers

## 6.1 What is Git?

Git is a version-control system that tracks changes in source code and configuration files.

It allows developers and IAM engineers to maintain project history, collaborate and restore previous versions.

GitHub is an online platform for hosting Git repositories and collaborating on projects.

## 6.2 Important Git terminology

| Term | Meaning |
|---|---|
| Repository | Project containing files and version history |
| Commit | Recorded snapshot of changes |
| Branch | Separate line of development |
| Merge | Combine changes from branches |
| Pull Request | Request to review and merge changes |
| .gitignore | Specifies files Git should ignore |

## 6.3 Git workflow diagram

```text
        CREATE PROJECT
              |
              v
          GIT INIT
              |
              v
        MODIFY FILES
              |
              v
         GIT ADD
              |
              v
        GIT COMMIT
              |
              v
       CREATE BRANCH
              |
              v
       PUSH TO GITHUB
              |
              v
       PULL REQUEST
              |
              v
        CODE REVIEW
              |
              v
           MERGE
```

## 6.4 Practical Demonstration: Git Commands

Open your terminal.

Initialize a repository:

```bash
git init
```

Check repository status:

```bash
git status
```

Add files:

```bash
git add .
```

Create a commit:

```bash
git commit -m "Initial IAM project"
```

Create a branch:

```bash
git branch feature/scim
```

Switch to the branch:

```bash
git switch feature/scim
```

Push the branch to GitHub:

```bash
git push -u origin feature/scim
```

After pushing, open GitHub and create a Pull Request.

## 6.5 What is secret hygiene?

Secret hygiene means protecting sensitive credentials from being exposed.

Never commit:

- Client secrets.
- API keys.
- Access tokens.
- Refresh tokens.
- Private keys.
- Database passwords.

Example `.gitignore`:

```gitignore
.env
.env.local
*.pem
*.key
secrets.json
node_modules/
```

For an IAM engineer, this is particularly important because leaked credentials can allow unauthorized access to identity systems.

### What to explain

"Git helps us track changes in IAM configurations and scripts. We use branches to work independently and pull requests to review changes before merging. We also use .gitignore to prevent sensitive files from being committed."

---

# 7. Containers and Docker

## 7.1 What is Docker?

Docker is a platform that allows us to package and run applications inside containers.

Imagine that an application requires:

- A particular programming language.
- Specific libraries.
- Configuration.
- Runtime dependencies.

Normally, configuring everything on another computer can take time.

Docker packages the application and its dependencies into an image, which can then be used to create containers.

## 7.2 What is a Docker Image?

A Docker image is a packaged template containing the application and its required dependencies.

It is used to create containers.

Example:

```text
nginx:latest
```

This is an example of an Nginx Docker image reference.

## 7.3 What is a Docker Container?

A container is a running instance of a Docker image.

For example, if we use an Nginx image to start a web server, Docker creates a container that runs that web server.

### Image vs Container

```text
          DOCKER IMAGE
          nginx:latest
                |
                |
           docker run
                |
                v
          DOCKER ENGINE
                |
          +-----+-----+
          |           |
          v           v
       Container 1  Container 2
          |           |
          v           v
        Web App     Web App
```

One image can create multiple containers.

## 7.4 Important Docker concepts

| Concept | Explanation |
|---|---|
| Image | Packaged application template |
| Container | Running instance of an image |
| Registry | Stores and distributes images |
| Port mapping | Connects host and container ports |
| Volume | Provides persistent storage |
| Environment variable | Passes runtime configuration |

## 7.5 Container vs Virtual Machine

| Container | Virtual Machine |
|---|---|
| Shares the host OS kernel | Runs a guest OS kernel |
| Lightweight | Generally heavier |
| Starts quickly | Generally takes longer to start |
| Packages application and dependencies | Includes a complete guest OS |
| Useful for disposable test services | Useful for full OS environments |

## 7.6 Docker architecture

```text
              USER
                |
                v
         DOCKER COMMAND
                |
                v
          DOCKER ENGINE
                |
          +-----+------+
          |            |
          v            v
       IMAGES       CONTAINERS
          |            |
          v            v
       nginx        Running App
       ubuntu       Running LDAP
       mysql        Running API
```

## 7.7 Important Docker commands

| Command | Purpose |
|---|---|
| docker pull nginx | Download an image |
| docker images | List local images |
| docker run | Create and start a container |
| docker ps | List running containers |
| docker ps -a | List all containers |
| docker logs | View container logs |
| docker stop | Stop a container |
| docker start | Start a stopped container |
| docker rm | Remove a container |

## 7.8 Practical Demonstration: Run Nginx

If Docker is installed on your computer, open PowerShell or CMD.

### Step 1: Download Nginx image

```bash
docker pull nginx
```

This downloads the Nginx image from Docker Hub.

### Step 2: Start a container

```bash
docker run -d --name web-demo -p 8080:80 nginx
```

Explanation:

- `docker run`: Creates and starts a container.
- `-d`: Runs it in the background.
- `--name web-demo`: Assigns a container name.
- `-p 8080:80`: Maps host port 8080 to container port 80.
- `nginx`: Specifies the image.

### Step 3: Check running containers

```bash
docker ps
```

You should see the web-demo container.

### Step 4: Open the browser

```text
http://localhost:8080
```

If everything works, you should see the Nginx welcome page.

### Step 5: Check container logs

```bash
docker logs web-demo
```

### Step 6: Stop the container

```bash
docker stop web-demo
```

### Step 7: Remove the container

```bash
docker rm web-demo
```

Remember:

- Stop means the container is no longer running.
- Remove means the container object is deleted.

A Docker image is not deleted just because you remove its container.

## 7.9 What is Port Mapping?

Port mapping connects a port on your host computer to a port inside a container.

Example:

```bash
-p 8080:80
```

```text
       YOUR COMPUTER
       localhost:8080
              |
              |
         Port Mapping
              |
              v
       DOCKER CONTAINER
            Port 80
              |
              v
          NGINX SERVER
```

Here, when you open localhost:8080, Docker forwards the traffic to port 80 inside the container.

## 7.10 What are Environment Variables?

Environment variables are configuration values passed to an application at runtime.

Example:

```bash
docker run -d \
  --name app-demo \
  -e APP_ENV=development \
  nginx
```

Here:

- `-e` specifies an environment variable.
- `APP_ENV` is the variable name.
- `development` is its value.

In IAM applications, environment variables may store configuration such as:

```text
IDP_URL
SCIM_BASE_URL
DATABASE_URL
```

Sensitive credentials must be handled using appropriate secrets management, not exposed through public configuration or logs.

## 7.11 Docker and IAM: Running an LDAP Test Environment

Docker can help IAM engineers create disposable test environments.

For example:

```text
             IAM ENGINEER
                  |
                  v
             DOCKER ENGINE
                  |
                  v
          LDAP TEST CONTAINER
                  |
                  v
            OPENLDAP SERVER
                  |
          +-------+-------+
          |               |
          v               v
        USERS           GROUPS
          |               |
          +-------+-------+
                  |
                  v
             LDAP SEARCH
                  |
                  v
           IAM TEST CLIENT
```

We can use this environment to practise:

- Creating test users.
- Creating groups.
- Searching LDAP entries.
- Testing directory connections.
- Understanding user attributes.

This is a test environment, not a production directory.

Do not expose LDAP ports or administration interfaces publicly.

---

# 8. Integrated IAM Practical Scenario

## Scenario: A new employee joins a company

Suppose Alex Taylor joins the Finance department.

The company uses:

- PingFederate as its Identity Provider.
- A web-based Finance application.
- SCIM for provisioning.
- GitHub for maintaining IAM scripts.
- Docker for test environments.

### Complete workflow

```text
          ALEX JOINS COMPANY
                  |
                  v
            HR SYSTEM
                  |
                  v
           IDENTITY PROVIDER
                  |
          +-------+-------+
          |               |
          v               v
      AUTHENTICATION   PROVISIONING
          |               |
          v               v
       OAuth/SAML        SCIM API
          |               |
          v               v
       Browser         SaaS Finance App
          |               |
          v               v
      Application      Alex Account
        Session           Created
          |               |
          +-------+-------+
                  |
                  v
          IAM MANAGEMENT
                  |
          +-------+-------+
          |               |
          v               v
       Git/GitHub        Docker
       Configuration    Test Lab
```

### Step-by-step explanation

**Step 1: Authentication**

Alex opens the company portal and is redirected to PingFederate for authentication.

**Step 2: Application architecture**

The IAM team checks whether the application is a SPA or a server-side application.

If it is a SPA using OAuth, it is a public client and should use Authorization Code Flow with PKCE.

**Step 3: Provisioning**

The Identity Provider sends a SCIM POST request to:

```http
POST /Users
```

This creates Alex's account in the Finance application.

**Step 4: JSON data**

The SCIM request contains attributes such as:

```json
{
  "userName": "alex.taylor@example.test",
  "active": true,
  "department": "Finance"
}
```

The department attribute requires the appropriate supported extension or application schema.

**Step 5: GitHub**

The IAM team maintains provisioning scripts and configuration in Git repositories.

Changes are reviewed through pull requests before merging.

**Step 6: Docker**

The team uses Docker to create a disposable LDAP environment and test directory integration without affecting production.

---

# 9. Important Interview Questions and Answers

### Q1. What is SAML HTTP-POST binding?

It is a SAML communication method in which the Identity Provider sends a SAML response through an HTML form using HTTP POST to the Service Provider's ACS endpoint.

### Q2. What is an ACS endpoint?

ACS stands for Assertion Consumer Service. It is the Service Provider endpoint that receives and processes SAML responses.

### Q3. What is the difference between SPA and server-side applications?

A SPA executes most of its frontend logic in the browser, whereas a traditional server-side application handles much of its application logic on the backend server.

### Q4. Why is a SPA a public OAuth client?

Because its JavaScript code and configuration are available to the browser user, it cannot securely protect a client secret.

### Q5. What is PKCE?

PKCE is a security extension for OAuth 2.0 that uses a code verifier and code challenge to protect the authorization code exchange.

### Q6. What is SCIM?

SCIM stands for System for Cross-domain Identity Management. It standardizes automated user and group provisioning between identity systems and applications.

### Q7. What is the difference between authentication and provisioning?

Authentication verifies who a user is. Provisioning creates, updates, disables or removes the user's account in an application.

### Q8. What is the difference between PUT and PATCH?

PUT replaces a resource representation, whereas PATCH modifies selected attributes.

### Q9. What is JSON.parse()?

JSON.parse() converts JSON-formatted text into a JavaScript value.

### Q10. Does decoding a JWT validate it?

No. Decoding only reveals its encoded contents. Signature and claims validation are still required.

### Q11. What is Git?

Git is a version-control system used to track changes in source code and configuration files.

### Q12. What is a Pull Request?

A Pull Request is a request to review and merge changes from one branch into another.

### Q13. What is Docker?

Docker is a platform used to package and run applications in containers.

### Q14. What is the difference between an image and a container?

An image is the packaged template, while a container is an instance created from that image.

### Q15. What is port mapping?

Port mapping connects a port on the host computer to a port inside a Docker container.

---

# 10. Final Revision: Important Commands and Code

| Topic | Important command or code |
|---|---|
| HTML POST | `<form method="POST">` |
| SAML | `SAMLResponse` |
| ACS | `/saml/acs` |
| PKCE | `crypto.subtle.digest("SHA-256", data)` |
| SCIM Create | `POST /Users` |
| SCIM Read | `GET /Users` |
| SCIM Update | `PATCH /Users/{id}` |
| JavaScript JSON | `JSON.parse()` |
| Python JSON | `json.loads()` |
| JWT | `header.payload.signature` |
| Git | `git commit` |
| Git branch | `git switch` |
| Docker | `docker run` |
| Docker status | `docker ps` |
| Docker logs | `docker logs` |
| Docker stop | `docker stop` |
| Docker remove | `docker rm` |

---

# 11. YouTube Presentation Structure

Suggested video duration: 30–40 minutes.

| Time | Topic | Demonstration |
|---|---|---|
| 00:00–02:00 | Introduction to Pillar 6 | Explain the learning objectives |
| 02:00–08:00 | HTML Form POST and SAML | JSFiddle + HTTPBin |
| 08:00–14:00 | SPA vs Server-Side and PKCE | Architecture diagram + JavaScript |
| 14:00–22:00 | SCIM 2.0 | SCIM Playground |
| 22:00–27:00 | JavaScript and Python | JSON parsing + JWT payload |
| 27:00–30:00 | Git and GitHub | Git commands |
| 30:00–36:00 | Docker | Run Nginx container |
| 36:00–40:00 | Integrated IAM scenario | Explain complete workflow |

## Suggested opening speech

"Hello everyone. Today I am going to explain Pillar 6, which covers Developer Foundations, SCIM and Hands-on Tooling.

The objective of this pillar is to understand the development concepts and tools that an IAM engineer commonly works with.

I will cover HTML Form POST and its relationship with SAML, SPA versus server-side applications, OAuth PKCE, SCIM provisioning, basic scripting, Git and Docker.

Along with the theoretical concepts, I will also demonstrate practical examples using browser-based tools and Docker."

## Suggested closing speech

"To conclude, Pillar 6 helps us understand how application development concepts connect with Identity and Access Management.

We learned how the browser transports SAML responses, why public OAuth clients require PKCE, how SCIM automates user provisioning, how scripts process identity information, how Git manages configuration changes and how Docker provides repeatable test environments.

These concepts are useful for an IAM engineer because identity management is not limited to authentication. It also involves application integration, provisioning, automation, configuration management and secure testing environments.

Thank you."

---

# 12. Practical Demonstration Checklist

Before recording your YouTube video, prepare the following:

- [ ] JSFiddle open with the HTML POST example.
- [ ] HTTPBin ready to receive the sample form.
- [ ] PKCE JavaScript example ready.
- [ ] SCIM Playground open.
- [ ] Sample SCIM User JSON ready.
- [ ] JavaScript JSON parsing example ready.
- [ ] Python JSON parsing example ready.
- [ ] JWT payload decoding example ready.
- [ ] Git terminal ready.
- [ ] Docker installed and working, if performing the local demonstration.
- [ ] Nginx container demonstration tested.
- [ ] All diagrams and examples prepared.

**Final security reminders:**

- Use dummy identities and sample data.
- Never expose actual passwords, client secrets, private keys or tokens.
- Do not use production SAML responses in public tools.
- Do not expose test LDAP or SCIM services to the public internet.
- Remember that Base64 is encoding, not encryption.
- Remember that JWT decoding is not JWT validation.

**Final objective:** You should be able to explain what each technology does, why IAM engineers use it, how it works internally at a beginner-to-intermediate level, and demonstrate a small practical example.