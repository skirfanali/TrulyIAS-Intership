# Workbook 4 – Complete IAM Notes (Modules 1–10)

**Topic:** Identity Directories, Enterprise Systems & Scripting for IAM  
**Level:** Beginner / Level 0 Engineering Foundation  
**Purpose:** Concept learning, practical labs, technical interviews and revision

---

# Module 1: Active Directory (AD) Core Architecture & Domain Services

## 1.1 What is Active Directory?

Active Directory (AD) is a directory service developed by Microsoft. It centrally manages users, computers, groups and other resources in an organisation.

**Simple example:** Imagine a company has 500 employees. Instead of creating separate login accounts on every computer, the company uses Active Directory to manage employee accounts centrally.

When an employee logs in, Active Directory helps verify their identity and determine which resources they can access.

### Main functions

1. User account management.
2. Computer account management.
3. Authentication – verifying who the user is.
4. Authorization – deciding what the user can access.
5. Group management.
6. Centralised security policies.
7. Directory information storage and lookup.

## 1.2 Active Directory hierarchy

The workbook describes the hierarchy as:

**Forest → Trees → Domains → Organizational Units (OUs) → Objects**

### A. Forest

A forest is the highest-level logical structure in Active Directory.

- It can contain one or more domain trees.
- Domains in a forest share a common schema and configuration.
- It provides the overall security boundary for the AD forest.

Example: `globalcompany.com`

### B. Tree

A tree is a collection of related domains that share a contiguous DNS namespace.

Example:

- `corp.globalcompany.com`
- `us.corp.globalcompany.com`
- `eu.corp.globalcompany.com`

### C. Domain

A domain is a logical administrative unit that contains users, computers, groups and other directory objects.

Example: `corp.globalcompany.com`

A company can use domains to organise administration and manage identities.

### D. Organizational Unit (OU)

An OU is a container used to organise objects inside a domain.

Example:

- OU=Engineering
- OU=Finance
- OU=Sales

OUs help administrators apply Group Policy and delegate administrative responsibilities.

### E. Objects

Objects are the actual resources stored in Active Directory.

Examples:

- User: John Doe
- Computer: PC-101
- Group: AWS-Admins
- Printer: Office-Printer

### Architecture diagram

```text
                 FOREST
          globalcompany.com
                   |
          DOMAIN TREE / ROOT
        corp.globalcompany.com
                   |
          +--------+--------+
          |                 |
      US DOMAIN         EU DOMAIN
          |                 |
       OU=USA            OU=Europe
          |                 |
    +-----+-----+      +----+----+
    |           |      |         |
Engineering  Finance Engineering Sales
    |           |      |         |
  Users       Users  Users     Users
```

## 1.3 Domain Controller (DC)

A Domain Controller is a Windows Server running Active Directory Domain Services (AD DS).

It is responsible for:

- Authenticating users.
- Providing directory information.
- Processing directory queries.
- Applying relevant domain policies.
- Replicating directory changes with other domain controllers.

**Real-world example:** John enters his company username and password on his office laptop. The laptop communicates with a Domain Controller to authenticate him.

## 1.4 Global Catalog (GC)

The Global Catalog contains a searchable, partial representation of objects across the forest.

It helps users and applications find directory objects across different domains.

Ports:

- TCP 3268 – Global Catalog.
- TCP 3269 – Secure Global Catalog.

**Example:** An application needs to locate an employee who belongs to another domain. The Global Catalog can help with cross-domain searches.

## 1.5 Important Active Directory attributes

Attributes are properties that describe an AD object.

| Attribute | Example | Purpose |
|---|---|---|
| `sAMAccountName` | `jdoe` | Legacy Windows login name |
| `userPrincipalName` | `john.doe@company.com` | User login name in UPN format |
| `objectGUID` | `a8f3b2c1-...` | Unique identifier of an AD object |
| `memberOf` | `AWS-Admins` | Direct group membership information |

### sAMAccountName

- Traditional Windows login name.
- Used in the format `DOMAIN\username`.
- Example: `CORP\jdoe`.

### userPrincipalName (UPN)

- Commonly uses an email-like format.
- Example: `john.doe@company.com`.
- Often used for modern sign-in experiences.
- Can be mapped to a SAML NameID by an Identity Provider.

### objectGUID

- A unique identifier assigned to an AD object.
- It remains stable during the lifetime of that object.
- Changing the user's name does not normally change the GUID.
- Deleting and recreating an account produces a new objectGUID.

**IAM importance:** Synchronisation systems can use objectGUID as a SourceAnchor to identify the same directory object even when its username changes.

### memberOf

- A multi-valued attribute representing direct group memberships.
- Example: `CN=AWS-Admins,OU=Groups,DC=corp,DC=com`.
- Used in identity and authorization-related processes.

**Important:** Do not assume that `memberOf` automatically lists every nested or indirect group membership.

## 1.6 Active Directory and IAM

IAM systems can connect to AD to retrieve employee identities and group information.

Example flow:

```text
Employee
   |
   v
Active Directory
   |
   | User details and group membership
   v
Identity Provider (IdP)
   |
   | SAML assertion / OIDC token
   v
Cloud Application
   |
   v
Access based on assigned permissions
```

**Remember:** AD stores directory identities. An Identity Provider can use those identities to authenticate users and issue federation tokens.

---

# Module 2: LDAP Protocol, Directory Schemas & Binds

## 2.1 What is LDAP?

LDAP stands for **Lightweight Directory Access Protocol**.

It is a protocol used to access and manage directory information over a network.

LDAP is commonly used by IAM systems to:

- Search for users.
- Read user attributes.
- Retrieve group information.
- Authenticate using directory credentials.
- Connect enterprise applications to directories.

**Real-world example:** An organisation uses Active Directory to store employee accounts. Its Identity Provider connects to AD using LDAP to search for an employee during login.

## 2.2 LDAP ports

| Port | Protocol | Meaning |
|---|---|---|
| 389 | LDAP | Standard LDAP connection |
| 636 | LDAPS | LDAP over TLS |

**Important:** LDAP on port 389 can support StartTLS, which upgrades the connection to TLS. LDAPS commonly uses TLS from the beginning of the connection.

## 2.3 Directory Information Tree (DIT)

LDAP stores information in a hierarchical structure called the Directory Information Tree.

Example:

```text
dc=company,dc=com
       |
   ou=Employees
       |
   +---+---+
   |       |
  uid=jdoe uid=jsmith
   |       |
  User    User
```

### Important LDAP terms

| Term | Meaning |
|---|---|
| DIT | Directory Information Tree |
| DN | Distinguished Name |
| CN | Common Name |
| OU | Organizational Unit |
| DC | Domain Component |
| UID | User identifier |

Example DN:

`cn=John Doe,ou=Engineering,dc=company,dc=com`

This identifies the John Doe entry in the Engineering OU.

## 2.4 What is LDAP Bind?

A Bind is an LDAP operation used to establish an authenticated or unauthenticated LDAP session.

Common bind types:

**Anonymous Bind**
- No username or password is supplied.
- Access depends on the directory's anonymous access policy.
- Generally restricted in production environments.

**Simple Bind**
- Uses a Distinguished Name and password.
- Credentials must be protected with TLS because simple bind does not itself encrypt the password.

**SASL Bind**
- Uses a SASL authentication mechanism.
- Can support different authentication methods, depending on server configuration.

## 2.5 LDAP Search

An LDAP search retrieves directory entries based on a search base, scope and filter.

Example command from the workbook:

```bash
ldapsearch -x \
-H ldap://ldap.forumsys.com:389 \
-D "cn=read-only-admin,dc=example,dc=com" \
-w "password" \
-b "dc=example,dc=com" \
-s sub "(uid=einstein)"
```

### Explanation of the command

| Option | Meaning |
|---|---|
| `ldapsearch` | LDAP search command |
| `-x` | Use simple authentication |
| `-H` | LDAP server URL |
| `-D` | Bind Distinguished Name |
| `-w` | Password |
| `-b` | Base DN |
| `-s sub` | Subtree search scope |
| `(uid=einstein)` | Search filter |

The workbook's public test example returns an entry for Albert Einstein, including attributes such as email and common name.

**Lab note:** This is a public training environment, not a production directory. Do not use real credentials or sensitive information in public test services.

## 2.6 Three LDAP Search Scopes

This is a very important interview topic.

### 1. Base Object

Searches only the exact entry specified by the Base DN.

Example:

```text
Base DN: uid=jdoe,ou=Engineering,dc=company,dc=com
Scope: Base
```

Use case: Retrieve the attributes of one known user entry.

### 2. OneLevel

Searches only the immediate children of the Base DN.

It does not search deeper nested OUs.

Example:

```text
ou=Employees
   |
   +--- uid=jdoe
   |
   +--- ou=Engineering
             |
             +--- uid=jsmith
```

A OneLevel search under `ou=Employees` does not search inside the nested Engineering OU.

### 3. Subtree

Searches the Base DN and all its descendants recursively.

```text
ou=Employees
   |
   +--- uid=jdoe
   |
   +--- ou=Engineering
             |
             +--- uid=jsmith
```

A Subtree search can find both users.

**IAM best practice:** Subtree is commonly used when an IAM system must find users across nested OUs.

---

# Module 3: Advanced LDAP Search Filters & Query Construction

## 3.1 What is an LDAP filter?

An LDAP filter specifies which directory entries should be returned by a search.

Example:

```ldap
(uid=jdoe)
```

This searches for an entry whose UID is `jdoe`.

LDAP uses prefix notation, also known as Polish notation. The operator appears before its conditions.

## 3.2 Important LDAP operators

| Operator | Meaning | Example |
|---|---|---|
| `&` | AND | `(&(A)(B))` |
| `|` | OR | `(|(A)(B))` |
| `!` | NOT | `(!(A))` |
| `*` | Wildcard | `(mail=*)` |
| `>=` | Greater than or equal | `(age>=18)` |
| `<=` | Less than or equal | `(age<=60)` |

## 3.3 Filter 1: Active users in Engineering or DevOps

Workbook filter:

```ldap
(&(objectClass=person)(mail=*)(|(department=Engineering)(department=DevOps)))
```

### Explanation

- `objectClass=person`: Select person objects.
- `mail=*`: The mail attribute must exist.
- `|`: OR condition.
- `department=Engineering`: Department is Engineering.
- `department=DevOps`: Department is DevOps.
- `&`: All the top-level conditions must be satisfied.

In simple words:

Find person objects that have an email address and belong to either Engineering or DevOps.

**Note:** This filter does not itself check whether an AD account is disabled. It only checks the listed conditions.

## 3.4 Filter 2: Excluding disabled AD accounts

Workbook filter:

```ldap
(&(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
```

### Explanation

- `objectClass=user`: Search user objects.
- `!`: NOT condition.
- `userAccountControl`: AD account-control attribute.
- `1.2.840.113556.1.4.803`: AD bitwise AND matching rule.
- `2`: The disabled-account flag.

This filter excludes accounts where the disabled bit is set.

**Real-world IAM use case:** When an employee leaves a company, their AD account may be disabled. An IAM system can exclude disabled accounts from an authentication or synchronisation search.

## 3.5 Filter 3: Username OR email login

Workbook filter:

```ldap
(&(objectClass=user)(|(sAMAccountName=${username})(userPrincipalName=${username})))
```

### Explanation

The filter checks whether the supplied username matches:

- `sAMAccountName`, OR
- `userPrincipalName`.

Example:

The user enters `jdoe`.

The system checks the legacy username.

If the user enters `john.doe@company.com`, the system checks the UPN.

**Important:** `${username}` is a placeholder in the workbook example. The application must safely substitute the user input and escape LDAP filter special characters. Never build LDAP filters by blindly concatenating untrusted input.

## 3.6 LDAP filter practical revision

| Requirement | Filter idea |
|---|---|
| Find a particular user | `(uid=jdoe)` |
| Find users with email | `(mail=*)` |
| Department is Engineering | `(department=Engineering)` |
| Engineering OR DevOps | `(|(department=Engineering)(department=DevOps))` |
| Exclude a condition | `(!(condition))` |
| Combine all conditions | `(&(condition1)(condition2))` |

**Interview tip:** Be ready to explain AND, OR, NOT, wildcard searches and the AD disabled-account filter.

---

# Module 4: Linux OS Administration for Identity Servers

## 4.1 Why Linux is important in IAM

Enterprise Identity Providers and IAM engines can run on Linux or Windows servers.

Examples of enterprise identity platforms mentioned in the workbook:

- PingFederate
- SailPoint
- ForgeRock

The workbook gives `/opt/pingfederate` as a standard example installation location.

## 4.2 Linux file system basics

Linux uses a hierarchical file system.

| Directory | Purpose |
|---|---|
| `/` | Root directory |
| `/opt` | Optional application software |
| `/var` | Variable data such as logs |
| `/etc` | Configuration files |
| `/home` | User home directories |
| `/tmp` | Temporary files |
| `/var/log` | System and application logs |

Example:

```bash
cd /opt/pingfederate
```

This changes the current directory to the PingFederate installation directory.

## 4.3 Root user vs normal user

### Root user

- Has highly privileged access.
- Can modify system files and configurations.
- Can control most operating-system resources.

### Normal user

- Has limited permissions.
- Can perform only permitted operations.

### Why must IAM servers not run as root?

The workbook highlights the principle of least privilege.

If an attacker exploits an application running with root privileges, the attacker may gain extensive control over the server.

Therefore, an identity service should run under a dedicated, unprivileged service account.

Example from the workbook:

```text
User: pingfed
Group: pingfed
```

This is a security practice, not a guarantee that the service account is automatically safe. Its permissions should be restricted to what the service needs.

## 4.4 What is a daemon?

A daemon is a background service or process that runs without requiring continuous user interaction.

Examples:

- Web server
- Database service
- Identity Provider service

## 4.5 What is systemd?

`systemd` is a service manager used by many Linux distributions.

It manages system services, including starting, stopping and monitoring them.

### Important systemctl commands

**Start a service**

```bash
sudo systemctl start pingfederate
```

Starts the PingFederate service.

**Check service status**

```bash
sudo systemctl status pingfederate
```

Displays whether the service is running and its current status.

**Stop a service**

```bash
sudo systemctl stop pingfederate
```

Stops the service.

**Restart a service**

```bash
sudo systemctl restart pingfederate
```

Restarts the service.

**Enable automatic startup**

```bash
sudo systemctl enable pingfederate
```

Configures the service to start automatically during system boot.

**View live service logs**

```bash
sudo journalctl -u pingfederate -f
```

Displays new log entries as they arrive.

### Real-world scenario

An employee reports that the company SSO login is unavailable.

An IAM engineer checks:

```bash
sudo systemctl status pingfederate
```

If the service is stopped, the engineer investigates the reason and starts it if appropriate.

---

# Module 5: Enterprise Server Log Triage & Text Processing

## 5.1 What is log triage?

Log triage means checking application and system logs to identify the cause of an error or failure.

IAM engineers use logs to investigate:

- Failed login attempts.
- LDAP connection failures.
- SAML authentication errors.
- Service startup problems.
- Network timeouts.
- Token and session issues.

The workbook identifies three example logs:

| Log | General purpose |
|---|---|
| `server.log` | Application and server activity |
| `audit.log` | Security and administrative audit events |
| `transaction.log` | Transaction-related events |

Exact log contents and filenames depend on the product and configuration.

## 5.2 Important Linux commands for log analysis

### View a complete file

```bash
cat server.log
```

Displays the file contents.

### View the last lines

```bash
tail server.log
```

Displays the end of the file.

### Follow logs in real time

```bash
tail -f server.log
```

Shows new lines as they are written.

### Search for a keyword

```bash
grep "ERROR" server.log
```

Finds lines containing `ERROR`.

### Case-insensitive search

```bash
grep -i "robert.smith" server.log
```

Finds the username regardless of letter case.

### Search with line numbers

```bash
grep -n "ERROR" server.log
```

Shows matching lines with their line numbers.

## 5.3 Understanding a Transaction ID (TID)

A Transaction ID is an identifier that helps connect related log events for a particular operation.

Example:

```text
tid:TRX-987214
```

It allows an engineer to trace the stages of a login request.

## 5.4 Workbook Lab 5B: Tracking a transaction

### Scenario

Robert Smith reports that he cannot log in to Salesforce.

**Step 1: Search for the user**

```bash
grep -i "robert.smith" /var/log/pingfederate/server.log
```

Example output:

```text
tid:TRX-987214
[SSO] User=robert.smith
Initiating SAML SP Connection
```

The engineer finds the Transaction ID:

```text
TRX-987214
```

**Step 2: Search using the Transaction ID**

```bash
grep "TRX-987214" /var/log/pingfederate/server.log
```

Example output:

```text
CommunicationException:
ldap.corp.com:636 connection timed out!
```

### What is the actual problem?

The error indicates a connection timeout to the LDAP server over port 636.

Port 636 is commonly used for LDAPS.

This is a connectivity problem, not necessarily an incorrect user password.

### Troubleshooting flow

```text
User reports login failure
          |
          v
Search username in server.log
          |
          v
Find Transaction ID
          |
          v
Search using Transaction ID
          |
          v
Find LDAP timeout on port 636
          |
          v
Check network, firewall and LDAP server
          |
          v
Restore connectivity and retest
```

## 5.5 Useful additional commands

```bash
grep -i "error" server.log
grep -i "exception" server.log
tail -f server.log
less server.log
```

**Interview tip:** Explain why tracing a Transaction ID is more reliable than searching only for a username: one user may have many transactions, while a transaction identifier helps isolate one specific operation.

---

# Module 6: Shell Scripting (Bash) for IAM Automation

## 6.1 What is Bash?

Bash stands for **Bourne Again Shell**.

It is a command-line shell commonly used on Linux systems.

Bash scripting allows engineers to automate repetitive tasks.

Examples in IAM:

- Checking whether an Identity Provider is available.
- Monitoring endpoints.
- Running scheduled health checks.
- Automating maintenance tasks.
- Generating operational alerts.

## 6.2 What is a shell script?

A shell script is a file containing commands that execute in sequence.

Example:

```bash
#!/bin/bash

echo "Hello IAM Engineer"
```

The first line is called a shebang. It specifies the interpreter used to run the script.

## 6.3 Important Bash concepts

### Variable

A variable stores a value.

```bash
NAME="PingFederate"
echo "$NAME"
```

Output:

```text
PingFederate
```

### Command substitution

Stores the output of a command in a variable.

```bash
STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://example.com)
```

### Conditional statement

```bash
if [ "$STATUS" -eq 200 ]; then
    echo "Success"
else
    echo "Failed"
fi
```

### Exit status

`$?` stores the exit status of the last executed command.

- `0` means success.
- A non-zero value indicates failure.

Example:

```bash
ls
echo $?
```

## 6.4 Workbook Lab 6A: Automated Identity Endpoint Health Check

### Purpose

Check whether an Identity Provider's OpenID Connect discovery endpoint is responding successfully.

Workbook script:

```bash
#!/bin/bash

IDP_URL="https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"

HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
  --connect-timeout 5 "$IDP_URL")

if [ "$HTTP_STATUS" -eq 200 ]; then
    echo "[SUCCESS] IDP is healthy and responding (HTTP 200 OK)."
    exit 0
else
    echo "[CRITICAL ALERT] IDP check failed! HTTP Status: $HTTP_STATUS"
    exit 1
fi
```

### Explanation

**Step 1: Define the endpoint**

```bash
IDP_URL="https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration"
```

Stores the Identity Provider discovery URL.

**Step 2: Send the HTTP request**

```bash
curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "$IDP_URL"
```

- `curl`: Sends an HTTP request.
- `-s`: Silent mode.
- `-o /dev/null`: Discards the response body.
- `-w "%{http_code}"`: Prints the HTTP status code.
- `--connect-timeout 5`: Limits the connection-establishment time to five seconds.

**Step 3: Check the response**

```bash
if [ "$HTTP_STATUS" -eq 200 ]; then
```

Checks whether the HTTP status is 200.

**Step 4: Return a result**

```bash
exit 0
```

Indicates successful script execution.

```bash
exit 1
```

Indicates failure.

### How to run the script

Save it as:

```text
idp_health_check.sh
```

Give it execute permission:

```bash
chmod +x idp_health_check.sh
```

Run it:

```bash
./idp_health_check.sh
```

**Important:** HTTP 200 confirms that the endpoint responded successfully. It does not prove that every authentication flow, user login or backend dependency is healthy.

---

# Module 7: Windows PowerShell for Identity & Active Directory

## 7.1 What is PowerShell?

PowerShell is a command-line shell and scripting language developed by Microsoft.

It is widely used for Windows administration, automation and Active Directory management.

### Bash vs PowerShell

| Bash | PowerShell |
|---|---|
| Common on Linux | Common on Windows |
| Uses shell commands and text streams | Works with structured .NET objects |
| Example: `grep` | Example: `Where-Object` |
| Script extension: `.sh` | Script extension: `.ps1` |

## 7.2 What is an API?

API stands for **Application Programming Interface**.

An API allows one application to communicate with another application.

Example:

An IAM audit script requests user information from an API and exports the information into a CSV file.

## 7.3 What is REST API?

REST stands for Representational State Transfer.

A REST API commonly uses HTTP methods such as:

| Method | Purpose |
|---|---|
| GET | Retrieve information |
| POST | Create or submit information |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

## 7.4 What is JSON?

JSON stands for JavaScript Object Notation.

It is a data format commonly used by REST APIs.

Example:

```json
{
  "id": 1,
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe"
}
```

## 7.5 What is Invoke-RestMethod?

`Invoke-RestMethod` is a PowerShell command used to send HTTP and REST requests.

It can automatically convert JSON responses into PowerShell objects.

## 7.6 Workbook Lab 7B: REST API Identity Audit

Workbook example:

```powershell
$apiUrl = "https://reqres.in/api/users?page=1"

$response = Invoke-RestMethod -Uri $apiUrl -Method Get

$response.data |
    Select-Object id, email, first_name, last_name |
    Format-Table
```

### Explanation

**Step 1: Store API URL**

```powershell
$apiUrl = "https://reqres.in/api/users?page=1"
```

Stores the API endpoint.

**Step 2: Send GET request**

```powershell
$response = Invoke-RestMethod -Uri $apiUrl -Method Get
```

Retrieves data from the API.

**Step 3: Select required attributes**

```powershell
$response.data |
    Select-Object id, email, first_name, last_name
```

Selects the required fields.

**Step 4: Display as a table**

```powershell
Format-Table
```

Displays the selected data in a readable format.

## 7.7 Export the audit report to CSV

Workbook command:

```powershell
$response.data |
    Select-Object id, email, first_name, last_name |
    Export-Csv -Path "$HOME\identity_audit.csv" -NoTypeInformation
```

### What is CSV?

CSV stands for Comma-Separated Values.

It stores tabular data in a text-based format that can be opened in Excel.

Example:

```csv
id,email,first_name,last_name
1,john@example.com,John,Doe
```

### What is the PowerShell pipeline?

The pipeline symbol is:

```powershell
|
```

It passes objects from one command to another.

Example:

```powershell
Get-Process | Select-Object ProcessName
```

PowerShell passes structured objects rather than merely passing formatted text between commands.

**IAM use case:** An engineer can retrieve user information, select required attributes and export an identity audit report.

**Note:** The workbook uses ReqRes as a practice REST API. It is not a real Active Directory server.

---

# Module 8: Version Control – Git Fundamentals for Identity-as-Code

## 8.1 What is Git?

Git is a distributed version-control system.

It tracks changes to files and allows developers and engineers to manage different versions of a project.

In IAM, configuration files and automation scripts may be stored in Git repositories.

Examples:

- Terraform files.
- YAML configuration.
- JSON configuration.
- Automation scripts.
- Infrastructure definitions.

## 8.2 What is GitHub?

GitHub is an online platform for hosting and collaborating on Git repositories.

### Git vs GitHub

| Git | GitHub |
|---|---|
| Version-control tool | Online repository hosting platform |
| Runs locally | Runs online |
| Tracks file changes | Supports collaboration and remote storage |

## 8.3 Important Git commands

**Check Git version**

```bash
git --version
```

**Initialize repository**

```bash
git init
```

**Check status**

```bash
git status
```

**Add files**

```bash
git add .
```

**Commit changes**

```bash
git commit -m "Initial IAM configuration"
```

**Connect remote repository**

```bash
git remote add origin https://github.com/username/repository.git
```

**Push changes**

```bash
git push -u origin main
```

## 8.4 What is `.gitignore`?

`.gitignore` is a file that tells Git which untracked files and directories it should ignore.

It is especially important in IAM because repositories may contain sensitive files.

Examples:

- Private keys.
- Certificates containing private keys.
- Password files.
- Secret configuration files.
- Environment files.

## 8.5 Workbook Lab 8B: Prevent private key leaks

Workbook `.gitignore` example:

```gitignore
*.key
*.pem
*.p12
*.pfx
secrets.json
vault_password.txt
```

### Explanation

| Pattern | Purpose |
|---|---|
| `*.key` | Ignore files ending in `.key` |
| `*.pem` | Ignore PEM files |
| `*.p12` | Ignore PKCS#12 files |
| `*.pfx` | Ignore PFX files |
| `secrets.json` | Ignore a specific secret file |
| `vault_password.txt` | Ignore a password file |

### Why is this important in IAM?

Private keys and credentials must remain confidential.

If an attacker obtains a private key, they may be able to impersonate a system or use the key in an attack, depending on its purpose and protections.

### Important security warning

`.gitignore` does not automatically remove files that have already been committed or tracked by Git.

If a secret has already been pushed:

1. Treat it as exposed.
2. Revoke or rotate the affected key or credential.
3. Remove the secret from the repository history where appropriate.
4. Review access and investigate potential misuse.

**Interview question:** Why should private keys never be committed to a public GitHub repository?

**Answer:** Because an exposed private key can allow an attacker to impersonate an identity, decrypt protected information in some scenarios, or compromise systems that trust that key.

---

# Module 9: Containers & Sandbox Environments – Docker for IAM

## 9.1 What is Docker?

Docker is a platform used to build and run applications inside containers.

A container packages an application and its required dependencies so that it can run in a consistent environment.

**Simple example:** Instead of manually installing and configuring an LDAP server on your computer, you can run an LDAP server using a Docker container.

## 9.2 What is a container?

A container is an isolated running environment for an application.

It shares the host operating system's kernel but keeps the application and its dependencies separated from other processes.

## 9.3 What is an image?

A Docker image is a template used to create containers.

Example:

```text
osixia/openldap:latest
```

This is the image used by the workbook's OpenLDAP lab.

## 9.4 Image vs Container

| Image | Container |
|---|---|
| Template | Running instance of an image |
| Used to create containers | Runs the application |
| Example: OpenLDAP image | Example: local-ldap container |

## 9.5 Important Docker commands

**Check Docker version**

```bash
docker --version
```

**List running containers**

```bash
docker ps
```

**List all containers**

```bash
docker ps -a
```

**List images**

```bash
docker images
```

**Stop a container**

```bash
docker stop local-ldap
```

**Remove a container**

```bash
docker rm local-ldap
```

## 9.6 Workbook Lab 9A: Run a local OpenLDAP container

Workbook command:

```bash
docker run -d \
  --name local-ldap \
  -p 389:389 \
  -p 636:636 \
  --env LDAP_ORGANISATION="TrainingCorp" \
  --env LDAP_DOMAIN="training.local" \
  --env LDAP_ADMIN_PASSWORD="AdminPassword123!" \
  osixia/openldap:latest
```

### Explanation

| Command / option | Meaning |
|---|---|
| `docker run` | Creates and starts a container |
| `-d` | Runs in detached/background mode |
| `--name local-ldap` | Sets the container name |
| `-p 389:389` | Maps host port 389 to container port 389 |
| `-p 636:636` | Maps host port 636 to container port 636 |
| `--env` | Sets an environment variable |
| `LDAP_ORGANISATION` | Sets organisation name |
| `LDAP_DOMAIN` | Sets LDAP domain |
| `LDAP_ADMIN_PASSWORD` | Sets administrator password |
| `osixia/openldap:latest` | Docker image |

### What happens after running the command?

```text
               Your Computer
                    |
                    v
             Docker Engine
                    |
                    v
             local-ldap
                    |
              OpenLDAP Server
                    |
          +---------+---------+
          |                   |
       Port 389            Port 636
        LDAP                LDAPS
```

Your computer can communicate with the container through the mapped ports, provided the service is running and configured to listen on those ports.

### Verify the container

```bash
docker ps
```

### Stop and remove the container

```bash
docker stop local-ldap
docker rm local-ldap
```

The workbook combines these commands as:

```bash
docker stop local-ldap && docker rm local-ldap
```

### Important security note

The password in the workbook is a lab example only. Do not use it in production.

For a production environment:

- Use strong secrets.
- Avoid exposing directory ports unnecessarily.
- Configure TLS certificates properly.
- Restrict network access.
- Use secure secret management.
- Avoid relying on an unpinned `latest` image tag for controlled deployments.

---

# Module 10: SCIM 2.0 User Provisioning Protocol Fundamentals

## 10.1 What is SCIM?

SCIM stands for **System for Cross-domain Identity Management**.

SCIM 2.0 is a standard protocol used to automate the exchange and management of identity information between systems.

It is defined by RFC 7644.

SCIM commonly uses REST APIs and JSON.

### Why do we need SCIM?

Imagine a company uses:

- Active Directory for employee identities.
- An Identity Provider for authentication.
- Salesforce for CRM.
- Slack for communication.
- Other cloud applications for business operations.

When a new employee joins, their accounts may need to be created in several applications.

Creating each account manually takes time and can lead to mistakes.

SCIM automates identity provisioning and deprovisioning.

## 10.2 Authentication vs Provisioning

This is one of the most important differences in IAM.

| SAML / OIDC | SCIM |
|---|---|
| Used for authentication and federation | Used for identity provisioning |
| Helps users sign in | Creates and manages user accounts |
| Exchanges assertions or tokens | Exchanges user-resource data through REST APIs |
| Used during login | Used during account lifecycle changes |

**Remember:**

- SAML/OIDC: Who are you, and can you sign in?
- SCIM: Should your account exist in this application, and what information should it have?

## 10.3 What is provisioning?

Provisioning means creating or assigning an account and its required attributes in a target application.

Example:

A new employee named John joins the Engineering department.

The organisation creates his account and provisions the necessary application access.

## 10.4 What is deprovisioning?

Deprovisioning means removing or disabling an account when access is no longer required.

Example:

John leaves the company.

The organisation needs to disable or remove his access to Salesforce, Slack and other applications.

Deprovisioning reduces the risk of former employees retaining access.

## 10.5 SCIM HTTP methods

The workbook covers four important operations.

### 1. GET – Search or retrieve a user

Example:

```http
GET /v2/Users?filter=userName eq "john.doe@company.com"
```

Purpose:

Search for a user by username.

Expected result: The matching user resource, if found.

### 2. POST – Create a new user

Example:

```http
POST /v2/Users
Content-Type: application/scim+json
```

Example request body:

```json
{
  "schemas": [
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "userName": "john.doe@company.com",
  "name": {
    "givenName": "John",
    "familyName": "Doe"
  },
  "active": true
}
```

Purpose:

Create a new user in the target application.

A successful creation commonly returns:

```http
201 Created
```

### 3. PATCH – Update selected user attributes

Example:

```http
PATCH /v2/Users/{id}
Content-Type: application/scim+json
```

Example request body:

```json
{
  "schemas": [
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations": [
    {
      "op": "replace",
      "path": "department",
      "value": "Finance"
    }
  ]
}
```

Purpose:

Update a particular attribute without replacing the entire user profile.

Example: Change John's department from Engineering to Finance.

### 4. DELETE – Remove a user resource

Example:

```http
DELETE /v2/Users/{id}
```

Purpose:

Delete the user resource from the target application.

**Important:** The exact deprovisioning behaviour depends on the target system. Some applications disable accounts instead of permanently deleting them.

## 10.6 PUT vs PATCH

| PUT | PATCH |
|---|---|
| Replaces a resource representation | Applies partial modifications |
| Usually sends the full updated resource | Sends only the required changes |
| Can overwrite existing attributes | Can update selected attributes |

**Interview answer:** PATCH is used when we want to update specific attributes of a user without replacing the complete profile.

## 10.7 Real-world SCIM provisioning flow

Scenario: A new employee joins the company.

```text
          HR System
              |
              | Employee created
              v
       Identity Provider
              |
              | SCIM POST /Users
              v
        Salesforce App
              |
              | User account created
              v
         Employee can use
         assigned application
```

### Employee department changes

```text
       HR / Identity Source
                |
                | Department changes
                v
        Identity Provider
                |
                | SCIM PATCH /Users/{id}
                v
          Target Application
                |
                v
        Updated user attributes
```

### Employee leaves the company

```text
          HR System
              |
              | Employee leaves
              v
        Identity Provider
              |
              | SCIM DELETE /Users/{id}
              v
        Target Application
              |
              v
       Account deprovisioned
```

## 10.8 Important SCIM concepts

| Concept | Meaning |
|---|---|
| Resource | An identity object such as a user |
| User | A SCIM user resource |
| `userName` | User's unique login identifier in the SCIM service |
| `active` | Indicates whether the account is active |
| `id` | Identifier assigned by the SCIM service |
| Provisioning | Creating or managing an account |
| Deprovisioning | Removing or disabling account access |
| PATCH | Partially updating a resource |

## 10.9 SAML vs OIDC vs SCIM

| Feature | SAML 2.0 | OIDC | SCIM 2.0 |
|---|---|---|---|
| Main purpose | Authentication federation | Authentication and identity claims | Provisioning |
| Common format | XML | JSON / JWT | JSON |
| Main mechanism | Assertions | ID Tokens and other OAuth tokens | REST APIs |
| Common use | Enterprise SSO | Modern application login | User lifecycle management |

**Important:** SCIM is not a replacement for SAML or OIDC. They solve different identity-management problems and are often used together.

---

# Final Revision: Important Commands from Modules 1–10

## Linux commands

| Command | Purpose |
|---|---|
| `pwd` | Show current directory |
| `ls` | List files |
| `cd` | Change directory |
| `cat server.log` | Display file contents |
| `tail server.log` | Display last lines |
| `tail -f server.log` | Follow live logs |
| `grep "ERROR" server.log` | Search for errors |
| `grep -i "username" server.log` | Case-insensitive search |
| `chmod +x script.sh` | Add execute permission |
| `curl URL` | Send an HTTP request |

## systemd commands

| Command | Purpose |
|---|---|
| `systemctl start service` | Start service |
| `systemctl stop service` | Stop service |
| `systemctl restart service` | Restart service |
| `systemctl status service` | Check status |
| `systemctl enable service` | Enable startup at boot |
| `journalctl -u service -f` | Follow service logs |

## PowerShell commands

| Command | Purpose |
|---|---|
| `Get-Process` | List processes |
| `Get-Service` | List services |
| `Invoke-RestMethod` | Make REST API requests |
| `Select-Object` | Select object properties |
| `Format-Table` | Display data as a table |
| `Export-Csv` | Export data to CSV |

## Git commands

| Command | Purpose |
|---|---|
| `git init` | Initialise repository |
| `git status` | Check repository status |
| `git add .` | Stage files |
| `git commit -m "message"` | Commit changes |
| `git push` | Push commits |
| `git pull` | Retrieve and integrate remote changes |

## Docker commands

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker version |
| `docker images` | List images |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker run` | Create and start a container |
| `docker stop` | Stop a container |
| `docker rm` | Remove a container |

---

# Important Interview Questions with Answers

### 1. What is Active Directory?

Active Directory is Microsoft's directory service used to centrally manage users, computers, groups, authentication and access policies.

### 2. What is the difference between a domain and a forest?

A domain is a logical administrative unit containing directory objects. A forest is the top-level AD structure containing one or more domains that share a schema and configuration.

### 3. What is LDAP?

LDAP is a protocol used to access and manage directory information.

### 4. What is the difference between LDAP and LDAPS?

LDAP commonly uses port 389. LDAPS uses TLS-secured LDAP, commonly on port 636.

### 5. What is a Distinguished Name?

A DN uniquely identifies an entry's location in the directory hierarchy.

Example:

`cn=John,ou=Engineering,dc=company,dc=com`

### 6. What are LDAP search scopes?

Base Object searches one entry, OneLevel searches immediate children, and Subtree searches the base and all descendants.

### 7. Why is Subtree commonly used in IAM?

Because users may exist in nested OUs. Subtree searches all nested levels.

### 8. What is an LDAP filter?

An LDAP filter specifies conditions used to find directory entries.

### 9. What is the purpose of `userAccountControl`?

It is an Active Directory attribute containing account-control flags, including the disabled-account flag.

### 10. Why should an IAM service not run as root?

To follow least privilege and reduce the impact of a potential application compromise.

### 11. What is systemd?

It is a Linux service manager used to control and monitor services.

### 12. What does `journalctl -u pingfederate -f` do?

It follows the logs of the PingFederate systemd service in real time.

### 13. What is grep?

`grep` searches text for matching patterns.

### 14. How do you trace a login failure using logs?

Search for the username, find its transaction ID, then search the logs using that transaction ID to identify the cause.

### 15. What is Bash scripting?

Bash scripting automates commands and tasks on Linux systems.

### 16. What does `$?` mean in Bash?

It contains the exit status of the last executed command. Zero normally indicates success.

### 17. What is PowerShell?

PowerShell is Microsoft's command-line shell and scripting language used for automation and administration.

### 18. What is `Invoke-RestMethod`?

It sends HTTP requests and processes REST API responses in PowerShell.

### 19. Why is PowerShell useful for identity audits?

It can retrieve API or directory data, select attributes and export structured information into reports.

### 20. What is Git?

Git is a distributed version-control system used to track changes in files.

### 21. What is `.gitignore`?

It tells Git which untracked files and directories to ignore.

### 22. Does `.gitignore` remove an already committed private key?

No. Already tracked files remain tracked. The secret must be treated as exposed and rotated or revoked.

### 23. What is Docker?

Docker is a platform for building and running applications inside containers.

### 24. What is the difference between a Docker image and a container?

An image is a template. A container is a running instance created from an image.

### 25. What is SCIM?

SCIM is a standard protocol used to automate identity provisioning and deprovisioning between systems.

### 26. What is the difference between SAML and SCIM?

SAML handles identity federation and SSO authentication. SCIM handles user account provisioning and lifecycle management.

### 27. What does SCIM POST do?

It creates a new resource, such as a user.

### 28. What does SCIM PATCH do?

It partially updates an existing resource.

### 29. What does SCIM DELETE do?

It requests deletion of a user resource.

### 30. What is the difference between PUT and PATCH?

PUT generally replaces a resource representation, whereas PATCH applies partial modifications.

---

# Final Learning Checklist

Use this checklist to check your preparation.

- [ ] Module 1: Explain AD, Forest, Tree, Domain, OU, Objects, DC and GC.
- [ ] Module 1: Explain sAMAccountName, UPN, objectGUID and memberOf.
- [ ] Module 2: Explain LDAP, LDAPS, DIT, DN, Bind and search scopes.
- [ ] Module 2: Understand the `ldapsearch` command.
- [ ] Module 3: Write LDAP AND, OR and NOT filters.
- [ ] Module 3: Explain the disabled-account filter.
- [ ] Module 4: Understand Linux directories and root vs non-root.
- [ ] Module 4: Practise systemctl and journalctl commands.
- [ ] Module 5: Understand grep, tail and transaction IDs.
- [ ] Module 5: Troubleshoot an LDAP timeout using logs.
- [ ] Module 6: Understand Bash variables, conditions and exit codes.
- [ ] Module 6: Run the Identity Provider health-check script.
- [ ] Module 7: Understand PowerShell, REST APIs, JSON and pipelines.
- [ ] Module 7: Run the API audit and CSV export example.
- [ ] Module 8: Understand Git, GitHub and `.gitignore`.
- [ ] Module 9: Understand Docker images, containers and port mapping.
- [ ] Module 9: Run and remove the OpenLDAP lab container.
- [ ] Module 10: Explain SCIM provisioning and deprovisioning.
- [ ] Module 10: Understand GET, POST, PATCH and DELETE.

**End of Workbook 4 – Modules 1–10.**

*Study advice: First understand the concept, then explain the real-world example, and finally practise the command. This will prepare you to answer both theoretical and practical IAM questions.*