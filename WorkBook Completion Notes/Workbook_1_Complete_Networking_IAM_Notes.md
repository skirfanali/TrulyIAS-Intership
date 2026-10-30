# Workbook 1: Networking & Traffic Architecture for IAM

## Complete Notes, Practical Labs, Troubleshooting & Knowledge Examination

**Course:** Level 0 Engineering Foundation\
**Subject:** Networking and Traffic Architecture for Identity and Access
Management (IAM)\
**Platform:** Windows (unless otherwise stated)

------------------------------------------------------------------------

# Contents

1.  OSI Model, TCP/IP and the TCP Three-Way Handshake
2.  IP Addressing, CIDR Subnetting and Network Routing
3.  DNS Architecture, Record Types and the HOSTS File
4.  Enterprise Port Numbers and Socket Verification
5.  Reverse Proxies, SSL Offloading and Forwarded Headers
6.  Load Balancing, Clustering and Session Stickiness
7.  Enterprise DMZ Architecture and Network Placement
8.  VPN and RADIUS Architecture for MFA
9.  Mutual TLS (mTLS) and Client Certificate Authentication
10. Network Troubleshooting Toolkit
11. Curated Self-Study Resources
12. Production Outage Case Studies
13. Comprehensive Knowledge Examination

------------------------------------------------------------------------

# Module 1: OSI Model, TCP/IP and the Three-Way Handshake

## 1.1 Why networking matters in IAM

Identity and Access Management (IAM) systems handle login,
authentication, authorization and identity federation. A login request
travels from a user's device to an identity provider (IdP) through a
network.

Some important networking layers and concepts are:

-   **Layer 3 -- Network:** IP addresses and routing.
-   **Layer 4 -- Transport:** TCP ports, UDP and sockets.
-   **Layer 7 -- Application:** HTTP, SAML, OAuth and LDAP application
    communication.

Understanding these layers helps troubleshoot login failures,
connectivity problems and secure communication.

## 1.2 OSI model

The OSI model divides network communication into seven conceptual
layers.

  --------------------------------------------------------------------------
  Layer             Name              Main responsibility  Examples
  ----------------- ----------------- -------------------- -----------------
  7                 Application       Network services     HTTP, DNS, SMTP
                                      used by applications 

  6                 Presentation      Data representation  Data encoding,
                                      and                  TLS-related
                                      encryption-related   presentation
                                      functions            concepts

  5                 Session           Establishes and      Session
                                      manages              management
                                      communication        
                                      sessions             

  4                 Transport         End-to-end transport TCP, UDP
                                      and port addressing  

  3                 Network           Logical addressing   IP, routers
                                      and routing          

  2                 Data Link         Local network frames Ethernet,
                                      and MAC addressing   switches

  1                 Physical          Transmission of raw  Cables, radio
                                      bits                 signals
  --------------------------------------------------------------------------

**Important:** TCP/IP is a practical networking model and protocol
suite. It is commonly represented using four layers: Application,
Transport, Internet and Link. It is not another name for the OSI model.

## 1.3 TCP/IP model

  -----------------------------------------------------------------------
  TCP/IP layer            Main function           Examples
  ----------------------- ----------------------- -----------------------
  Application             Application-level       HTTP, HTTPS, DNS, SSH
                          communication           

  Transport               Process-to-process      TCP, UDP
                          communication           

  Internet                IP addressing and       IPv4, IPv6, ICMP
                          routing                 

  Link                    Communication over the  Ethernet, Wi-Fi
                          local network           
  -----------------------------------------------------------------------

The TCP/IP model groups some OSI layers together. For example, the
TCP/IP Application layer covers responsibilities represented by the OSI
Application, Presentation and Session layers.

## 1.4 TCP three-way handshake

TCP is a connection-oriented transport protocol. Before normal TCP
application data is exchanged, the client and server establish a
connection using a three-way handshake.

### Three steps

1.  **SYN:** The client sends a SYN packet to request a TCP connection.
2.  **SYN-ACK:** The server responds with SYN-ACK to acknowledge the
    request and indicate that it is ready.
3.  **ACK:** The client sends ACK. The TCP connection is now
    established.

``` text
Client                                      Server
  |                                           |
  |------------ SYN ------------------------->|
  |                                           |
  |<----------- SYN + ACK --------------------|
  |                                           |
  |------------ ACK ------------------------->|
  |                                           |
  |          TCP connection established       |
  |                                           |
  |====== Application data can begin ========|
```

The sequence and acknowledgement numbers help TCP track bytes and
confirm reliable delivery.

### What if the target port is closed?

If a host is reachable but no service is listening on the requested TCP
port, it commonly responds with a TCP **RST (Reset)** packet. This
indicates that the connection is refused.

If packets are silently dropped by a firewall or network, the client may
receive no response and eventually time out.

  -----------------------------------------------------------------------
  Result                              Typical meaning
  ----------------------------------- -----------------------------------
  SYN, SYN-ACK, ACK                   TCP connection established

  RST                                 Connection refused / port not
                                      listening

  No response until timeout           Packets may be dropped, filtered or
                                      the host may be unreachable
  -----------------------------------------------------------------------

## 1.5 TLS handshake

TLS (Transport Layer Security) protects communication by providing
encryption, integrity and authentication.

For HTTPS, TCP connection establishment generally occurs before the TLS
handshake when using HTTP/1.1 or HTTP/2 over TCP.

A simplified TLS 1.3 handshake includes:

1.  Client sends **ClientHello**, including supported TLS versions and
    key-share information.
2.  Server responds with **ServerHello** and selects the cryptographic
    parameters.
3.  The parties derive shared traffic keys.
4.  The server proves its identity using its certificate and handshake
    messages.
5.  The client verifies the server certificate and handshake.
6.  Encrypted application data is exchanged.

TLS 1.3 can complete its main handshake in one round trip in the usual
full-handshake case. The exact message flow can vary.

## 1.6 Lab 1A: Visualizing TCP and TLS timings in Chrome DevTools

**Objective:** Observe browser-reported connection timing information
without installing additional software.

### Procedure

1.  Open Google Chrome or Microsoft Edge.
2.  Press **F12**, or right-click a page and select **Inspect**.
3.  Open the **Network** tab.
4.  Select **Disable cache**.
5.  Visit `https://accounts.google.com`.
6.  In the Network request list, locate the main document request. It
    may appear as `accounts.google.com` or a redirected Google Account
    hostname.
7.  If necessary, select the **Doc** filter to show document requests.
8.  Click the main document request.
9.  Open the **Timing** tab in its request details.

### Timing fields

  -----------------------------------------------------------------------
  Field                               Meaning
  ----------------------------------- -----------------------------------
  Queueing                            Time the request waited before it
                                      could begin

  Stalled                             Time spent waiting before request
                                      transmission

  DNS Lookup                          Time spent resolving the hostname

  Initial connection                  Browser-reported connection
                                      establishment time; may include TCP
                                      setup

  SSL                                 Browser-reported TLS negotiation
                                      time

  Request sent                        Time spent sending the request

  Waiting for server response         Time waiting for the first response
                                      byte

  Content Download                    Time spent receiving the response
  -----------------------------------------------------------------------

**Interpretation:** The Initial connection and SSL timing sections
provide browser-level evidence of connection setup and TLS negotiation.
They do not show each individual SYN, SYN-ACK and ACK packet. A packet
capture tool such as Wireshark is needed to inspect individual packets.

## 1.7 Lab 1B: Capturing the TCP handshake with Wireshark

**Objective:** Observe the actual TCP handshake packets.

### Procedure

1.  Open Wireshark.
2.  On the start screen, identify the active network adapter, usually
    Wi-Fi or Ethernet.
3.  Double-click the active adapter to start capturing.
4.  In the display filter bar, enter:

``` text
tcp.port == 443 && ip.addr == 1.1.1.1
```

5.  Press Enter.
6.  Open Command Prompt and run:

``` cmd
curl.exe -I https://1.1.1.1
```

7.  Return to Wireshark and stop the capture using the red square
    button.
8.  Inspect the **Info** column for the TCP connection packets.

### Expected packet sequence

``` text
Client ephemeral port -> 443 [SYN]
443 -> Client ephemeral port [SYN, ACK]
Client ephemeral port -> 443 [ACK]
```

The actual ephemeral port and sequence numbers will vary. The connection
may also show additional packets and TLS traffic.

------------------------------------------------------------------------

# Module 2: IP Addressing, CIDR Subnetting and Network Routing

## 2.1 IP addressing

An IP address identifies a network interface so that data can be
delivered across networks.

-   **IPv4:** 32-bit address, usually written as four decimal octets,
    e.g. `192.168.1.10`.
-   **IPv6:** 128-bit address, written in hexadecimal groups,
    e.g. `2001:db8::1`.

Routers use IP addresses to forward packets between networks.

## 2.2 CIDR notation

CIDR stands for **Classless Inter-Domain Routing**. CIDR notation uses a
slash followed by the number of network-prefix bits.

Example:

``` text
198.51.100.16/28
```

-   `/28` means 28 bits are used for the network prefix.
-   IPv4 has 32 total bits.
-   Host bits = 32 − 28 = 4.
-   Total addresses = 2⁴ = 16.
-   In a typical IPv4 subnet, network and broadcast addresses are not
    assigned to ordinary hosts.
-   Usable host addresses = 16 − 2 = 14.

## 2.3 Lab 2A: Subnet calculation

**Scenario:** An organization wants to configure a trusted office
network using `198.51.100.16/28`.

### Calculation

1.  Host bits = 32 − 28 = 4.
2.  Total IP addresses = 2⁴ = 16.
3.  Network address = `198.51.100.16`.
4.  Broadcast address = `198.51.100.31`.
5.  Usable host range = `198.51.100.17` to `198.51.100.30`.
6.  Usable hosts = 14.

### Why correct CIDR matters in IAM

An IAM conditional-access policy may trust a particular corporate IP
range. Entering a broader subnet than intended can allow more IP
addresses than planned. This can weaken access restrictions.

For example, `/16` covers a much larger range than `/28`. Always verify
the network prefix before applying a trusted-network policy.

## 2.4 Network routing and gateways

A router forwards packets between different IP networks. A **default
gateway** is the router a device uses to reach destinations outside its
local network.

A route trace helps show the sequence of network hops toward a
destination.

## 2.5 Lab 2B: Trace network hops using tracert

Open Command Prompt and run:

``` cmd
tracert 8.8.8.8
```

Example output (your results will differ):

``` text
Tracing route to dns.google [8.8.8.8]
1    1 ms    1 ms    1 ms    192.168.1.1
2    9 ms    8 ms    8 ms    10.240.0.1
3   14 ms   14 ms   14 ms    8.8.8.8
```

-   Hop 1 may be your local Wi-Fi router.
-   Later hops may belong to your ISP or other networks.
-   A `*` means no response was received within the probe timeout. It
    does not always mean the route is broken; routers may filter or
    deprioritize traceroute probes.

------------------------------------------------------------------------

# Module 3: DNS Architecture, Record Types and the HOSTS File

## 3.1 What is DNS?

DNS stands for **Domain Name System**. It translates human-readable
hostnames into IP addresses so computers can locate network services.

For example:

``` text
www.example.com -> an IP address
```

Without DNS, users would generally need to remember numerical IP
addresses to reach internet services.

## 3.2 Common DNS record types

  -----------------------------------------------------------------------
  Record                  Purpose                 Example use
  ----------------------- ----------------------- -----------------------
  A                       Maps a hostname to an   `app.example.com` to an
                          IPv4 address            IPv4 address

  AAAA                    Maps a hostname to an   IPv6 service address
                          IPv6 address            

  CNAME                   Makes one hostname an   `www` aliasing a
                          alias of another        canonical hostname
                          hostname                

  MX                      Identifies mail servers Email delivery
                          for a domain            

  TXT                     Stores text data        Domain ownership
                          associated with a       verification and email
                          domain                  security policies

  NS                      Identifies              DNS zone delegation
                          authoritative name      
                          servers                 

  PTR                     Reverse lookup from IP  Reverse DNS
                          address to hostname     
  -----------------------------------------------------------------------

## 3.3 Lab 3A: Query DNS records

Open Command Prompt or PowerShell.

### Query an A/AAAA result

``` cmd
nslookup login.microsoftonline.com
```

This asks DNS to resolve the hostname. The response may include IPv4 or
IPv6 addresses depending on the DNS response.

### Query TXT records

``` cmd
nslookup -type=TXT microsoft.com
```

TXT records can be used by cloud identity providers to verify domain
ownership. A provider may ask an organization to publish a specific TXT
value before allowing a custom login domain or federation configuration.

## 3.4 What is the HOSTS file?

The Windows HOSTS file is a local text file that maps hostnames to IP
addresses. It can provide a manual hostname mapping on one computer.

Windows HOSTS file location:

``` text
C:\Windows\System32\drivers\etc\hosts
```

Example entry:

``` text
127.0.0.1 sso.training.local
```

This tells Windows to resolve `sso.training.local` to `127.0.0.1` on
that computer.

### Important terms

-   **127.0.0.1:** IPv4 loopback address; it refers to the local
    computer.
-   **localhost:** Common hostname for the local computer.
-   **DNS:** A distributed naming system that resolves hostnames.
-   **HOSTS file:** A local file that can manually map hostnames to IP
    addresses.
-   **Comment marker (`#`):** A line beginning with `#` is ignored.

## 3.5 Why is the HOSTS file used?

-   Local development and testing.
-   Testing a hostname against a specific IP before changing DNS.
-   Cybersecurity and networking labs.
-   Troubleshooting hostname resolution.
-   Simulating an internal training hostname on one computer.

In IAM, a test hostname such as `sso.training.local` can help
demonstrate name resolution for an SSO application.

**Important:** Mapping a hostname to `127.0.0.1` does not start an SSO
application. A local web server or SSO service must be running and
listening on the requested port.

## 3.6 Lab 3B: Simulating local hostname resolution with the HOSTS file

### Procedure

1.  Open the Windows Start menu.
2.  Search for **Notepad**.
3.  Right-click Notepad and choose **Run as administrator**.
4.  Approve the administrator prompt.
5.  In Notepad, select **File → Open**.
6.  Navigate to:

``` text
C:\Windows\System32\drivers\etc\
```

7.  Change the file-type dropdown to **All Files (*.*)**.
8.  Open the file named `hosts`.
9.  Add the following line at the bottom:

``` text
127.0.0.1 sso.training.local
```

10. Save using **Ctrl + S**.

If the file already contains:

``` text
# 127.0.0.1 sso.training.local
```

the mapping is commented out. Remove the `#` to activate it.

> Use the IP address supplied by your instructor if the training server
> is not running on your own computer. Do not assume `127.0.0.1` is
> correct for a remote training server.

### Flush the DNS resolver cache

Open PowerShell or Command Prompt and run:

``` powershell
ipconfig /flushdns
```

Expected message:

``` text
Windows IP Configuration

Successfully flushed the DNS Resolver Cache.
```

### Test hostname resolution

Run:

``` powershell
ping sso.training.local
```

If the local example mapping is active, the output should begin similar
to:

``` text
Pinging sso.training.local [127.0.0.1]
```

The ping reply may vary because ICMP echo traffic can be blocked. The
important check is whether the hostname resolves to the expected IP.

### Troubleshooting

  ------------------------------------------------------------------------------------
  Problem                              Possible reason         Solution
  ------------------------------------ ----------------------- -----------------------
  `Ping request could not find host`   HOSTS entry missing,    Check the exact
                                       misspelled or commented hostname and remove `#`
                                       out                     

  Cannot save the file                 Notepad lacks           Reopen Notepad as
                                       administrator           Administrator
                                       permission              

  HOSTS file is not visible            Open dialog filters     Select **All Files
                                       text files              (*.*)**

  Ping times out but hostname resolves ICMP may be blocked     Verify the resolved IP
                                                               separately

  Website does not open                No local service is     Start the application
                                       listening               and check its port
  ------------------------------------------------------------------------------------

### Cleanup

After the lab, remove the line you added from the HOSTS file and save
it. Then run:

``` powershell
ipconfig /flushdns
```

This restores normal hostname resolution for that entry.

------------------------------------------------------------------------

# Module 4: Enterprise Port Numbers and Socket Verification

## 4.1 What is a port?

A port number identifies a particular service or application endpoint on
a device. IP addresses identify network interfaces; port numbers help
direct transport-layer traffic to the correct service.

A **socket** is commonly described by an IP address, transport protocol
and port number.

## 4.2 Important enterprise IAM ports

  -------------------------------------------------------------------------
                   Port Protocol         Service          Role in identity
                                                          architecture
  --------------------- ---------------- ---------------- -----------------
                    443 TCP              HTTPS            SSO portals,
                                                          OAuth token
                                                          requests and SCIM
                                                          provisioning

                    636 TCP              LDAPS            Encrypted
                                                          directory lookups
                                                          and
                                                          authentication
                                                          against directory
                                                          services

                     88 TCP/UDP          Kerberos         Windows
                                                          integrated
                                                          authentication
                                                          and single
                                                          sign-on

                   1812 UDP              RADIUS           VPN and
                                         authentication   network-access
                                                          authentication,
                                                          including MFA
                                                          workflows

                   9999 TCP              PingFederate     Administrative
                                         Admin (as        console; should
                                         specified in     not be publicly
                                         this workbook)   exposed
  -------------------------------------------------------------------------

## 4.3 Lab 4A: Test ports using PowerShell

Open PowerShell.

### Test HTTPS port 443

``` powershell
Test-NetConnection -ComputerName "login.microsoftonline.com" -Port 443
```

A successful connection may show:

``` text
TcpTestSucceeded : True
```

### Test another port

``` powershell
Test-NetConnection -ComputerName "login.microsoftonline.com" -Port 389
```

If the destination does not accept TCP connections on that port, the
result may show:

``` text
TcpTestSucceeded : False
```

A failed test does not by itself prove that the port is closed; firewall
filtering, routing and service configuration can also cause failure.

------------------------------------------------------------------------

# Module 5: Reverse Proxies, SSL Offloading and Forwarded Headers

## 5.1 What is a reverse proxy?

A reverse proxy sits in front of one or more backend servers. It
receives client requests and forwards them to the appropriate backend
application.

Common functions include:

-   Forwarding client requests to backend servers.
-   TLS termination or SSL offloading.
-   Load balancing.
-   Security filtering through a Web Application Firewall (WAF).
-   Hiding internal server details from public clients.

## 5.2 SSL/TLS offloading

In SSL/TLS offloading, the reverse proxy handles the public HTTPS
connection. It may then communicate with an internal application using
HTTP or another protected internal connection, depending on the
architecture.

Example:

``` text
Browser
  |
  | HTTPS (public connection)
  v
Reverse Proxy / WAF
  |
  | Internal HTTP (example: port 8080)
  v
Identity Provider (IdP)
```

The proxy should pass the correct request context to the backend.

## 5.3 Important forwarded headers

  -----------------------------------------------------------------------
  Header                              Purpose
  ----------------------------------- -----------------------------------
  `Host`                              Carries the original host requested
                                      by the client

  `X-Forwarded-For`                   Carries the client IP address as
                                      seen by the proxy chain

  `X-Forwarded-Proto`                 Indicates the original request
                                      scheme, such as `https`

  `X-Forwarded-Port`                  Indicates the original
                                      client-facing port
  -----------------------------------------------------------------------

These headers must be trusted only when they are set or sanitized by
trusted proxies. Otherwise, clients may spoof them.

## 5.4 Infinite 302 redirect loop

A 302 response means **Found** and commonly instructs the browser to
visit another URL.

A redirect loop can happen when:

1.  The browser connects to the reverse proxy over HTTPS.
2.  The proxy terminates TLS and forwards an HTTP request to the
    identity provider.
3.  The proxy fails to send `X-Forwarded-Proto: https`.
4.  The identity provider believes the original request used HTTP.
5.  The identity provider redirects the browser to HTTPS.
6.  The browser repeats the same path and the loop continues.

The browser may eventually display:

``` text
ERR_TOO_MANY_REDIRECTS
```

## 5.5 Lab 5A: Inspect SSO redirect headers in DevTools

1.  Open Chrome.
2.  Press **F12** and select **Network**.
3.  Enable **Preserve log**.
4.  Visit:

``` text
https://github.com/login
```

5.  Use the **Doc** filter.
6.  Inspect the document requests and redirects.
7.  Select a request with a 302 status, if present.
8.  Open **Headers** and inspect the **Response Headers** section.
9.  Find the `Location` header, which contains the redirect destination.

Use only a test account and never submit real credentials to an
untrusted page. The exact requests and status codes may vary.

------------------------------------------------------------------------

# Module 6: Load Balancing, Clustering and Session Stickiness

## 6.1 What is load balancing?

Load balancing distributes incoming requests across multiple backend
servers. It helps improve availability, scalability and performance.

A load balancer may use algorithms such as:

-   Round-robin.
-   Least connections.
-   Weighted distribution.
-   IP-hash or cookie-based persistence.

## 6.2 What is clustering?

A cluster is a group of servers that work together to provide a service.
In an identity platform, multiple IdP nodes may process authentication
requests.

Clustering can improve availability, but the nodes must handle
authentication state consistently.

## 6.3 Why round-robin can break MFA

Consider a two-step MFA login:

1.  The user enters a password and reaches **Node 1**.
2.  Node 1 stores pending authentication state and triggers an MFA push.
3.  The user approves the push.
4.  The next request is routed to **Node 2** by round-robin.
5.  Node 2 does not have the pending state, so authentication may fail
    with an expired or invalid state message.

This can happen when session state is stored only in one node's memory
and is not shared.

## 6.4 Session stickiness

Session stickiness (session persistence) attempts to route a user's
related requests to the same backend node for a period of time.

It can be implemented using a load-balancer cookie or another
persistence mechanism.

Examples of cookies that may be seen in some environments:

-   `AWSALB`
-   `BIGipServer...`
-   `JSESSIONID`

The exact cookie depends on the load balancer and application. A
cookie's presence alone does not prove that the system is configured
securely or correctly.

## 6.5 Lab 6A: Locate cookies in Chrome DevTools

1.  Open Chrome and visit the application.
2.  Press **F12**.
3.  Select the **Application** tab.
4.  In the left panel, expand **Storage → Cookies**.
5.  Select the relevant website.
6.  Inspect cookie names and attributes.

Useful cookie attributes include:

-   **Secure:** Cookie is sent over secure HTTPS connections.
-   **HttpOnly:** JavaScript cannot access the cookie through
    `document.cookie`.
-   **SameSite:** Controls when cookies are sent in cross-site contexts.
-   **Expires/Max-Age:** Controls cookie lifetime.

------------------------------------------------------------------------

# Module 7: Enterprise DMZ Architecture and Network Placement

## 7.1 What is a DMZ?

DMZ stands for **Demilitarized Zone**. It is a separate network segment
placed between an untrusted external network and a trusted internal
network.

It allows organizations to expose selected services while reducing
direct access to internal systems.

## 7.2 Three common zones

1.  **Public zone:** Internet users and remote clients.
2.  **DMZ zone:** Public-facing reverse proxies, gateways and WAFs.
3.  **Internal intranet:** Protected identity engines, directory
    services and administrative systems.

## 7.3 Example IAM architecture

``` text
PUBLIC ZONE
Internet / Remote Users
        |
        | HTTPS, TCP 443
        v
DMZ ZONE
Reverse Proxy / WAF
(F5 BIG-IP / NGINX)
        |
        | Approved internal traffic
        v
INTERNAL NETWORK
Identity Provider Cluster
(PingFederate Nodes)
        |
        | LDAPS, TCP 636
        v
Active Directory

Internal administrator
        |
        | Restricted management access
        v
IDP Admin Console
(example port 9999 in this workbook)
```

## 7.4 Security principles

-   Expose only required public services.
-   Restrict administrative interfaces to trusted management networks.
-   Apply firewall rules between zones.
-   Monitor and log access.
-   Keep identity servers and directory services protected from direct
    public access.

Port 9999, identified in this workbook as the PingFederate Admin Console
port, should not be exposed to the public internet.

------------------------------------------------------------------------

# Module 8: VPN and RADIUS Architecture for MFA

## 8.1 What is a VPN?

A Virtual Private Network (VPN) creates a protected communication path
over an underlying network. Organizations use VPNs to provide remote
users with access to approved internal resources.

## 8.2 What is RADIUS?

RADIUS stands for **Remote Authentication Dial-In User Service**. It is
a protocol used for centralized authentication, authorization and
accounting (AAA).

RADIUS commonly uses UDP:

-   UDP 1812: Authentication.
-   UDP 1813: Accounting.

The workbook focuses on UDP port 1812 for authentication.

## 8.3 RADIUS-based MFA flow

``` text
Remote User       VPN Gateway       RADIUS Agent / NPS       Cloud MFA
    |                  |                     |                   |
    |-- Username/Pass->|                     |                   |
    |                  |-- Access-Request -->|                   |
    |                  |                     |-- Check password->|
    |                  |                     |-- Push challenge->|
    |<---------------------- Mobile push ------------------------|
    |-- User approves ------------------------------------------->|
    |                  |<-- Access-Accept ---|<-- Approval -------|
    |<-- VPN access ---|                     |                   |
```

The precise architecture varies by vendor and deployment.

## 8.4 Why timeout settings matter

RADIUS uses connectionless UDP. A VPN gateway waits for a response for a
configured period.

Example incident:

-   The VPN gateway has a 5-second timeout.
-   The employee takes 8 seconds to approve a mobile push.
-   The gateway stops waiting before the approval result returns.
-   The login fails even though the user approved the push.

The workbook's suggested remediation is to increase the RADIUS timeout
to **60 seconds** with **2 retries**, subject to the organization's
security and vendor configuration requirements.

## 8.5 Important distinction

A timeout does not always mean that the password is incorrect. It may
mean that the authentication response did not arrive before the
gateway's timer expired.

------------------------------------------------------------------------

# Module 9: Mutual TLS (mTLS) and Client Certificate Authentication

## 9.1 What is TLS?

TLS (Transport Layer Security) is a protocol that protects data
exchanged over a network. HTTPS uses TLS to protect HTTP communication.

In ordinary server-authenticated TLS, the server presents a certificate
so the client can verify the server's identity.

## 9.2 What is Mutual TLS?

Mutual TLS (mTLS) adds client certificate authentication. Both sides
authenticate using certificates:

-   The client verifies the server certificate.
-   The server requests and verifies the client's certificate.

This is useful for machine-to-machine APIs, enterprise integrations and
services that require strong client identity.

## 9.3 Public key and private key

-   **Public key:** Can be shared and is included in a certificate. It
    is used in cryptographic operations such as verification or
    encryption, depending on the algorithm.
-   **Private key:** Must be kept secret by its owner. It is used for
    signing or decryption operations, depending on the algorithm.

A certificate binds an identity to a public key. The private key should
not be shared.

## 9.4 How mTLS works

1.  Client connects to the server.
2.  Server presents its certificate.
3.  Client validates the server certificate.
4.  Server requests a client certificate.
5.  Client presents its certificate and proves possession of the
    corresponding private key.
6.  Server validates the client certificate against its trust
    configuration.
7.  If validation succeeds, the connection proceeds.

## 9.5 Lab 9A: Test client certificate authentication using BadSSL

**Website:** `https://client.badssl.com/`

The workbook describes the following test:

1.  Open Chrome.
2.  Visit `https://client.badssl.com/`.
3.  Observe the response indicating that a required client certificate
    was not sent.
4.  Visit `https://badssl.com/download/`.
5.  Download the test client certificate file named
    `badssl.com-client.p12`.
6.  The workbook specifies the password `badssl.com`.
7.  Double-click the `.p12` file and follow the Windows Certificate
    Import Wizard.
8.  Select **Current User**.
9.  Enter the provided test password when prompted.
10. Finish the import.
11. Reopen or refresh `https://client.badssl.com/`.
12. If prompted, select the test client certificate.

The test site should accept the certificate when the test is configured
correctly.

**Safety note:** Use only the public test certificate supplied for this
exercise. Never import an unknown certificate or share a private key
from a real work or personal account.

## 9.6 How mTLS can protect API tokens

Some systems bind an access token to a client certificate. A token
holder then also needs the corresponding private key to use the token
successfully. This can reduce the risk of token replay if a token is
stolen without the key.

------------------------------------------------------------------------

# Module 10: Network Troubleshooting Toolkit

A systematic troubleshooting process helps identify whether a failure is
related to DNS, routing, ports, TLS or the application.

## 10.1 Step 1: Check DNS

``` cmd
nslookup sso.company.com
```

-   If resolution fails, check the hostname, DNS server and DNS records.
-   If resolution succeeds, continue to routing checks.

## 10.2 Step 2: Check reachability and routing

``` cmd
ping sso.company.com
```

A ping failure can occur because ICMP is blocked. Use it as one
diagnostic signal, not as the only proof of network failure.

## 10.3 Step 3: Check the TCP port

Open PowerShell:

``` powershell
Test-NetConnection -ComputerName sso.company.com -Port 443
```

Check:

``` text
TcpTestSucceeded
```

-   `True`: A TCP connection to the tested port succeeded.
-   `False`: The TCP connection could not be established. Possible
    causes include firewall filtering, routing problems or no service
    listening.

## 10.4 Step 4: Check TLS and HTTP response

``` cmd
curl.exe -Iv https://sso.company.com
```

This command asks curl to show verbose connection and TLS information
while making an HTTP HEAD request.

Possible observations:

-   Certificate verification error: certificate may be expired,
    untrusted or mismatched.
-   HTTP 500: server encountered an internal application error.
-   HTTP 502: gateway or proxy received an invalid response from an
    upstream server.
-   HTTP 503: service is unavailable.
-   Connection refused: destination actively refused the TCP connection.
-   Connection timed out: no successful connection was established
    before the timeout.

## 10.5 Troubleshooting summary

  ---------------------------------------------------------------------------------
  Check                   Command/tool                      What it helps
                                                            investigate
  ----------------------- --------------------------------- -----------------------
  DNS                     `nslookup hostname`               Hostname-to-IP
                                                            resolution

  Reachability            `ping hostname`                   Basic IP reachability,
                                                            where ICMP is allowed

  Route                   `tracert hostname`                Network path and
                                                            responding hops

  TCP port                `Test-NetConnection`              Whether a TCP port can
                                                            be reached

  TLS/HTTP                `curl.exe -Iv https://hostname`   TLS negotiation and
                                                            HTTP response

  Packet details          Wireshark                         Individual network
                                                            packets
  ---------------------------------------------------------------------------------

------------------------------------------------------------------------

# Module 11: Curated YouTube and Self-Study Resources

Use the resources below for additional learning. Mark each resource as
completed after studying it.

  ---------------------------------------------------------------------------------------------------------------------------------------------------
  Core domain       Resource            Link                                                                                        Status
  ----------------- ------------------- ------------------------------------------------------------------------------------------- -----------------
  How Internet      How Does the        https://www.youtube.com/watch?v=INQsmPf24go                                                 \[ \]
  works             Internet Work?                                                                                                  

  How Internet      Computer Networking https://www.youtube.com/watch?v=zN8YNNHCaZc                                                 \[ \]
  works             Explained                                                                                                       

  OSI Model         OSI Model explained https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/   \[ \]

  Port numbers      TCP and UDP Ports   https://www.youtube.com/watch?v=RDotMcs0Erg                                                 \[ \]
                    Explained                                                                                                       

  Port numbers      BleepingComputer    https://www.bleepingcomputer.com/tutorials/tcp-and-udp-ports-explained/                     \[ \]
                    Ports & Protocols                                                                                               
                    Guide                                                                                                           

  Web applications  Client-server       https://www.youtube.com/watch?v=N5CQK1KnRKo                                                 \[ \]
                    architecture                                                                                                    

  DNS               How DNS Works       https://www.youtube.com/watch?v=Rck3BALhl5c                                                 \[ \]

  DNS               DNS Records, TTL    https://www.youtube.com/watch?v=mpQZVYPuDGU                                                 \[ \]
                    and Root Servers                                                                                                

  HTTPS and TLS     How Does HTTPS      https://www.youtube.com/watch?v=rRoGW1fA5qE                                                 \[ \]
                    Work?                                                                                                           

  HTTPS and TLS     SSL/TLS Handshake   https://www.youtube.com/watch?v=t4Dfs_cojAs                                                 \[ \]

  Clustering        High-Availability   https://en.wikipedia.org/wiki/High-availability_cluster                                     \[ \]
                    Clustering                                                                                                      
  ---------------------------------------------------------------------------------------------------------------------------------------------------

## Personal self-study log

    No. Topic / Technology   Video or resource   URL   Date / Sign-off
  ----- -------------------- ------------------- ----- -----------------
      1                                                
      2                                                
      3                                                
      4                                                
      5                                                
      6                                                
      7                                                
      8                                                
      9                                                
     10                                                
     11                                                
     12                                                

------------------------------------------------------------------------

# Module 12: Production Outage Case Studies

## Case Study 1: Infinite redirect loop

**Incident:** NGINX reverse proxy is placed in front of PingFederate.
Remote users receive `ERR_TOO_MANY_REDIRECTS`.

### Question 1: Why is PingFederate issuing endless 302 redirects?

**Answer:** The reverse proxy terminates public TLS but forwards the
request internally without correctly indicating that the original client
connection used HTTPS. If `X-Forwarded-Proto: https` is missing or not
trusted, the identity provider may believe the original request was HTTP
and redirect to HTTPS repeatedly.

### Question 2: Which headers are identified in the workbook?

**Answer:**

``` nginx
proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Forwarded-Port 443;
```

The exact proxy configuration must match the deployment and the
application's trusted-proxy settings.

## Case Study 2: Intermittent morning MFA timeout

**Incident:** At 9:00 AM, some VPN users receive a mobile push and
approve it, but the VPN reports `Authentication Challenge Timed Out`.

### Question 1: Why can the push succeed while the VPN login fails?

**Answer:** The mobile push may be delivered and approved, but the VPN
gateway is waiting for the RADIUS response. If the gateway's timeout is
too short, it may stop waiting before the approval response returns.

### Question 2: What fix does the workbook suggest?

**Answer:** Increase the VPN gateway's RADIUS timeout from 5 seconds to
**60 seconds** with **2 retries**, after validating the appropriate
vendor and organizational settings.

------------------------------------------------------------------------

# Module 13: Comprehensive Knowledge Examination

## Question 1

**At which OSI layer does an IP router operate, and at which layer does
a Web Application Firewall (WAF) operate?**

**Answer:** An IP router primarily operates at Layer 3 (Network),
forwarding packets based on IP addresses. A WAF operates at Layer 7
(Application), inspecting HTTP traffic such as requests, headers and
payloads.

## Question 2

**What are the three packets exchanged in a TCP handshake? Which flag is
commonly returned if a TCP port is closed?**

**Answer:** The three steps are SYN, SYN-ACK and ACK. A host that is
reachable but has no service listening on the requested TCP port
commonly responds with RST (Reset).

## Question 3

**An access policy restricts API access to `203.0.113.128/28`. How many
usable IPv4 hosts exist, and what is the usable range?**

**Answer:** Host bits = 32 − 28 = 4. Total addresses = 2⁴ = 16. Usable
hosts = 16 − 2 = 14. The usable range is `203.0.113.129` through
`203.0.113.142`.

## Question 4

**Why do cloud identity providers require a TXT record before
configuring a custom login domain?**

**Answer:** A TXT record can be used to prove domain ownership. The
domain owner publishes a verification value in the authoritative DNS
zone so the provider can confirm control of the domain.

## Question 5

**List the default ports and protocols for LDAPS, Kerberos, PingFederate
Admin Console and HTTPS as given in this workbook.**

**Answer:** - LDAPS: TCP 636. - Kerberos: TCP/UDP 88. - PingFederate
Admin Console: TCP 9999 (as specified in this workbook). - HTTPS: TCP
443.

## Question 6

**What is the purpose of the `X-Forwarded-For` header?**

**Answer:** It carries the client IP address information through a
reverse proxy chain. An identity service may use this information for
logging, risk evaluation and access policies, provided the header comes
from a trusted proxy.

## Question 7

**Why can multi-step authentication in a clustered IdP deployment
require session stickiness?**

**Answer:** Pending authentication state may be stored in the memory of
the node that handled the first step. If the next request reaches
another node that does not have that state, the login may fail. Session
stickiness can keep related requests on the same node.

## Question 8

**Why should an administrative console such as the PingFederate Admin
Console not be exposed to the public internet?**

**Answer:** It is a sensitive management interface. Public exposure
increases the risk of unauthorized access, brute-force attempts and
compromise of identity infrastructure. It should be restricted to
approved management networks.

## Question 9

**How can mTLS reduce the risk of stolen OAuth access-token replay?**

**Answer:** In certificate-bound token deployments, the token is
associated with the client's certificate. The client must prove
possession of the corresponding private key when using the token, so a
stolen token alone may not be sufficient.

## Question 10

**What is the difference between `Connection refused` and
`Connection timed out`?**

**Answer:** `Connection refused` generally means the destination
actively rejected the connection, often because no service is listening
on that port. `Connection timed out` means no successful connection was
established before the timeout; packets may have been dropped or the
destination may not be reachable.

------------------------------------------------------------------------

# Quick Revision Sheet

  -----------------------------------------------------------------------
  Topic                               Key point to remember
  ----------------------------------- -----------------------------------
  OSI model                           Seven conceptual layers

  TCP/IP model                        Application, Transport, Internet
                                      and Link

  TCP handshake                       SYN → SYN-ACK → ACK

  Closed TCP port                     Commonly responds with RST

  TLS                                 Protects communication using
                                      encryption and authentication

  IPv4                                32-bit address

  CIDR `/28`                          4 host bits, 16 total addresses,
                                      usually 14 usable hosts

  DNS                                 Resolves hostnames to IP addresses

  A record                            Hostname to IPv4 address

  AAAA record                         Hostname to IPv6 address

  CNAME                               Hostname alias

  TXT record                          Text data; often used for domain
                                      verification

  HOSTS file                          Local hostname-to-IP mapping

  `127.0.0.1`                         Loopback address / local computer

  HTTPS                               TCP port 443 in the common HTTP/1.1
                                      and HTTP/2 deployment

  LDAPS                               TCP port 636

  Kerberos                            Port 88 over TCP/UDP

  RADIUS authentication               UDP port 1812

  Reverse proxy                       Receives client requests and
                                      forwards them to backend services

  `X-Forwarded-For`                   Carries client IP information
                                      through proxies

  `X-Forwarded-Proto`                 Indicates original request scheme

  Load balancer                       Distributes requests across servers

  Session stickiness                  Keeps related requests on a
                                      selected backend node

  DMZ                                 Buffer network between public and
                                      trusted internal networks

  mTLS                                Both client and server authenticate
                                      using certificates

  Public key                          Can be shared

  Private key                         Must be kept secret

  `nslookup`                          Tests DNS resolution

  `tracert`                           Shows responding network hops

  `Test-NetConnection`                Tests TCP connectivity to a host
                                      and port

  `curl.exe -Iv`                      Inspects verbose TLS/HTTP
                                      connection details
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Final Conclusion

This workbook introduces the networking concepts used in enterprise
Identity and Access Management. It covers the OSI and TCP/IP models, TCP
and TLS handshakes, IP addressing, CIDR subnetting, DNS, enterprise
ports, reverse proxies, load balancing, DMZ architecture, VPN/RADIUS
authentication, mTLS and network troubleshooting.

The practical labs demonstrate how to inspect browser network timings,
capture TCP packets, query DNS, configure a local HOSTS entry, test TCP
ports, inspect redirects and examine client certificate authentication.

A strong understanding of these topics helps engineers troubleshoot
authentication failures, connectivity issues and secure communication in
IAM environments.
