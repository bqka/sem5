# PGP

# ⭐ **Pretty Good Privacy (PGP) — Exam-Ready Answer**

**Pretty Good Privacy (PGP)** is a widely used software package that provides **confidentiality, integrity, authentication, and email security**. It uses a **hybrid cryptographic system**, combining **symmetric-key encryption** for fast data encryption and **public-key cryptography** for secure key exchange. PGP is mainly used for **email protection, file encryption, and digital signatures**.

---

# ⭐ **How PGP Works (Hybrid Encryption)**

### **1. Symmetric-Key Cryptography**

- A random **session key (Ks)** is generated.
- The message is encrypted using Ks with algorithms like **CAST-128, IDEA, or 3DES**.
### **2. Public-Key Cryptography**

- The session key (Ks) is encrypted using the receiver’s **public key (PUb)**.
- Receiver decrypts Ks using their **private key (KPb)**.

This combines **speed** (symmetric) and **security** (asymmetric).

---

# ⭐ **Services Provided by PGP**

## **1. Authentication (Digital Signature)**

Authentication ensures the message is really from the sender and has not been altered.

**At the Sender:**

1. Message is hashed using SHA-1 → hash value.
2. Hash is encrypted using the sender’s **private key (KPa)** → digital signature.
3. Signature + message are compressed and sent.

**At the Receiver:**

1. Signature decrypted using sender’s **public key (PUa)**.
2. Receiver hashes the message again.
3. If both hashes match → sender is authenticated and message is intact.

---

## **2. Confidentiality**

Confidentiality ensures that only the intended receiver can read the message.

**At the Sender:**

1. Generate session key Ks.
2. Encrypt message using Ks (symmetric encryption).
3. Encrypt Ks using receiver’s **public key (PUb)**.
4. Send encrypted message + encrypted Ks.

**At the Receiver:**

1. Decrypt Ks using receiver’s **private key (KPb)**.
2. Decrypt message using Ks.
3. Decompress to recover original message.

---

## **3. Email Compatibility**

PGP converts binary data into ASCII (Radix-64 encoding) so that email systems can transmit it safely.

---

## **4. Segmentation**

Large messages are broken into smaller blocks so email systems can handle them.  
Receiver reassembles them.

---

# ⭐ **Advantages of PGP**

- Very strong encryption; extremely difficult to break.
- Provides confidentiality + authentication in a single system.
- Suitable for securing emails, files, backups, and cloud data.

---

# ⭐ **Disadvantages of PGP**

- Not very user-friendly; requires understanding of keys.
- Loss of private key makes encrypted data unrecoverable.
- Does not provide anonymity (sender/receiver are known).

---

# ⭐ **Final Short Summary**

PGP is a hybrid encryption system that combines symmetric and public-key cryptography to provide secure email communication. It offers **authentication** through digital signatures, **confidentiality** through encryption, **email compatibility**, and **segmentation**. PGP remains one of the most secure methods for protecting digital information.

# MIME

Multipurpose Internet Mail Extension (MIME) Protocol

![[Pasted image 20251124011453.png]]
![[Pasted image 20251124011522.png]]

![[Pasted image 20251124011600.png]]


# ⭐ **S/MIME (Secure/Multipurpose Internet Mail Extensions)**

**S/MIME** is a widely used standard for **secure email communication**.  
It provides **encryption**, **digital signatures**, and **authentication** for email messages.

S/MIME is built on two things:

1. **MIME** — allows rich email content (HTML, images, attachments)
2. **Public Key Cryptography** — provides encryption & signatures

It uses **X.509 certificates** for identity verification.

---

# ⭐ **Services Provided by S/MIME**

## ✅ 1. **Confidentiality**

Email content is encrypted using:

- A random **session key** (symmetric encryption), and
- The session key is encrypted with the receiver’s **public key**.

Only the receiver’s private key can decrypt it.

---

## ✅ 2. **Authentication & Non-Repudiation**

Sender signs the message using their **private key**.  
Receiver verifies signature with sender’s **public key** (from an X.509 certificate).

This ensures:
- Sender identity is verified
- Sender **cannot deny** sending the email
- Message integrity is preserved

---

## ✅ 3. **Integrity**

Hash of the message is signed.  
If the message is modified, signature verification fails.

---

## ⭐ **How S/MIME Works (Simple)**

### **Sender Side**

1. Hash the message (SHA).
2. Sign the hash using sender’s private key.
3. Generate session key & encrypt message.
4. Encrypt session key with receiver’s public key.
5. Send email containing:
    - Encrypted message
    - Encrypted session key
    - Digital signature

### **Receiver Side**

1. Decrypt session key using private key.
2. Decrypt message using session key.
3. Verify signature using sender’s public key.

---

# ⭐ **Advantages of S/MIME**

- Strong security (encryption + signature)
- Protects against spoofing and tampering
- Uses trusted X.509 certificates
- Widely supported (Outlook, Gmail for business, etc.)

---

# ⭐ **Disadvantages of S/MIME**

- Requires certificate installation
- Managing keys and certificates can be difficult
- No anonymity — sender identity is known

![[Pasted image 20251124020226.png]]

# ⭐ **Diffie–Hellman Key Exchange**

**Diffie–Hellman (DH)** is a method that allows two users to **securely generate a shared secret key** over an **insecure public channel**.  
It was proposed by **Whitfield Diffie and Martin Hellman in 1976**.

This shared secret key can then be used for **symmetric encryption** (AES, DES, etc.).

---

# ⭐ **Purpose of Diffie–Hellman**

- Allows two parties to **establish a common key** without actually sending the key.
- Prevents eavesdroppers from discovering the key even if they see all exchanged data.

DH itself **does not provide authentication** → vulnerable to MITM unless combined with signatures/certificates.

---



# ⭐ **Steps of Diffie–Hellman (Simple Explanation)**

### **1. Public parameters (known to everyone):**

- A **large prime number** p
    
- A **primitive root (generator)** g
    

These are NOT secret.

---

### **2. Each party picks a private key**

Alice chooses secret number:

a

Bob chooses secret number:

b

These are NEVER shared.

![[Pasted image 20251124012113.png]]

An attacker cannot compute this because finding ababab requires solving the **Discrete Logarithm Problem**, which is computationally hard.

![[Pasted image 20251124012213.png]]
# ⭐ **Advantages**

- Secure way to establish keys
    
- Key never transmitted
    
- Based on strong math (discrete logs)
    
- Forms basis for many modern protocols (TLS, SSH, IPsec)
    

---

# ⭐ **Disadvantages**

- No authentication → can be attacked by MITM
    
- Requires large primes for high security
    
- Vulnerable if parameters are weak

# X.509

![[Pasted image 20251124012604.png]]
![[Pasted image 20251124012617.png]]


![[Pasted image 20251124012736.png]]
# ⭐ 1. **How an X.509 Certificate is Created**

### **Step 1 — Entity generates key pair**

A user/server generates a **public–private key pair**.

### **Step 2 — Create CSR (Certificate Signing Request)**

The server/user sends a **CSR** to the CA containing:

- Public key
- Identity (domain name, organization name, etc.)
- Email
- Country
- Hash algorithm
- Signature using the private key (proves ownership)

### **Step 3 — CA verifies identity**

The Certificate Authority checks:

- Domain ownership
- Organization documents
- Legal identity
- Email verification  
    Depending on certificate type (DV/OV/EV).
### **Step 4 — CA signs the certificate**

The CA signs the certificate using its **private key**.

The certificate now contains:

- Subject identity
- Subject public key
- Validity period
- Serial number
- CA digital signature
- Optional extensions (key usage, SAN, etc.)

---

# ⭐ 2. **How an X.509 Certificate is Used in Communication**

Example: **HTTPS (TLS)**

1. Client connects to server.
2. Server sends its **X.509 certificate** to the client.
3. Client performs **signature verification**:
    - Uses CA’s **public key** (stored in the browser)
    - Verifies the CA’s signature on the certificate
4. If valid:
    - Client trusts the server’s **public key**
    - Establishes encrypted session (e.g., using Diffie–Hellman or RSA key exchange)

If verification fails → browser shows “Untrusted Certificate”.

---

# ⭐ 3. **Certificate Chain (Chain of Trust)**

X.509 uses a **hierarchical trust model**.

### Structure:

1. **Root CA**
    - Self-signed
    - Stored in browsers/OS by default
    - Very secure; rarely used directly
2. **Intermediate CA**
    - Signed by Root CA
    - Issues certificates to servers/users
    - Used to reduce exposure of root key
3. **End-Entity Certificate**
    - Issued to the actual user/server
    - Used in TLS (HTTPS)
# ⭐ 4. **Certificate Revocation**

Certificates may become invalid before expiry, due to:

- Private key compromise
- CA error
- Organization changes
- Misuse

X.509 supports two revocation mechanisms:

---

# ⭐ A. **CRL — Certificate Revocation List**

A CRL is a **digitally signed list of revoked certificates** published by the CA.

- The list contains serial numbers of revoked certificates.
- Clients download CRL from the CA’s “CRL Distribution Point (CDP)”.
- Client checks if the certificate’s serial number is inside the CRL.
### **Problems with CRL**

- Large downloads
- Not real-time
- Slow to update

---

# ⭐ B. **OCSP — Online Certificate Status Protocol**

OCSP is a **real-time revocation checking method**.

Is certificate X revoked?

OCSP responder replies:

    GOOD
    REVOKED
    UNKNOWN

Benefits:
    Fast
    Up-to-date
    Lightweight

OCSP Stapling (Improvement)
Server attaches (“staples”) OCSP response during TLS handshake →
Client does not need to contact CA.

# VPN

A VPN (Virtual Private Network) is a security technology that encrypts your internet traffic and creates a secure tunnel between your device and the internet. It hides your IP address, routes data through remote servers, prevents tracking by hackers or ISPs, and allows you to access restricted content while maintaining online privacy and anonymity.

![[Pasted image 20251124021020.png]]

![[Pasted image 20251124021202.png]]
![[Pasted image 20251124021248.png]]

# Firewall

A firewall is a network security system, available as hardware or software, that monitors and controls incoming and outgoing traffic based on predefined rules. It acts like a security guard, filtering data packets to either:

- ****Accept:**** Allow the traffic.
- ****Reject:**** Block with an error response.
- ****Drop:**** Block silently without response.

![[Pasted image 20251124021436.png]]

![[Pasted image 20251124021528.png]]

Here is a **clear, simple, exam-ready explanation** of **IPSec**.

---

# ⭐ **IPSec (Internet Protocol Security)**

**IPSec** is a suite of protocols used to **secure IP communication** over public or private networks.  
It provides **Confidentiality, Integrity, Authentication, and Anti-Replay protection** at the **Network Layer (Layer 3)**.

It is widely used in **VPNs**, especially **Site-to-Site VPNs** and **Remote Access VPNs**.

---

# ⭐ **Main Services Provided by IPSec**

### ✔ **1. Confidentiality**

Encrypts data packets (using AES, 3DES).

### ✔ **2. Integrity**

Ensures data is not modified (using HMAC-SHA).

### ✔ **3. Authentication**

Authenticates sender using:

- Digital signatures
- Pre-Shared Keys (PSK)
- Certificates (X.509)

### ✔ **4. Anti-Replay Protection**

Prevents attacker from re-using old packets.  
Uses sequence numbers.

---

# ⭐ **Two Main IPSec Protocols**

## 🔵 **1. AH (Authentication Header)**

- Provides **authentication & integrity**
- **Does NOT provide encryption**
- Protects the entire packet except mutable fields

Used when encryption is NOT required.

---

## 🔵 **2. ESP (Encapsulating Security Payload)**

- Provides **encryption, authentication, integrity**
- Most commonly used in VPNs
- Can encrypt only payload or entire packet (in Tunnel mode)

---

# ⭐ **Modes of IPSec**

IPSec works in **two modes**:

## 🔵 **1. Transport Mode**

- ONLY the **payload** (data) is encrypted/authenticated
- IP header is NOT encrypted
- Used in **end-to-end** communication (host ↔ host)

```
| IP Header | Encrypted Data |
```

---

## 🔵 **2. Tunnel Mode**

- Entire **original IP packet** is encrypted
- A **new IP header** is added
- Used in **VPNs (gateway ↔ gateway)**

```
| New IP Header | Encrypted (Old IP Header + Data) |
```

Tunnel mode is the most common configuration for IPSec VPNs.

---

# ⭐ **IPSec Key Exchange – IKE**

IPSec uses **IKE (Internet Key Exchange)** to:

- Negotiate security parameters
- Authenticate peers
- Exchange secret keys
- Establish Security Associations (SAs)

IKE has two versions:

- **IKEv1** (older)
- **IKEv2** (newer, faster, more secure)

---

# ⭐ **Security Association (SA)**

An **SA** is a one-way secure connection between two parties.

Each SA defines:

- Encryption algorithm
- Hash algorithm
- Key lifetime
- Mode (Tunnel/Transport)

Two SAs are required for full communication:

- One for inbound
- One for outbound

---

# ⭐ **Where IPSec is Used**

- Site-to-Site VPN
- Remote Access VPN
- Secure corporate WANs
- Secure communication between routers/servers
- IPv6 mandatory security support

---

# ⭐ **Advantages of IPSec**

- Very strong encryption and authentication
- Works at Layer 3 → transparent to applications
- Protects all IP traffic
- Forms basis of secure VPNs

---

# ⭐ **Disadvantages**

- Complex to configure
- Higher overhead → slower
- Compatibility issues with NAT (solved with NAT-Traversal)

---

# ⭐ **Exam-Ready Summary**

**IPSec is a network-layer security protocol suite that provides confidentiality, integrity, authentication, and anti-replay protection for IP traffic. It uses AH and ESP protocols, operates in transport or tunnel mode, and relies on IKE for key exchange and Security Associations to secure VPN communication.**

![[Pasted image 20251124023610.png]]

