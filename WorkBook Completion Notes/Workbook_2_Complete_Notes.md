# Workbook 2: Cryptography, PKI & Trust Chains for IAM

## Complete Study Notes 

**Level:** Engineering Foundation\
**Focus:** Cryptography, password protection, key management, digital
signatures, TLS, PKI, certificates, revocation, and JWT security in
Identity and Access Management (IAM).

------------------------------------------------------------------------

## Introduction: Why cryptography matters in IAM

Identity and Access Management (IAM) systems need to establish trust
between users, applications, Identity Providers (IdPs), and Service
Providers (SPs).

Cryptography supports this trust in several ways:

-   Password hashing protects stored credentials.
-   Encryption protects confidential information.
-   Digital signatures help verify the origin and integrity of SAML
    assertions and JWTs.
-   TLS protects data while it travels across a network.
-   PKI and certificates help systems establish trust in public keys.
-   JWKS allows applications to obtain public keys used to verify signed
    tokens.

------------------------------------------------------------------------

# Module 1: Encoding vs. Encryption vs. Hashing in IAM

## 1.1 Encoding

Encoding changes data into another representation so it can be safely
transmitted or stored in a particular format.

**Key points:** - Reversible: anyone who knows the encoding method can
decode it. - Does not require a secret key. - Does not provide
confidentiality or password security.

**Example:** Base64 converts binary or text data into an
ASCII-compatible representation.

## 1.2 Encryption

Encryption converts readable plaintext into ciphertext to protect
confidentiality.

**Key points:** - Reversible by an authorized party with the correct
decryption key. - Requires cryptographic keys. - Used to protect
sensitive information.

**Examples:** AES and RSA.

## 1.3 Hashing

Hashing converts input data into a fixed-length output called a hash or
digest.

**Key points:** - Designed as a one-way operation. - Does not normally
use a decryption key. - A small change in the input should produce a
substantially different digest. - Used for integrity checks and as part
of digital-signature and password-storage systems.

**Examples:** SHA-256 and SHA-512. For password storage, use a dedicated
password-hashing algorithm such as Argon2id or bcrypt rather than a fast
general-purpose hash alone.

## 1.4 Comparison

  -----------------------------------------------------------------------
  Mechanism         Main purpose      Reversible?       Secret key
                                                        required?
  ----------------- ----------------- ----------------- -----------------
  Encoding          Data              Yes               No
                    representation /                    
                    transmission                        

  Encryption        Confidentiality   Yes, with the     Yes
                                      correct key       

  Hashing           Fixed-length      No practical      No
                    digest /          reversal          
                    integrity support                   
  -----------------------------------------------------------------------

## 1.5 SAML 2.0 transmission pipelines

SAML messages are XML documents. The binding determines how the message
is transported.

### HTTP Redirect Binding

The workbook's pipeline is:

`Raw XML → Raw Deflate → Base64 → URL Encode → SAMLRequest query parameter`

-   **Raw XML:** the original SAML message.
-   **Raw Deflate:** compresses the XML without a zlib wrapper.
-   **Base64:** converts binary compressed data into text.
-   **URL Encode:** makes the text safe to place in a URL query string.

A redirect request may be carried in a URL parameter such as
`SAMLRequest`.

### HTTP POST Binding

The workbook's pipeline is:

`Raw XML → Base64 → hidden HTML form input → POST`

The Base64-encoded SAML message is placed in a hidden form field such as
`SAMLResponse`.

### Base64URL in OAuth and JWT

Standard Base64 uses characters such as `+`, `/`, and `=`. Base64URL
adapts the representation for URLs:

-   `+` becomes `-`
-   `/` becomes `_`
-   trailing `=` padding is omitted

JWT uses Base64URL-encoded components.

## Lab 1A: Deconstructing the SAML encoding pipeline in CyberChef

**Tool:** [CyberChef](https://gchq.github.io/CyberChef/)

1.  Open CyberChef.
2.  Paste the sample XML into the Input box:

``` xml
<samlp:AuthnRequest xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="_12345" Version="2.0" IssueInstant="2026-10-30T10:00:00Z"/>
```

3.  Add these operations to the Recipe in order:
    -   Raw Deflate
    -   To Base64
    -   URL Encode
4.  Inspect the output. It is the compact representation used in a SAML
    Redirect URL.
5.  To reverse the process, use the inverse sequence:
    -   URL Decode
    -   From Base64
    -   Raw Inflate

The original XML should be restored.

**Important:** The decompression operation must match the compression
format. Raw Inflate is used to reverse Raw Deflate.

## Lab 1B: Avalanche effect of cryptographic hashing

The avalanche effect means that a small change in input causes a large,
unpredictable change in the resulting hash.

1.  In CyberChef, add **SHA2** and select **SHA-256**.
2.  Enter:

`user=alice&role=employee`

The workbook's sample digest is:

`3ab10a41f6e2b2c9df76de45d4eefd45906ef7efb8df29be99ef7665ccfe5a8d`

3.  Change one character so the input becomes:

`user=alice&role=employed`

The workbook's sample digest is:

`8d893f4a3e2c3664f33b1e621d98fa6a524c7f0d01bc18ff394145718dfd38a0`

4.  Compare the two digests. A one-character change produces a
    substantially different digest.

**Why it matters in IAM:** Hashes are used in integrity checks and
digital-signature processes. A changed message should fail the
corresponding integrity/signature verification.

------------------------------------------------------------------------

# Module 2: Password Storage, Salting & Directory Cryptography

## 2.1 Why plain SHA-256 is not suitable for password storage

SHA-256 is designed to be fast. That is useful for many integrity tasks,
but it also makes large-scale password guessing easier for an attacker
who obtains a password database.

Password storage should use a dedicated password-hashing function that
is deliberately expensive to calculate.

Examples: - Argon2id - bcrypt - PBKDF2

The workbook emphasizes slow, salted password-hashing algorithms for
directory password storage.

## 2.2 What is salting?

A **salt** is a unique, random value added to a password before password
hashing.

Why use a salt? - Identical passwords can produce different stored hash
values when different salts are used. - Precomputed lookup tables become
less useful. - It makes it harder to identify users who chose the same
password.

The salt is normally stored alongside the password hash. It does not
need to be secret.

## 2.3 bcrypt hash structure

A bcrypt string commonly has fields representing: - Algorithm/version
identifier, such as `$2b$` - Cost factor - Salt - Resulting password
hash

The cost factor controls how much work bcrypt performs. For example,
cost 12 corresponds to a work factor of approximately `2^12` iterations.

**Note:** The sample bcrypt-looking string in the workbook is
illustrative. In real work, use a valid hash generated by a trusted
bcrypt implementation.

## 2.4 Argon2id

Argon2id is a password-hashing algorithm designed to make password
guessing expensive in both time and memory.

Example structure:

`$argon2id$v=19$m=65536,t=3,p=4$SALT$HASH`

  Field        Meaning
  ------------ ------------------------------
  `argon2id`   Algorithm
  `v=19`       Argon2 version
  `m=65536`    Memory cost in KiB (64 MiB)
  `t=3`        Number of passes / time cost
  `p=4`        Parallelism setting (lanes)
  `SALT`       Encoded salt
  `HASH`       Encoded derived hash

**Parallelism `p=4`:** Argon2 uses four lanes of work. It may use
multiple threads depending on the implementation and available
resources.

## 2.5 LDAP SSHA hash structure

The workbook's sample LDAP value is:

`{SSHA}D4Gk2aVd3h9kK6v+P4f/6Rz7L6A0b3JhbmRvbQ==`

SSHA means salted SHA-1. In the format described in the workbook: - The
value after `{SSHA}` is Base64-encoded. - After Base64 decoding, the
first 20 bytes are the SHA-1 digest. - Remaining bytes are the salt.

### Lab 2B: Extracting the LDAP SSHA digest and salt in CyberChef

1.  Paste the LDAP value into CyberChef.
2.  Remove the `{SSHA}` prefix.
3.  Add **From Base64**.
4.  Add **To Hex** to view the decoded bytes as hexadecimal.
5.  Remember: one byte is represented by two hexadecimal characters.
6.  SHA-1 produces a 160-bit digest:
    -   160 bits ÷ 8 = 20 bytes
    -   20 bytes × 2 hex characters = 40 hex characters
7.  The first 40 hexadecimal characters represent the 20-byte digest.
8.  The remaining hexadecimal characters represent the salt.

To split by bytes in CyberChef, perform the extraction on the decoded
binary data: - Use **Take bytes**, start `0`, length `20`, to obtain the
digest. - Use **Drop bytes**, start `0`, length `20`, to obtain the
remaining salt. - If you have already converted to hexadecimal text,
remember that 20 bytes correspond to 40 hex characters, not 20
characters.

**Why this matters:** It demonstrates how an LDAP password value can
contain both a digest and salt. The password is verified by hashing the
candidate password with the stored salt and comparing the resulting
digest.

------------------------------------------------------------------------

# Module 3: Symmetric, Asymmetric & Hybrid Encryption in IAM

## 3.1 Symmetric encryption

Symmetric encryption uses the same secret key to encrypt and decrypt
data.

**Advantages:** - Fast. - Suitable for large amounts of data.

**Challenge:** - The communicating parties must securely share the
secret key.

**Example:** AES-256.

## 3.2 Asymmetric encryption

Asymmetric cryptography uses a mathematically related key pair: -
**Public key:** can be shared with other parties. - **Private key:**
must be kept secret by its owner.

Depending on the algorithm and use, asymmetric cryptography supports
encryption/key establishment or digital signatures.

**Examples:** RSA and elliptic-curve cryptography.

## 3.3 Hybrid encryption

Hybrid encryption combines symmetric and asymmetric cryptography.

Typical process: 1. The sender generates a random symmetric
data-encryption key. 2. The sender encrypts the large message using the
symmetric key. 3. The sender protects the small symmetric key using the
recipient's public-key mechanism. 4. The recipient uses the
corresponding private key to recover the symmetric key. 5. The recipient
decrypts the message with the symmetric key.

**Why hybrid encryption?** - Symmetric encryption is efficient for large
data. - Asymmetric cryptography helps solve the key-distribution
problem. - The combination provides practical performance and key
protection.

The workbook relates this approach to SAML EncryptedAssertion and JWE.

## 3.4 RSA-2048 and ECC P-256

### RSA key pair using OpenSSL

Generate the private key:

``` bash
openssl genrsa -out idp_rsa_priv.key 2048
```

-   `openssl`: runs the OpenSSL toolkit.
-   `genrsa`: generates an RSA private key.
-   `-out`: specifies the output filename.
-   `idp_rsa_priv.key`: private-key file.
-   `2048`: RSA modulus/key size in bits.

Extract the public key:

``` bash
openssl rsa -in idp_rsa_priv.key -pubout -out idp_rsa_pub.key
```

-   `rsa`: performs an RSA key operation.
-   `-in`: input private-key file.
-   `-pubout`: outputs the corresponding public key.
-   `-out`: destination public-key file.

Verify the files in Windows Command Prompt:

``` cmd
dir idp_rsa*
```

Expected filenames: - `idp_rsa_priv.key` - `idp_rsa_pub.key`

### ECC P-256 key pair using OpenSSL

Generate the private key:

``` bash
openssl ecparam -name prime256v1 -genkey -noout -out idp_ecc_priv.key
```

-   `ecparam`: works with elliptic-curve parameters.
-   `-name prime256v1`: selects the P-256 curve.
-   `-genkey`: generates a private key.
-   `-noout`: suppresses separate parameter output.
-   `-out`: saves the private key.

Extract the public key:

``` bash
openssl ec -in idp_ecc_priv.key -pubout -out idp_ecc_pub.key
```

Verify:

``` cmd
dir idp_ecc*
```

Expected filenames: - `idp_ecc_priv.key` - `idp_ecc_pub.key`

**Key protection:** Never share or commit private-key files to a public
repository. Protect them with suitable access controls and secure
storage.

## 3.5 RSA vs ECC

  Feature                         RSA-2048           ECC P-256
  ------------------------------- ------------------ -------------------
  Type                            Asymmetric         Asymmetric
  Nominal key size                2048 bits          256-bit curve
  Approximate security strength   112 bits           128 bits
  Public-key representation       Generally larger   Generally smaller
  Common JWT signing algorithm    RS256              ES256

ECC can provide strong security with smaller key material. Actual
performance depends on the platform and implementation.

**JWT terminology:** - RS256: RSA signature with SHA-256. - ES256: ECDSA
using P-256 and SHA-256.

## 3.6 Lab 3B: Hybrid-encryption simulation

The workbook's sample commands:

Create a sample payload:

``` bash
echo "SSN=000-12-3456,Salary=150000,Role=VP" > user_pii.txt
```

Generate a temporary random value and encrypt the payload:

``` bash
openssl rand -hex 32 > aes_key.txt
openssl enc -aes-256-cbc -salt -in user_pii.txt -out assertion.enc -pass file:aes_key.txt
```

Encrypt the key file using the recipient's RSA public key:

``` bash
openssl pkeyutl -encrypt -in aes_key.txt -pubin -inkey idp_rsa_pub.key -out encrypted_key.bin
```

Recover the key file using the corresponding private key:

``` bash
openssl pkeyutl -decrypt -in encrypted_key.bin -inkey idp_rsa_priv.key -out recovered_key.txt
```

Decrypt the payload:

``` bash
openssl enc -d -aes-256-cbc -in assertion.enc -out decrypted_user_pii.txt -pass file:recovered_key.txt
type decrypted_user_pii.txt
```

**Lab interpretation:** The exercise demonstrates the two-part idea:
protect the payload with symmetric encryption and protect the key
material with public-key cryptography.

**Practical caution:** These commands are a training simulation, not a
complete production encryption design. `openssl enc -pass` uses a
password-based key derivation process; it is not the same as directly
supplying a raw AES key. AES-CBC also does not provide authenticated
encryption by itself. Production systems should use an authenticated
design such as AES-GCM and carefully specified key wrapping.

------------------------------------------------------------------------

# Module 4: Digital Signatures, XMLDSig & JWT JWS

## 4.1 What is a digital signature?

A digital signature is a cryptographic mechanism used to verify: -
**Authenticity:** whether the data was signed by the holder of the
corresponding private key. - **Integrity:** whether the signed data has
changed.

A simplified signing process: 1. Calculate a digest of the data. 2. Sign
the digest using the signer's private key. 3. Send the data and
signature. 4. The receiver verifies the signature using the signer's
public key.

A digital signature does not automatically encrypt the message or hide
its contents.

## 4.2 Lab 4A: Sign and verify user claims with OpenSSL

Create a claims file:

``` bash
echo user=admin.smith,clearance=topsecret > claims.txt
```

Sign it using the RSA private key:

``` bash
openssl dgst -sha256 -sign idp_rsa_priv.key -out claims.sig claims.txt
```

-   `dgst -sha256`: calculates a SHA-256 digest as part of the signing
    operation.
-   `-sign`: signs using the private key.
-   `-out`: writes the signature to a file.

Verify using the public key:

``` bash
openssl dgst -sha256 -verify idp_rsa_pub.key -signature claims.sig claims.txt
```

Expected result:

`Verified OK`

### Tamper drill

Change the claims in `claims.txt`, for example from `admin.smith` to
`admin.jones`, and run the verification command again.

Expected result: verification failure.

**Reason:** The changed content no longer matches the data that was
signed.

## 4.3 XMLDSig and canonicalization (C14N)

SAML commonly uses XML Digital Signature (XMLDSig).

XML can have different textual formatting while representing similar
logical elements. Canonicalization (C14N) transforms XML into a
standardized representation before the relevant digest/signature
operations.

**Why it matters:** - Formatting changes can affect bytes and digest
calculations. - XMLDSig defines canonicalization rules so signer and
verifier process a consistent representation. - Do not casually reformat
or modify a signed SAML assertion.

The workbook highlights Exclusive XML Canonicalization (Exclusive C14N)
in SAML.

## 4.4 JWS

JWS means JSON Web Signature. A compact JWS has three Base64URL-encoded
parts:

`Header.Payload.Signature`

-   **Header:** identifies the signing algorithm and may include a key
    identifier (`kid`).
-   **Payload:** contains the claims.
-   **Signature:** allows the receiver to verify integrity and the
    signing key.

JWS provides integrity/authenticity, not confidentiality. A signed JWT
payload is generally readable unless it is separately encrypted.

------------------------------------------------------------------------

# Module 5: Ciphers, Cipher Suites & Hardening IAM Endpoints

## 5.1 What is a cipher?

A cipher is an algorithm used to perform encryption or decryption.

Examples: - AES: symmetric encryption. - RSA: asymmetric cryptography. -
ChaCha20: symmetric stream cipher.

## 5.2 What is a cipher suite?

A TLS cipher suite identifies cryptographic algorithms used by a TLS
connection. Depending on the TLS version, it describes the encryption
and integrity/protection algorithms and related cryptographic choices.

Modern TLS configurations prefer strong, modern algorithms and disable
obsolete options.

The workbook highlights: - ECDHE for ephemeral key agreement. - AES-GCM
for authenticated encryption. - Avoiding obsolete CBC cipher
configurations where appropriate.

## 5.3 ECDHE and Perfect Forward Secrecy

ECDHE stands for Elliptic Curve Diffie-Hellman Ephemeral.

It allows two parties to establish a shared secret over an open network
using ephemeral elliptic-curve key material.

**Perfect Forward Secrecy (PFS):** If a server's long-term private key
is compromised later, previously recorded sessions that used properly
generated ephemeral session keys should remain protected.

The important idea is that each session derives fresh key material
rather than relying only on the server's long-term private key to
protect every past session.

## 5.4 Lab 5A: Audit an enterprise endpoint with SSL Labs

1.  Open [SSL Labs SSL Server Test](https://www.ssllabs.com/ssltest/).
2.  Enter `login.microsoftonline.com`.
3.  Start the test.
4.  Review the supported TLS protocols and cipher suites.
5.  Look for modern key exchange and authenticated-encryption choices.

The workbook asks trainees to notice TLS_ECDHE suites using AES_GCM and
the disabling of obsolete CBC configurations.

**Purpose:** Learn how to inspect a public TLS endpoint's cryptographic
configuration.

------------------------------------------------------------------------

# Module 6: TLS 1.2 vs TLS 1.3 Handshake & Mutual TLS (mTLS)

## 6.1 What is TLS?

TLS (Transport Layer Security) is a protocol that protects data
exchanged between systems over a network.

TLS provides: - **Confidentiality:** encrypts communication. -
**Authentication:** helps verify the server's identity using
certificates. - **Integrity:** detects unauthorized changes to protected
traffic.

Examples include HTTPS, secure APIs, and LDAP over TLS.

## 6.2 TLS 1.2 vs TLS 1.3

The workbook compares handshake round trips: - TLS 1.2 commonly needs 2
round trips for a full handshake. - TLS 1.3 commonly completes its full
handshake in 1 round trip.

TLS 1.3 simplifies and improves the handshake, reducing connection setup
latency.

## 6.3 What is SNI?

SNI stands for Server Name Indication.

The client includes the target hostname in the initial TLS ClientHello.
This allows a server or reverse proxy hosting multiple domains on one IP
address to select the correct certificate.

## 6.4 What is mTLS?

mTLS stands for Mutual TLS.

-   In ordinary server-authenticated TLS, the client verifies the server
    certificate.
-   In mTLS, the server also requests and validates a client
    certificate.

This provides certificate-based authentication for both sides of the
connection.

Common uses include service-to-service communication and high-trust
enterprise APIs.

## 6.5 Lab 6A: Inspect TLS and SNI in Chrome

1.  Open Chrome.
2.  Navigate to <https://accounts.google.com>.
3.  Press `F12` to open Developer Tools.
4.  Select the **Security** tab.
5.  Review the connection information.
6.  Select **View Certificate**.
7.  Open the **Details** tab.
8.  Inspect the Subject Alternative Name (SAN).

The workbook's example review values are: - Protocol: TLS 1.3 - Key
exchange: X25519 - Cipher: AES_128_GCM

Live endpoint settings can change, so record what your browser actually
reports during the lab.

------------------------------------------------------------------------

# Module 7: PKI & X.509 Certificate Architecture

## 7.1 What is PKI?

PKI stands for Public Key Infrastructure. It is the framework of
policies, roles, processes, cryptographic keys, and certificates used to
establish and manage trust in public keys.

PKI includes: - Certificate Authorities (CAs) - Digital certificates -
Public and private keys - Certificate issuance and renewal - Certificate
validation and revocation

## 7.2 What is an X.509 certificate?

An X.509 certificate binds an identity (such as a DNS hostname or
organization) to a public key. It is digitally signed by a Certificate
Authority or, in some cases, self-signed.

## 7.3 Important X.509 extensions

  ------------------------------------------------------------------------
  Extension               Example value            Purpose
  ----------------------- ------------------------ -----------------------
  Subject Alternative     `DNS:sso.company.com`,   Identifies the
  Name (SAN)              `IP:10.0.5.50`           hostnames or addresses
                                                   covered by the
                                                   certificate

  Basic Constraints       `CA:FALSE`               Indicates that the
                                                   certificate is not a CA
                                                   certificate

  Key Usage               `digitalSignature`,      Restricts permitted
                          `keyEncipherment`        cryptographic uses

  Extended Key Usage      `serverAuth`,            Specifies intended
  (EKU)                   `clientAuth`             purposes such as TLS
                                                   server or client
                                                   authentication
  ------------------------------------------------------------------------

Modern hostname verification relies on SAN entries.

## 7.4 Three-tier CA chain

A typical enterprise chain is:

`Root CA → Intermediate CA → Leaf / Server Certificate`

-   **Root CA:** trust anchor; often protected offline.
-   **Intermediate CA:** issued and signed by the root, and can issue
    subordinate certificates within its permitted scope.
-   **Leaf certificate:** used by an endpoint such as an HTTPS server.

A verifier checks the certificate signatures and constraints up the
chain to a trusted root.

## 7.5 Lab 7B: Build a three-tier CA chain with OpenSSL

### Step 1: Create a root CA

``` bash
openssl genrsa -out rootCA.key 4096
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.crt -subj "/CN=EnterpriseRootCA"
```

-   Creates a 4096-bit RSA root private key.
-   Creates a self-signed root certificate valid for 3650 days.

### Step 2: Create an intermediate CA

``` bash
openssl genrsa -out intermediateCA.key 2048
openssl req -new -nodes -key intermediateCA.key -out intermediateCA.csr -subj "/CN=EnterpriseIntermediateCA"
openssl x509 -req -in intermediateCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out intermediateCA.crt -days 1825 -sha256
```

-   Generates an intermediate private key.
-   Creates a Certificate Signing Request (CSR).
-   Uses the root CA certificate and key to sign the intermediate
    certificate.

### Step 3: Create a server leaf certificate

``` bash
openssl genrsa -out sso_server.key 2048
openssl req -new -nodes -key sso_server.key -out sso_server.csr -subj "/CN=sso.training.local"
openssl x509 -req -in sso_server.csr -CA intermediateCA.crt -CAkey intermediateCA.key -CAcreateserial -out sso_server.crt -days 365 -sha256
```

-   Generates the server private key.
-   Creates a CSR.
-   Uses the intermediate CA to sign the server certificate.

### Step 4: Verify the chain

``` bash
openssl verify -CAfile rootCA.crt -untrusted intermediateCA.crt sso_server.crt
```

Expected output:

`sso_server.crt: OK`

**Important lab limitation:** These simplified commands demonstrate
signing relationships. A production CA configuration should include
appropriate CA extensions, path-length constraints, key-usage
extensions, and SAN values. A real server certificate also needs a SAN
matching the hostname clients use.

------------------------------------------------------------------------

# Module 8: Enterprise Certificate Management, Keystores & Zero-Downtime Rollover

## 8.1 What is a keystore?

A keystore stores private keys and their associated identity
certificates. It is used by an application or service that needs to
present its identity or perform signing operations.

## 8.2 What is a truststore?

A truststore contains certificates of trusted Certificate Authorities or
other trusted parties. It helps an application decide whether to trust a
certificate presented by another system.

  -----------------------------------------------------------------------
  Keystore                            Truststore
  ----------------------------------- -----------------------------------
  Holds private keys and identity     Holds trusted CA / public
  certificates                        certificates

  Used to present the system's        Used to validate another party's
  identity                            certificate

  Protects private-key material       Establishes trust anchors
  -----------------------------------------------------------------------

## 8.3 PKCS#12

PKCS#12 is a file format that can package private keys and certificates,
often protected with a password. Files commonly use `.p12` or `.pfx`.

## 8.4 Lab 8A: Export a PKCS#12 bundle

The workbook's command:

``` bash
openssl pkcs12 -export -out sso_complete_bundle.p12 -inkey sso_server.key -in sso_server.crt -certfile intermediateCA.crt
```

Explanation: - `pkcs12`: works with the PKCS#12 format. - `-export`:
creates an export bundle. - `-out`: names the resulting `.p12` file. -
`-inkey`: supplies the private key. - `-in`: supplies the leaf
certificate. - `-certfile`: adds the intermediate certificate.

OpenSSL will normally prompt for an export password.

## 8.5 Windows certificate stores

The workbook asks trainees to inspect: - `certmgr.msc`: Current User
certificate store. - `certlm.msc`: Local Machine certificate store.

These tools show certificates available to the user or machine. Services
may use different stores or application-specific keystores depending on
their configuration.

## 8.6 Zero-downtime SAML and OAuth signing certificate rollover

A signing certificate must sometimes be replaced before it expires. If
applications do not know about the new certificate when the IdP starts
using it, signature verification can fail.

The workbook's three-phase process:

**Phase 1 --- T-30 days** - Publish both the old certificate (A) and new
certificate (B) in federation metadata and/or JWKS. - Give relying
applications time to obtain and trust the new public key.

**Phase 2 --- Cutover day** - Change the IdP's active signing
configuration to certificate B. - Applications already have the new
public key available.

**Phase 3 --- T+30 days** - Remove the old certificate A after the
transition and any required overlap period.

**Why do this?** Publishing the new public key before switching the
signing key avoids a sudden trust gap for applications.

------------------------------------------------------------------------

# Module 9: Certificate Revocation, Pinning & High-Assurance Validation

## 9.1 Certificate revocation

A certificate may need to be invalidated before its expiry date. Reasons
can include: - The private key is compromised. - The certificate was
issued incorrectly. - The domain or identity is no longer controlled by
the certificate holder.

Clients need a way to check whether a certificate has been revoked.

## 9.2 OCSP

OCSP stands for Online Certificate Status Protocol.

It allows a client or another system to ask an OCSP responder about a
certificate's status.

Typical status responses include: - `good` - `revoked` - `unknown`

## 9.3 OCSP Stapling

With OCSP stapling: 1. The server obtains a signed OCSP status response.
2. The server caches it for its validity period. 3. The server sends
(staples) the response during the TLS handshake. 4. The client can
validate the response without making a separate OCSP request at that
moment.

Benefits: - Reduces client-side OCSP lookup latency. - Reduces direct
client requests to the OCSP responder. - Can improve privacy compared
with every client independently querying the CA's responder.

The workbook describes the server refreshing and caching the response
periodically. The exact refresh interval depends on the response's
validity and server configuration; it is not universally fixed at 24
hours.

## 9.4 Lab 9A: Inspect OCSP information with OpenSSL

Run:

``` bash
openssl s_client -connect login.microsoftonline.com:443 -status -tlsextdebug < /dev/null
```

Options: - `s_client`: starts a TLS client connection for testing. -
`-connect`: specifies the host and port. - `-status`: requests OCSP
stapling information. - `-tlsextdebug`: displays TLS extension
details. - `< /dev/null`: sends no additional input on systems that use
this shell convention.

Look for output such as: - `OCSP Response Status: successful (0x0)` -
`Cert Status: good`

**Important:** A server may not provide a stapled response in every
situation. Absence of a stapled response does not by itself prove that
the certificate is revoked. Interpret the result alongside the
certificate chain and the endpoint's configuration.

------------------------------------------------------------------------

# Module 10: JWT Cryptography --- JWS, JWE & JWKS

## 10.1 What is JWT?

JWT stands for JSON Web Token. It is a compact token format used to
carry claims between systems.

JWT is commonly used in authentication and authorization systems.

A signed JWT commonly has three Base64URL-separated sections:

`Header.Payload.Signature`

## 10.2 JWS

JWS stands for JSON Web Signature.

It provides a way to sign content so that a receiver can verify its
integrity and the signer's key.

A compact JWS contains: 1. Header 2. Payload 3. Signature

The header may include: - `alg`: signing algorithm, such as RS256 or
ES256. - `kid`: identifier for the key used to sign the token.

**Important:** A signed JWT is not necessarily encrypted. Its header and
payload can generally be decoded and read. Do not put secrets in an
ordinary signed JWT payload.

## 10.3 JWE

JWE stands for JSON Web Encryption.

It provides confidentiality by encrypting content so that only an
authorized recipient can decrypt it.

  -----------------------------------------------------------------------
  JWS                                 JWE
  ----------------------------------- -----------------------------------
  Provides signature / integrity and  Provides encryption /
  authenticity                        confidentiality

  Content is generally readable       Content is encrypted

  Common compact form has 3 parts     Compact form has 5 parts
  -----------------------------------------------------------------------

A system can use signing and encryption together when both
integrity/authenticity and confidentiality are required.

## 10.4 JWKS

JWKS stands for JSON Web Key Set.

It is a JSON document containing one or more public keys. Applications
use it to obtain public keys needed to verify signed tokens from an
Identity Provider.

A JWKS may contain fields such as: - `kty`: key type, for example RSA or
EC. - `use`: intended use, often `sig` for signature. - `kid`: key
identifier. - `alg`: algorithm associated with the key, when provided. -
`n`: RSA modulus. - `e`: RSA public exponent. - EC keys may use curve
and point parameters such as `crv`, `x`, and `y`.

## 10.5 What is `kid`?

`kid` means Key ID.

When a JWT is received: 1. The application reads the token header. 2. It
finds the `kid` value. 3. It searches the trusted JWKS for a matching
key ID. 4. It selects the corresponding public key. 5. It verifies the
token signature using that key and an allowed algorithm.

The application must validate the token's issuer, audience, expiry,
algorithm, and other required claims as well as its signature.

## 10.6 Lab 10A: Query a live Identity Provider JWKS endpoint

Open this URL in Chrome:

[Microsoft identity platform JWKS
endpoint](https://login.microsoftonline.com/common/discovery/v2.0/keys)

Steps: 1. Open the endpoint. 2. Inspect the `keys` array in the JSON
response. 3. Find a key's `kid`. 4. Compare it with the `kid` in a JWT
header issued by the same Identity Provider. 5. For an RSA key, inspect
`n` (modulus) and `e` (exponent). 6. Understand that the application
uses the matching public key to validate the JWT signature.

**Security note:** Use the issuer's trusted discovery metadata to
identify the correct JWKS endpoint. Do not trust an arbitrary key URL
supplied by an untrusted token.

------------------------------------------------------------------------

# Quick Revision: Important Terms

  Term      Full form / meaning
  --------- ---------------------------------------------------
  IAM       Identity and Access Management
  IdP       Identity Provider
  SP        Service Provider
  SAML      Security Assertion Markup Language
  XMLDSig   XML Digital Signature
  C14N      XML Canonicalization
  JWT       JSON Web Token
  JWS       JSON Web Signature
  JWE       JSON Web Encryption
  JWKS      JSON Web Key Set
  RSA       Rivest--Shamir--Adleman
  ECC       Elliptic Curve Cryptography
  AES       Advanced Encryption Standard
  TLS       Transport Layer Security
  mTLS      Mutual Transport Layer Security
  SNI       Server Name Indication
  PFS       Perfect Forward Secrecy
  PKI       Public Key Infrastructure
  CA        Certificate Authority
  CSR       Certificate Signing Request
  SAN       Subject Alternative Name
  EKU       Extended Key Usage
  OCSP      Online Certificate Status Protocol
  SSHA      Salted SHA-1
  P-256     256-bit elliptic curve also known as `prime256v1`

## Important ports mentioned in IAM practice

  Service                     Common port
  ------------------------- -------------
  HTTPS / TLS web traffic             443
  LDAP                                389
  LDAPS                               636

## Final concepts to remember

1.  **Encoding** changes representation; it does not provide secrecy.
2.  **Encryption** protects confidentiality and uses keys.
3.  **Hashing** creates a one-way digest.
4.  **Salting** makes password hashes harder to precompute and compare.
5.  **Symmetric encryption** is efficient for large data.
6.  **Asymmetric cryptography** uses a public/private key pair.
7.  **Hybrid encryption** combines symmetric encryption with asymmetric
    key protection.
8.  **Digital signatures** help establish authenticity and integrity.
9.  **TLS** protects data in transit.
10. **PKI** establishes and manages trust in public keys and
    certificates.
11. **Certificate rollover** should be planned to avoid service
    interruption.
12. **OCSP** provides certificate status information.
13. **JWS** signs content; **JWE** encrypts content.
14. **JWKS** publishes public keys that applications can use to verify
    tokens.

------------------------------------------------------------------------

**Scope:** These notes cover Workbook 2, Modules 1 through 10. Modules
11 and 12 are intentionally excluded.
