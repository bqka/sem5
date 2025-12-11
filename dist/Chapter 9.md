# Index

## Introduction
### Dependability & Security Goals
- Dependability: availability, reliability, safety, maintainability  
- Added requirements: confidentiality & integrity  
- Four threat types:
  - Interception
  - Interruption
  - Modification
  - Fabrication (incl. replay)

### Security Policy & Mechanisms
- Security policy defines allowed/prohibited actions  
- Mechanisms:
  - Encryption
  - Authentication
  - Authorization
  - Auditing  

---

## Design Issues
### Focus of Control
- Protect data directly (integrity constraints)  
- Control access to operations (method/interface-based)  
- Control access by user roles (RBAC)  

### Layering of Security Mechanisms
- Security vs. trust distinction  
- Security can be applied at:
  - Network/link layer  
  - Transport layer (SSL)  
  - Middleware (secure RPC)  
  - Application  
- Higher layers depend on trust in lower layers  

### Distribution of Security Mechanisms
- Trusted Computing Base (TCB): all components enforcing policy  
- Goal: minimize TCB size  
- Using trusted servers & RISSC approach  
- Microkernel advantage for isolating trusted components  

### Simplicity
- Keep core mechanisms simple & auditable  
- Some applications (e.g., digital payments) require complexity  
- Need simple underlying cryptographic primitives  

---

## Cryptography
### Attack Types Defended by Encryption
- Interception  
- Modification  
- Fabrication  
- Note: Encryption alone does not stop interruption  

### Common Cryptographic Tools
- DES  
- RSA  
- MD5  

---

## Secure Channels
### Requirements
- Authentication  
- Message integrity  
- Confidentiality  
- Protection against interception / modification / fabrication  
- Still vulnerable to interruption (DoS)

---

## Authentication
### Authentication Based on Shared Secret Key
- Challenge–response mutual authentication  
- Five-message protocol  
- Three-message “optimized” version → reflection attack  
- Lessons: avoid symmetric challenge patterns, verify peer before sending sensitive data  

### Authentication Using a KDC
- Avoids O(N²) secret keys  
- Session key generation  
- Needham–Schroeder shared-key protocol (with nonces)  
- Replay vulnerabilities & fixes  
- Improved version binds request to Bob’s nonce  

### Authentication Using Public-Key Cryptography
- Mutual authentication using encrypted challenges  
- Session key included in responder’s message  
- Requires trustworthy distribution of public keys  

---

## Message Integrity & Confidentiality
### Digital Signatures
- Prevent message alteration  
- Prevent repudiation  
- Signing message digests instead of full messages  
- Signature verification via public key  
- Issues: stolen keys, key changes, timestamping  

### Session Keys
- Reduced long-term key usage  
- Replay protection  
- Limits scope of key compromise  
- Easier trust management  

---

## Secure Group Communication
### Confidential Group Communication
- Single shared key → simple but risky  
- Pairwise keys → secure but O(N²) overhead  
- Public-key model → scalable  

### Secure Replicated Servers
- Active replication with signed responses  
- Client validates results using threshold signatures (c+1)  
- Servers pre-verify & forward a single validated response  
- Threshold schemes ensure correctness despite up to c faulty servers  

---

## Access Control
### General Issues
- Subjects, objects, operations  
- Reference monitor enforces access control  
- Must be tamperproof  

### Access Control Matrix
- Conceptual model  
- Implemented via:
  - Access Control Lists (ACLs)  
  - Capabilities (unforgeable tokens)  

### Protection Domains
- Grouping subjects (groups, roles)  
- Grouping objects (interfaces, subtyping)  
- Certificate-based membership  

---

## Firewalls
### Purpose
- Protect internal network from outside traffic  
- Acts as perimeter reference monitor  

### Types
#### Packet-Filtering Gateway
- Filters based on packet headers  
- Supports simple blocking rules  
- Used for egress/ingress security  

#### Application-Level Gateway
- Inspects actual message content  
- Mail gateways, Web proxies  
- Can block dangerous scripts/applications  

---

## Secure Mobile Code
### Protection Challenges
- Protecting mobile agents from hostile hosts  
- Protecting hosts from malicious agents  

### Protecting the Agent (Ajanta Examples)
- Read-only signed state  
- Append-only secure logs  
- Selective encrypted state for designated servers  

### Protecting the Host
#### Sandbox Model
- Code executed in restricted environment  
- Interpreted languages simplify sandboxing  

#### Java Security Model
- Class loaders (trusted loaders only)  
- Bytecode verifier  
- Security manager (runtime checks)  

#### Playground Model
- Execute mobile code on isolated machine  

#### Authentication-Based Access Control
- Code signing  
- Object references as capabilities  
- Stack introspection  
- Namespace management via class loaders  

---

## Denial of Service (DoS)
### Attack Types
- Bandwidth depletion  
- Resource depletion (e.g., TCP SYN flood)  

### Defenses
- Host-based monitoring  
- Egress filtering  
- Ingress filtering  
- ISP-level filtering (traffic ratio heuristics)  

---

## Security Management
### Key Management
- Key establishment (Diffie–Hellman)  
- Key distribution challenges  
- Public-key certificates & certificate chains  

### Certificate Lifetimes
- CRLs  
- Expiration-based lifetimes  
- Short or near-zero lifetimes (high overhead)  
- Real-world issue: many clients ignore CRLs  

---

## Secure Group Management
### Secure Group Membership
- Group keys: communication key + public/private key pair  
- Join protocol:
  - JR (join request) with RP + Kp,G + certificate  
  - Q authenticates P  
  - GA (group admittance) includes encrypted CKG & KG⁻  
  - P confirms via encrypted nonce  
- Reply pad (RP) avoids long-term compromise issues  

---

## Authorization Management
### Capabilities & Attribute Certificates
- Amoeba capability structure (port, object ID, rights, check field)  
- One-way functions ensure tamperproof rights  
- Restricted capabilities using rights masks  
- Attribute certificates generalize capabilities  

### Delegation
#### Basic Problems
- Need temporary transfer of rights  
- Simple solutions insufficient (named/bearer certificates)  

#### Neuman’s Delegation Scheme
- Proxy = certificate + secret  
- Certificate signed by delegator  
- Secret enables challenge-response verification  
- Supports further delegation without contacting originator  
- Ensures authenticity + unforgeability  


# Introduction

Security in computer systems is closely tied to **dependability**, which means we can justifiably trust a system to deliver its services. Traditionally, dependability includes **availability, reliability, safety, and maintainability**, but for true trustworthiness, **confidentiality** and **integrity** must also be included.
- **Confidentiality** ensures information is accessible only to authorized parties.
- **Integrity** ensures system assets (hardware, software, data) can be altered only in authorized ways and that improper changes are detectable and recoverable.

Security aims to protect a system's services and data from **four main types of threats**:
1. **Interception** – Unauthorized access to data or services (e.g., eavesdropping on communication, breaking into private files).
2. **Interruption** – Making data or services unavailable or destroyed (e.g., file corruption, denial-of-service attacks).
3. **Modification** – Unauthorized alteration of data or services (e.g., tampering with transmitted data, changing database entries, modifying programs).
4. **Fabrication** – Creating fake data or activities (e.g., adding fake entries to a password file, replaying old messages).  
    Interruption, modification, and fabrication are all forms of **data falsification**.

Creating a secure system requires defining a **security policy**, which clearly states what actions are allowed or prohibited for all system entities (users, services, machines, data, etc.). Once a policy is in place, **security mechanisms** are used to enforce it. Key mechanisms include:
5. **Encryption** – Protects confidentiality by making data unreadable to attackers; also supports integrity checking.
6. **Authentication** – Verifies the identity of users or systems (commonly via passwords but also other methods).
7. **Authorization** – Determines whether an authenticated entity is allowed to perform specific actions (e.g., accessing or modifying medical records).
8. **Auditing** – Tracks who accessed what and how; useful for analyzing breaches, though it doesn’t prevent attacks. Attackers often try to avoid leaving audit trails.

Overall, security policies and mechanisms work together to guard systems against threats while ensuring trustworthy operation.

## Design Issues

### Focus of Control

When protecting a (possibly distributed) application, there are **three main approaches**:

1. **Protect the data directly.**  
    This approach focuses on ensuring **data integrity** regardless of which operations are performed on the data. It is common in database systems, where **integrity constraints** are defined and automatically checked whenever data is modified.
2. **Control access to operations.**  
    Here the emphasis is on specifying **which operations can be executed, by whom, and under what conditions**. This is closely tied to **access control mechanisms**. In object-based systems, permissions may be set per method, per interface, or for an entire object, allowing flexible levels of access granularity.
3. **Control access based on users (roles).**  
    This approach restricts access based on **who the user is**, independent of what operations they intend to perform. For example, only authorized bank staff may access certain databases, or only faculty and staff may access certain university systems. This leads to **role-based access control (RBAC)**, where user roles are defined and access decisions are made once the role is verified.

In designing secure systems, defining appropriate roles and providing mechanisms to enforce role-based access control is essential.

![[Pasted image 20251204032612.png|600]]

### Layering of Security Mechanisms

Designing secure systems also involves deciding **which system layer** should contain the security mechanisms. Systems—especially distributed systems—are organized into **layers**, such as applications, middleware, OS services, and the kernel, alongside network layers. Understanding these layers is important because **security and trust operate differently across them**.

![[Pasted image 20251204032947.png|600]]

A key distinction is that **security** is a technical property (a system is secure or not), whereas **trust** is an emotional judgment made by the client. Where you place security mechanisms depends on **which layers the client trusts**.

For example, consider several organizational sites connected by a wide-area backbone service such as **SMDS**. Security could be implemented at the network link level by placing **encryption devices** at each SMDS router. This protects intersite communication, but Alice (at site A) must **trust** that these devices are secure and properly maintained by system administrators.

![[Pasted image 20251204033102.png|500]]

If Alice does **not** trust the link-level security, she can use a **transport-layer security mechanism**, such as **SSL**, which encrypts communication over a TCP connection. In this case, her trust moves from the network layer to the transport layer.

If she does not trust SSL either, she could rely on **middleware-level security**, such as a secure RPC system. But trust in this middleware security depends on all underlying layers. For example, if the RPC service internally uses SSL and Alice does not trust SSL, then she cannot trust the RPC service either.

Overall, **security mechanisms at higher layers depend on the trustworthiness of the lower layers they rely on**, making trust a crucial factor in deciding where to implement security.
### Distribution of Security Mechanisms

Dependencies between services introduce the concept of a **Trusted Computing Base (TCB)**. The TCB consists of all security mechanisms in a system—distributed or not—that are required to enforce the security policy and therefore **must be trusted**. A key design goal is to **keep the TCB as small as possible**.

In distributed systems built on top of existing network operating systems, the TCB may include the **local operating systems** running on each host. For example, a distributed file server depends on the local OS for protecting its files and ensuring the server itself cannot be maliciously shut down.

If the middleware does **not trust** the underlying operating systems, some OS functionalities may need to be moved into the distributed system. In a microkernel OS, this is easier: services like the file system can be entirely replaced with customized secure components designed specifically for the distributed system.

A related strategy is to **separate critical security services from regular services** by running them on dedicated, trusted machines. For example, in a secure distributed file system, the file server might run on a highly trusted operating system, while clients run on less trusted machines. This reduces the TCB to only a small set of machines and software components.

This idea underlies the **Reduced Interfaces for Secure System Components (RISSC)** approach. In RISSC, any security-critical server runs on an isolated machine protected by low-level secure network interfaces. Clients access the server only through these constrained interfaces, reducing exposure and increasing overall trust in the system.

![[Pasted image 20251204033717.png]]
### Simplicity

Another major design concern when choosing where to place security mechanisms is **simplicity**. Secure systems are difficult to design, so using **simple, well-understood, and trustworthy mechanisms** is highly desirable.

However, simple mechanisms are often not enough. For example, link-level encryption can protect against interception between sites, but if Alice wants to ensure that **only Bob** receives her message, she needs more complex **user-level authentication services**. This requires understanding concepts like **cryptographic keys** and **certificates**, even if much of the process is automated and hidden from users.

Sometimes, the **application domain itself is complex**, and adding security increases this complexity even further. A key example is **digital payment systems**, where multiple parties interact and complex security protocols are necessary. In such cases, it becomes crucial that the **underlying security mechanisms** remain as simple and understandable as possible.

Overall, simplicity increases user trust and helps system designers reason more confidently about the system’s security, reducing the chance of hidden vulnerabilities.

## Cryptography

![[Pasted image 20251204033926.png]]

When transferring a message as ciphertext, there are **three major types of attacks** that encryption helps defend against:
1. **Interception (Eavesdropping).**  
    An intruder may secretly capture the message. If the message is properly encrypted, interception becomes useless because the attacker can only see unreadable ciphertext. (Although, in some situations, the mere _fact_ that communication is happening can reveal information, such as sudden traffic pattern changes.)
2. **Modification of the message.**  
    It’s easy to alter plaintext, but altering **encrypted** data is much harder. An attacker would need to decrypt the ciphertext first, change the message in a meaningful way, and then re-encrypt it correctly so the receiver does not detect tampering.
3. **Fabrication (inserting fake messages).**  
    An attacker might inject their own encrypted messages to trick the receiver into believing they came from the real sender. Encryption—combined with proper authentication methods—helps prevent or detect such insertion attacks. Also, the ability to modify messages generally implies the ability to insert them.

Overall, encryption provides strong protection against interception, modification, and fabrication when properly implemented.

![[Pasted image 20251204034202.png]]

- DES
- RSA
- MD5

# Secure Channels

Distributed systems are often organized using the **client–server model**, where servers may themselves act as clients to other servers. When considering security in such systems, two major issues arise:

1. **Securing communication between clients and servers**
To make client-server interactions secure, the system must provide:
- **Authentication** of both communicating parties
- **Message integrity** to ensure messages aren’t altered
- **Confidentiality** to prevent eavesdropping, when needed

This applies not only to client–server communication but also to communication **between servers within a group**.

These protections are achieved by establishing a **secure channel**.  
A secure channel defends against:
- **Interception** → via confidentiality (encryption)
- **Modification** → via integrity checks
- **Fabrication** → via mutual authentication protocols

However, a secure channel does _not_ necessarily protect against **interruption** (e.g., denial-of-service attacks).
The chapter then goes on to examine authentication protocols using both **symmetric-key** and **public-key** cryptography, with message confidentiality and integrity discussed separately.

2. **Authorization**
Once a server accepts a client’s request, it must determine whether the client is **allowed** to perform the requested action. This is the authorization problem and ties directly to **access control**, which is explored in the next section.

## Authentication

![[Pasted image 20251204035625.png]]
![[Pasted image 20251204035602.png]]

### Authentication based on shared secret key

![[Pasted image 20251204035704.png|400]]

In authentication protocols that rely on a **shared secret key**, Alice and Bob prove their identities by encrypting each other’s challenges. In the standard five-message protocol, Alice first identifies herself, Bob sends a random challenge Rb​, Alice returns the encrypted version Ka,b(Rb), and then Alice challenges Bob in return. When each party correctly encrypts the other’s challenge using the shared key, mutual authentication is achieved.

However, simplifying the protocol to reduce it from **five messages to three** introduces vulnerabilities. In the “optimized” version, Alice sends her identity and challenge together, and Bob responds with both his challenge and the encrypted reply. This redesigned protocol is vulnerable to a **reflection attack**.

![[Pasted image 20251204040451.png|500]]

In a reflection attack, an intruder (Chuck) pretends to be Alice and initiates communication with Bob. When Bob sends back his challenge and the encrypted response, Chuck cannot encrypt Bob’s challenge himself, so he starts a **second, fake session** with Bob—again pretending to be Alice—and sends Bob’s own challenge back to him. Because the protocol allows identical behavior in different runs, Bob unknowingly encrypts his own challenge and hands the answer to Chuck. Chuck then uses that encrypted value to complete the _first_ session, making Bob believe he is talking to Alice.

![[Pasted image 20251204040726.png|500]]

The core problem is that both parties used **the same type of challenge in both roles**, making the protocol symmetrical and exploitable. A safer design requires the initiator and responder to use **different kinds of challenges** (e.g., odd vs. even numbers), which would have allowed Bob to detect the attack. However, even such fixes may be vulnerable to other attacks (such as man-in-the-middle attacks).

Another design mistake in the flawed protocol is that Bob sends sensitive encrypted data without first verifying who he is talking to—something the original protocol avoided. This highlights a broader lesson: **cryptographic protocols are difficult to design correctly**, and small performance tweaks can easily break correctness. Developers have learned various design principles over time, many of which emphasize avoiding symmetry, avoiding unnecessary disclosure, and carefully ordering authentication steps.

### Authentication using Key Distribution Center

Using shared secret keys for authentication does not scale well. If a distributed system has N hosts, each pair of hosts would need a separate secret key, meaning the system must manage N(N−1)/2 keys. Each host must store N−1 keys, which becomes impractical as N grows. To solve this, systems often use a **Key Distribution Center (KDC)**. The KDC shares exactly one secret key with each host, reducing the total number of required keys to only N.

![[Pasted image 20251204195538.png|500]]

When Alice wants to communicate securely with Bob, she asks the KDC for help. The KDC generates a **session key** for Alice and Bob to use. It sends this key to:
- Alice, encrypted with the key Alice shares with the KDC
- Bob (indirectly), by giving Alice a **ticket** that contains the same session key encrypted with the key Bob shares with the KDC

![[Pasted image 20251204195618.png|500]]

Alice then forwards the ticket to Bob. Since only Bob and the KDC know Bob’s secret key, only Bob can decrypt the ticket and obtain the session key. This lets Alice and Bob communicate without needing a permanent shared key.

This idea forms the basis of the well-known **Needham–Schroeder authentication protocol**, which uses a challenge-response design. Here’s how it works in its standard form:
1. Alice sends a request to the KDC, including a **nonce** (a random number used once). Nonces prevent replay attacks and ensure the response matches the request.
2. The KDC responds with the session key and a ticket for Bob. It includes Alice’s nonce again so that Alice can verify the response is fresh and not a replayed old message.
3. Alice sends Bob the ticket along with a new challenge, encrypted with the session key.
4. After Bob decrypts the ticket and obtains the session key, he responds by manipulating Alice’s challenge to prove he actually decrypted it, and he includes his own challenge.
5. Alice responds to Bob’s challenge, completing mutual authentication.

![[Pasted image 20251204195705.png|500]]

Nonces are crucial. Without them, an attacker like Chuck could replay old responses from the KDC. For example, if Chuck once stole an old ticket encrypted with an old key, he could trick Alice or Bob by replaying old messages. The use of nonces makes this impossible, because an old response will contain a nonce that does not match the current request.

Including Bob’s identity in the KDC’s response is another safeguard. Without it, Chuck could modify Alice’s original request by replacing Bob’s identity with his own. The KDC would then generate a key for Alice and Chuck, and Chuck could intercept later messages and impersonate Bob. Including identities ensures Alice can detect tampering.

Even with these protections, the Needham–Schroeder protocol still had a weakness: if Chuck ever obtains an old session key, he can replay message 3 and trick Bob into believing Alice is starting a session again. A fix is to make the session key depend on the **original request**. To do this, Alice first asks Bob for a nonce, encrypted with Bob’s long-term key. She then includes this nonce in her request to the KDC. The KDC decrypts the nonce and includes it in Bob’s ticket. When Bob later sees the nonce inside the ticket, he knows that the session key is tied to the genuine request from Alice.

![[Pasted image 20251204200126.png|500]]

These refinements illustrate how challenging it is to design secure authentication protocols. Small changes made for convenience or performance can easily introduce vulnerabilities, and secure protocol design requires careful attention to replay prevention, identity binding, and message freshness.

### Authentication Using Public-Key Cryptography

Public-key cryptography can also be used for authentication without relying on a Key Distribution Center. Suppose Alice wants to establish a secure channel with Bob, and both already know each other’s public keys.

The protocol works as follows:
1. **Alice authenticates Bob first.**  
    She sends Bob a challenge (a random number) encrypted with Bob’s public key. Because only Bob has the matching private key, only he can decrypt the challenge. If he returns the correct value, Alice knows she is talking to the real Bob.  
    This requires Alice to be sure she is using Bob’s _real_ public key, not one supplied by an attacker.
2. **Bob authenticates Alice and creates a session key.**  
    Bob decrypts Alice’s challenge and sends it back, proving his identity. He also sends his own challenge to authenticate Alice and includes a newly generated session key for future communication.  
    This entire message is encrypted with **Alice’s** public key, ensuring that only Alice can read it.
3. **Alice proves she is Alice.**  
    After decrypting Bob’s message using her private key, Alice responds correctly to Bob’s challenge. She sends this response using the newly created session key, proving she was able to decrypt Bob’s message — and therefore confirming her identity.

![[Pasted image 20251204200524.png|500]]

This protocol achieves mutual authentication and establishes a shared session key, all without needing a KDC.

## Message Integrity and Confidentiality

Message integrity ensures that messages cannot be secretly altered, while confidentiality ensures that messages cannot be intercepted and read by unauthorized parties.

### Digital Signatures

Message integrity is important even after a message is delivered. For example, if Alice emails Bob agreeing to buy a record for $500, two major concerns arise:
1. **Alice must be protected from Bob altering the message**, such as changing $500 to a higher price.
2. **Bob must be protected from Alice denying she ever sent the message** (repudiation).

Both issues can be solved if Alice **digitally signs** her message in a way that uniquely binds her signature to the message content. If the message changes, the signature becomes invalid. And if Alice’s signature verifies correctly, she cannot later deny having sent it.

A common way to sign messages is to use **public-key cryptography**. Alice can encrypt the message (or part of it) with her **private key**, and Bob can verify it using her **public key**. This proves the message came from Alice and was not altered. If confidentiality is also needed, the message can additionally be encrypted with Bob’s public key.

![[Pasted image 20251204203012.png]]

However, several practical issues arise:
- **If Alice’s private key is stolen**, she can falsely claim a valid signature was forged.
- **If Alice changes her key**, old signatures may no longer be valid unless a trusted authority tracks key changes and timestamps.
- **Encrypting an entire message with a private key is expensive and unnecessary.**

A more efficient method is to use a **message digest** (hash). Alice computes a fixed-length digest of the message using a cryptographic hash function, then encrypts only the digest with her private key. She sends the original message plus the encrypted digest.

When Bob receives them, he:
1. Decrypts the digest using Alice’s public key.
2. Computes his own digest of the received message.
3. Compares the two digests.

![[Pasted image 20251204203125.png]]

If they match, Bob knows the message is authentic and unchanged.
This provides message integrity, authentication, and protection against repudiation.

### Session Keys

After two parties authenticate each other and establish a secure channel, they typically generate a **unique shared session key** to protect the confidentiality of their communication. This key is used only for that session and discarded afterward. Although they _could_ reuse the long-term keys used during authentication, using separate session keys provides several important benefits.

##### 1. Reduces the risk of key exposure
Reusing the same key frequently makes it easier for an attacker to uncover it by analyzing large amounts of encrypted data. Long-term authentication keys must be protected carefully and are often exchanged through slow, secure methods (like phone or mail). Minimizing their use reduces the chances of compromise.

##### 2. Protects against replay attacks
A fresh session key ensures that old sessions cannot be replayed in full. To prevent replay of individual messages, extra mechanisms like timestamps or sequence numbers may be used.

##### 3. Limits damage if a key is compromised
If a long-term key were used for confidentiality, anyone who obtained it later could decrypt **past conversations**. With session keys, compromising a key only exposes **one session**, and all other sessions remain secure.

##### 4. Allows selective trust
Alice may want confidentiality when talking to Bob but may not trust him enough to use long-term, highly sensitive keys. A temporary session key is safer for less-trusted communication partners.

##### **Overall**
Long-term authentication keys are expensive and difficult to replace, while session keys are cheap, short-lived, and provide better security properties. Therefore, combining permanent authentication keys with temporary per-session keys is the preferred approach for secure communication.

## Secure Group Communication

### Confidential Group Communication

When protecting communication within a group of **N users**, confidentiality becomes more complicated than in two-party communication.

##### 1. Single shared secret key
A simple approach is for all group members to share **one common secret key** used for encrypting and decrypting group messages.  
However:
- Every member must be fully trusted to keep the key secret.
- If _any_ member leaks the key, the entire group’s communication is compromised.
This makes the single-key method relatively weak and risky.

##### 2. Pairwise shared keys
Another solution is to use a **different shared key for every pair of members**.  
This allows the group to cut off any untrustworthy member by stopping communication with them, while other pairs remain secure.
But the downside is significant:
- The group must manage **N(N−1)/2** keys, which becomes impractical as the group grows.

##### 3. Using public-key cryptography
A more scalable approach is for each member to have their own **public–private key pair**.  
Benefits:
- Only **N key pairs** must be managed (one per user).
- Anyone can send encrypted messages to a member using that member’s public key.
- If a member becomes untrustworthy, they can simply be removed without affecting other members’ keys.

### Secure Replicated Servers

When a client sends a request to a **group of replicated servers**, the client expects a trustworthy response, even if some servers may be faulty or maliciously compromised. Simply collecting all server replies and checking for a majority is one solution, but it exposes the fact that replication is happening, breaking replication transparency.

#### Secret sharing for secure, replicated servers

Reiter and colleagues propose a method that keeps replication hidden while still protecting against corrupted servers. This approach is based on **secret sharing** — splitting a secret so that no single participant holds the entire secret, and reconstruction requires cooperation from multiple participants.

The goal:  
If up to **c** out of **N** servers are corrupted, the system should still produce a correct, trustworthy response.

#### How the method works

1. **Active replication:**  
    Every server receives the client’s request and sends back a response **rᵢ** along with:
    - a message digest md(rᵢ)
    - a digital signature created with the server’s private key
2. The client receives **all N signed responses**, but it cannot trust any individual server.
3. Instead of verifying signatures one by one, the client uses a **public decryption function D** that takes a set of **c + 1 signatures** and produces a digest.  
    If the result matches the digest of one server’s response, that response must have been supported by at least **c + 1 honest servers**, ensuring correctness.
    
    For example, if N = 5 and c = 2, the client tests combinations of 3 signatures at a time (10 possible combinations).
4. **Improving transparency:**  
    To hide replication from the client, servers send their signed responses to each other.  
    When a server receives at least **c + 1 signed responses**, it tries to compute a valid digest using D.  
    If successful, it forwards **only one response and its signature set** to the client.  
    The client verifies the response by checking that md(r) = D(V).

![[Pasted image 20251204204802.png]]

### **Threshold scheme interpretation**

This approach is an example of an **(m, n)-threshold scheme**, where:
- **n = N** (number of servers)
- **m = c + 1** (minimum signatures needed to validate a response)

In such schemes:
- Any **m** "shadows" (pieces) can reconstruct the valid signature.
- Any **m − 1** or fewer shadows provide no useful information.

Threshold schemes are a general method for securely distributing trust across multiple parties.

# Access Control

In the client–server model, once a secure channel is established, a client can send requests to a server to perform operations on resources the server controls. These resources often take the form of **objects** managed by an object server, and each client request typically involves invoking a method on a specific object. The server must verify that the client has the **appropriate access rights** to perform the requested operation.

This leads to two closely related concepts:
- **Authorization** – granting access rights
- **Access control** – enforcing those rights when requests are made

The terms are often used interchangeably.

There are many approaches to implementing access control, ranging from general-purpose models to specialized mechanisms such as **firewalls**. With modern systems that include **code mobility**, traditional access control methods became insufficient, leading to new techniques that are also discussed later in the chapter.

## General Issues in Access Control

To understand access control, a simple model is used:
- **Subjects** (typically processes acting on behalf of users) attempt to access
- **Objects** (resources encapsulated with their own state and operations)

Objects expose operations through interfaces, and subjects invoke those operations to perform tasks.

![[Pasted image 20251204210214.png|500]]

Role of reference monitor:
Access control is enforced by a **reference monitor**, a trusted software component that:
1. Keeps track of which subjects are allowed to perform which operations
2. Checks every invocation to ensure the request is permitted
Because the reference monitor is central to security, it must be **tamperproof**. If an attacker compromises the reference monitor, the entire access control system becomes unreliable.

### Access Control Matrix

A common way to model which subjects can perform which operations on which objects is the **access control matrix**.
- Each **row** represents a subject.
- Each **column** represents an object.
- Each entry M\[s, o] lists the operations subject _s_ is allowed to perform on object _o_.  
    The reference monitor checks this matrix whenever a subject invokes an operation.

##### Why the matrix is not implemented literally
In real systems with thousands of users and millions of objects, the matrix would be enormous and mostly empty. Storing it as a full matrix is impractical. Instead, systems use more efficient representations:

###### 1. Access Control Lists (ACLs)

The matrix is stored **column-wise**:
- Each object maintains a list (ACL) of subjects and their allowed operations.
- Empty matrix entries are simply omitted.
This means the server checks _“Do I know this client, and is this client allowed to perform this operation?”_  

###### 2. Capabilities

The matrix is stored **row-wise**:
- Each subject carries a list of capabilities, where each capability specifies the operations allowed on a particular object.
- If a subject lacks a capability for an object, it has no access rights.

A capability works like a **ticket** granting specific rights.  
Because subjects must not modify their own tickets, capabilities are typically **protected**, for example by attaching a cryptographic signature. Distributed systems like **Amoeba** make heavy use of signed capabilities.

In the capability model:
- The server does **not** check who the client is.
- The server only checks whether the capability presented is valid and whether it authorizes the requested operation.  

![[Pasted image 20251205004012.png|500]]

**Summary:**
- **ACLs**: Objects hold lists of who can access them.
- **Capabilities**: Subjects hold unforgeable tokens describing what they can access.

Both represent the same logical access control matrix but distribute it differently for efficiency and practicality.

### Protection Domains

ACLs and capabilities reduce the size of an access control matrix by eliminating empty entries, but lists can still grow large. To manage this complexity, systems often introduce **protection domains**.

A protection domain is a set of _(object, access rights)_ pairs. Each request is made within a specific domain, and the reference monitor checks whether the requested operation is allowed in that domain.

Protection domains can be implemented in several ways:
##### **1. Groups of Users**

Instead of listing every individual user in an ACL, users can be organized into groups.

Example:  
A company intranet page is accessible to all employees.  
Instead of listing thousands of users in the ACL, the page’s ACL simply lists the group **Employee**.

To check access, the system only verifies whether the user belongs to that group.

##### **Hierarchical Groups**

Groups can be organized in a hierarchy—for example:
- Employee
    - Employee.AMS
    - Employee.NYC
    - Employee.SF

![[Pasted image 20251205005320.png|500]]

This allows:
- Broad permissions (all employees can read intranet pages)
- Narrow permissions (only Amsterdam employees can modify Amsterdam pages)

The downside is that checking membership in a distributed group hierarchy may be expensive.

##### **Certificates Instead of Lookups**

To avoid complex distributed lookups, users can carry **certificates** listing their group memberships. These certificates are digitally signed so they cannot be forged. This shifts the burden from the reference monitor to the user.

##### **2. Roles as Protection Domains (Role-Based Access Control)**

Protection domains can also be modeled as **roles**.

A user logs in under a specific role—e.g., department head, project manager, or committee member. Each role has its own permissions.

Users may need to **switch roles** during a session (e.g., Dick switches from department head to project manager). This flexibility is harder to achieve when using groups alone.

##### **3. Grouping Objects by Interface**

Objects themselves can also be grouped to simplify access checks.
Instead of specifying rights for each individual object, objects are grouped by:
- The operations they support
- Their interfaces (with subtyping or interface inheritance)
When a subject invokes an operation, the reference monitor checks whether the subject can perform operations belonging to that **interface**, not necessarily that specific object.

#### **Combining Techniques**

Both **user-domain grouping** and **object-interface grouping** can be combined to scale ACLs effectively. Using these ideas, researchers have shown how to manage ACLs for massive object collections, such as in digital libraries.

## Firewalls

Up to this point, protection in distributed systems has relied on cryptographic techniques and access control matrices. These methods work well when all participants follow the same rules—such as within a self-contained distributed system. However, when **external users** interact with the system (e.g., sending email, downloading files, uploading documents), internal mechanisms alone are not enough.

To protect internal resources from outsiders, systems use a **firewall**, a specialized reference monitor that sits between the distributed system and the outside world. All incoming and outgoing traffic passes through the firewall, which inspects and filters packets, blocking anything unauthorized. Because it is the main barrier to external threats, the firewall must itself be extremely secure and reliable.

Two main types of firewalls:

![[Pasted image 20251205010113.png|500]]
##### **1. Packet-Filtering Gateway**

- Acts like a router that examines **packet headers** (source and destination addresses).
- Decides whether to allow or drop packets.
- Often used to block:
    - Incoming packets addressed to protected internal servers
    - Outgoing packets that should not leave the internal network

Packet filters can also be configured across multiple LANs (e.g., connected through SMDS) to allow only traffic originating from trusted internal networks, effectively creating a private virtual network.

##### **2. Application-Level Gateway**

- Inspects the **actual content** of messages, not just headers.
- Examples:
    - **Mail gateways** that block oversized emails or filter spam.
    - **Library gateways** that let external users access only document abstracts while requiring payment for full copies.

A special case is the **proxy gateway**, which acts as a controlled front end for a specific application.

Example:  
A **Web proxy** appears as a normal Web server to users, but filters pages and requests. It can block executable scripts or applets to prevent untrusted code from entering the internal network.
## Secure Mobile Code

Modern distributed systems support **mobile code**, meaning that programs—not just data—can move between hosts. While powerful, this capability introduces major security concerns.

##### **Two main security problems arise:**

1. **Protecting mobile agents from hostile hosts**  
    When an agent travels across the Internet, its owner wants to ensure that the host receiving it cannot steal or alter the sensitive information it carries.
2. **Protecting hosts from malicious mobile code**  
    Users often download programs without fully understanding their behavior. Even experts may not realize that a program is being downloaded or executed.  
    Without security measures, a malicious program running on a host can easily corrupt files, steal data, or damage the system.

This becomes an **access control problem**: a mobile program must not be allowed to access resources it should not use. The goal is not to prevent all code from being downloaded—mobile code is often needed—but to allow it to run **safely**, with **controlled and limited access** to local resources.

### Protecting the Agent

Before discussing how to protect hosts from malicious mobile code, the text examines the opposite problem: **protecting a mobile agent from hostile hosts**.

### **Why mobile agents need protection**

A mobile agent may travel between hosts to perform tasks—for example, searching for the cheapest flight and carrying an electronic credit card to complete a purchase. This introduces risks:
- A host might **steal the agent’s sensitive data** (like credit card information).
- A host might **modify the agent’s behavior** to benefit itself (e.g., preventing the agent from visiting a cheaper competitor).
- A host might **destroy or corrupt the agent**, or alter it so that it harms its owner when it returns.

##### **Complete protection is impossible**

Research shows that it is **impossible to fully protect a mobile agent** from all types of attacks by a hostile host. This is because a host ultimately has control over what code runs on its machine, so there is no absolute guarantee it will behave correctly.

##### **Partial protection: Detecting tampering (Ajanta system)**

The Ajanta mobile agent system takes a practical approach:  
**It cannot prevent tampering, but it ensures tampering is detectable.**

Ajanta provides three mechanisms:
##### **1. Read-only state**
- The agent carries certain data in a read-only section.
- The owner signs this data (via a message digest encrypted with the owner’s private key) before the agent departs.
- Any host that receives the agent can verify the signature to check for unauthorized modifications.

##### **2. Secure append-only logs**

These logs allow hosts to add information but prevent deleting or altering previous entries without detection.

Process:
- The log begins empty with an initial checksum.
- When a host appends data, it signs its contribution and produces a new checksum based on:
    - the previous checksum
    - the new signed data
    - the host’s identity
When the agent returns home, the owner verifies the log backwards by repeatedly checking signatures and checksums. Any mismatch reveals tampering.

##### **3. Selective revealing of state**

- The agent carries an array of data items, each encrypted with the public key of a specific server.
- Only the designated server can decrypt its own entry.
- The entire array is signed by the agent's owner to detect modifications.
This prevents hosts from viewing or tampering with information not intended for them.

##### **Host protection**

Ajanta also includes mechanisms to protect hosts from malicious agents, similar to other mobile-code systems. These are discussed in the following section.

### Protecting the Target

While protecting mobile agents from malicious hosts is difficult, **protecting hosts from malicious mobile code is even more critical**. Users may choose not to send agents out into the world, but they cannot avoid receiving code from outside if they rely on mobile applications. If a malicious agent enters the system, simply detecting damage afterward is not enough—the system must **prevent unauthorized access in advance**.

#### **Sandboxing: Protecting Hosts from Malicious Code**

A **sandbox** confines a downloaded program to a controlled environment where **every instruction is monitored**.  
A sandbox stops execution if the program tries to:
- perform forbidden instructions
- access unauthorized memory or registers
- interact with restricted system resources
Implementing a sandbox for compiled binaries is difficult, but becomes much easier for **interpreted code** such as Java.

#### **Java’s Sandbox Model**

Java uses several components to safely run mobile code:
##### **1. Trusted Class Loaders**
- Java code is downloaded by **class loaders**.
- Downloaded programs are **not allowed** to use their own class loaders, preventing them from bypassing loading rules.

![[Pasted image 20251205014731.png]]

##### **2. Bytecode Verifier**
- Checks downloaded classes for:
    - illegal instructions
    - violations of Java’s safety rules
    - stack or memory corruption attempts

Only **downloaded** classes are verified.

##### **3. Security Manager**
- Enforces runtime restrictions.
- Acts as a **reference monitor**:
    - Denies access to files
    - Prevents arbitrary network connections
    - Blocks attempts to manipulate the JVM
- Allows harmless actions like graphics operations or mouse event handling.

![[Pasted image 20251205014847.png]]

![[Pasted image 20251205014426.png]]

Originally, Java used one strict security policy for all downloaded programs, but this was too restrictive, so more flexible approaches were developed.
#### **Beyond the Sandbox: The Playground Model**

A **playground** is a separate machine dedicated to running mobile code.
- Downloaded programs have access to **only the playground’s** resources.
- They cannot access other machines or their files.
- Other users interact with the playground via standard mechanisms such as RPC.

This provides strong isolation but requires extra hardware.

![[Pasted image 20251205014345.png|500]]

#### **Authentication-Based Access Control (Code Signing)**

Instead of sandboxing everything, downloaded code can be **digitally signed**. Only trusted sources are allowed to run code on the host. The challenge is to enforce different **security policies** depending on the source.

Wallach et al. propose three mechanisms for Java:
##### **1. Object References as Capabilities**
- A program can access a resource only if it is explicitly given the **object reference** for that resource.
- If a program is not given a file-handling object, it **cannot access the file system**.
- Java’s type safety prevents programs from inventing their own references.

![[Pasted image 20251205015113.png|500]]

##### **2. Stack Introspection**

Used to enforce privileges during method calls.
- Before a method of a local resource is executed, the JVM runs **enable_privilege** to check permissions.
- When done, **disable_privilege** removes these permissions.
- Java automatically inserts these checks, preventing developers or malicious code from bypassing them.

![[Pasted image 20251205015421.png]]

![[Pasted image 20251205015332.png|400]]
	
Stack introspection also supports checking **call chains**.  
If a trusted object is invoked by an **untrusted caller**, the system denies access even if the trusted object normally has permission.

![[Pasted image 20251205015745.png]]

##### **3. Namespace Management**

Access to resources requires including certain class files.  
Java can **map the same class name to different implementations** depending on where code came from.  
Thus, untrusted programs may be given **safe stub versions** of classes instead of real ones.

This mechanism is implemented using adapted class loaders.
##### **Language Dependence**

These techniques rely heavily on Java's design and JVM behavior. Implementing similar protections in other languages is harder and may require:
- A secure operating system
- Kernel-level mediation of all accesses to local resources

This allows language-independent enforcement but is significantly more complex.

## Denial of Service

Access control aims to ensure that **authorized processes** can use system resources. However, a major related threat is when attackers deliberately **block authorized users** from accessing those resources. This kind of attack is known as a **denial-of-service (DoS)** attack. As distributed systems increasingly connect to the Internet, defending against DoS attacks has become critical.

The challenge becomes far greater with **distributed denial-of-service (DDoS)** attacks. In a DDoS attack, a large number of compromised machines—often hijacked without their owners’ knowledge—work together to overwhelm a target service.

#### **Two main types of DDoS attacks:**

1. **Bandwidth depletion attacks**
    - Attackers flood a target with huge volumes of traffic.
    - Legitimate messages cannot get through.
2. **Resource depletion attacks**
    - Attackers force the target to waste resources on useless or incomplete requests.
    - Example: **TCP SYN flooding**, where the attacker starts large numbers of TCP handshakes but never completes them, causing the server to hold many half-open connections.

#### **Defense challenges and strategies**

There is no single perfect solution. Compromised machines used in attacks are often unsuspecting victims, so systems must monitor their state to detect malware—but viruses spread too easily for this to be a reliable defense alone.

More effective strategies include:
- **Monitoring outbound traffic (egress filtering)**
    - Drop packets with forged source addresses not belonging to the organization.
    - Helps contain attacks originating from within compromised machines.
- **Monitoring inbound traffic (ingress filtering)**
    - Less effective alone, because by the time traffic reaches the organization, the attack may already be overwhelming.
- **Filtering at ISP-level routers**
    - Dropping packets early in the network helps prevent congestion.
    - Example: If the ratio of packets _to_ a node greatly exceeds packets _from_ that node, the router may suspect an attack and begin dropping traffic.

In practice, **multiple defenses** must be used together, and attackers constantly develop new techniques. Several studies provide comprehensive overviews and taxonomies of modern DDoS threats and countermeasures.

# Security Management

## Key Management

![[Pasted image 20251205021711.png]]

### Key Establishment

To set up a secure channel, Alice and Bob need a **session key**. Traditional methods—using public keys, shared secret keys, or a KDC—require that some secure key distribution already exists. To avoid this dependency, they can use the **Diffie–Hellman key exchange**, which creates a shared secret over an insecure network.

### Diffie–Hellman in brief
1. Alice and Bob publicly agree on large numbers **n** and **g**, which attackers may also know.
2. Alice chooses a private value **x**; Bob chooses a private value **y**.
3. Alice sends **gˣ mod n** to Bob; Bob sends **gʸ mod n** to Alice.
4. Each computes the shared key:
    - Alice computes (gʸ mod n)ˣ = gˣʸ mod n
    - Bob computes (gˣ mod n)ʸ = gˣʸ mod n

Both arrive at the **same secret key** without revealing x or y. The security relies on the difficulty of determining x from gˣ mod n (the discrete logarithm problem).

Diffie–Hellman can be viewed as a public-key method, but it still requires that public values be distributed securely to prevent impersonation.

![[Pasted image 20251205021740.png|500]]

### Key Distribution

A major challenge in key management is the **initial distribution of keys**.

#### **1. Distributing symmetric (shared secret) keys**

For symmetric cryptosystems, the initial shared secret key must be sent over a channel that provides both **authentication** and **confidentiality**.  
If Alice and Bob have no existing secure channel or keys, the key must be exchanged **out-of-band**—e.g., via phone, physical mail, or another trusted method.

#### **2. Distributing public keys**

In public-key cryptosystems, the public key can be sent in plaintext, but it must be received **authentically**—the receiver must be sure the public key truly belongs to the claimed entity.  
The corresponding **private key** must be distributed securely (confidential + authenticated).

![[Pasted image 20251205023911.png|500]]
#### **3. Public-key certificates**

To provide authenticated distribution of public keys, systems use **public-key certificates**.
A certificate contains:
- a **public key**
- an **identifier** (e.g., user, host, device)
- a **digital signature** from a Certification Authority (CA)
The CA signs the (public key + identifier) using its private key **K_CA**.  
Its public key **K_CA⁺** is widely known—for example, major CA public keys are built directly into Web browsers.

#### **How a certificate is verified**
A client:
1. Retrieves the certificate.
2. Uses the CA’s public key to verify the CA’s signature.
3. If the signature matches, the client accepts that the public key belongs to the stated entity.
This process relies on **trusting the CA itself**. If the client is unsure whether the CA’s public key is genuine, it may confirm it using another certificate, forming a **chain of certificates**.

#### **4. Hierarchical trust models**
Public-key infrastructures often rely on **hierarchical trust chains**.
Example: **PEM (Privacy Enhanced Mail)**
- Lowest-level CAs are authenticated by **Policy Certification Authorities (PCAs)**
- PCAs are authenticated by the **Internet Policy Registration Authority (IPRA)**
- If a user does not trust IPRA, they cannot trust the entire certificate chain.

Other such trust models exist and are discussed in standard cryptography literature.

### Lifetime of Certificates

A key issue in certificate-based security is **how long certificates remain valid**. If a certification authority (CA) issues certificates that never expire, this becomes dangerous: if an entity’s private key is ever compromised, attackers could continue using the corresponding public key indefinitely. To prevent this, certificates must be revocable.

##### **1. Certificate Revocation Lists (CRLs)**

One common revocation method is for the CA to publish a **CRL**, listing certificates that are no longer valid.

When a client checks a certificate, it must also consult the latest CRL.  
Drawbacks include:
- Delay: A certificate is still usable until the next CRL is published.
- Overhead: Clients must retrieve CRLs regularly.
- The shorter the publication interval, the more load on the system.

##### **2. Limited certificate lifetimes**

Instead of lifelong certificates, CAs issue certificates with **expiration dates**, similar to leases.  
Benefits:
- Certificates naturally become invalid after some time.
- CRLs are still needed for early revocation, but their load is reduced.
Clients must still check the CRL to confirm a certificate hasn’t been revoked before expiration.
##### **3. Extreme approach: near-zero lifetime**

Certificates can be made to expire almost immediately.  
This forces clients to contact the CA **every time** they need to verify a key, making the CA effectively always online.  
This approach is highly secure but impractical due to heavy load and high latency.

#### **Practical reality**

Most Internet certificates have long lifetimes (e.g., up to one year).  
Although CRLs should be checked regularly, **many client applications do not check them at all**, trusting certificates until they expire. This creates security weaknesses in real-world systems.
## Secure Group Management

Security systems such as **Key Distribution Centers (KDCs)** and **Certification Authorities (CAs)** must be both **highly trusted** and **highly available**.
- If a CA is compromised, the entire public-key infrastructure becomes useless.
- If a KDC becomes unavailable, new secure channels cannot be established.
To achieve high availability, these services are often **replicated**. However, replication increases the attack surface, so the group of replicated servers must itself be managed securely.

![[Pasted image 20251205025705.png]]
### **Secure Group Membership**
![[Pasted image 20251205025554.png]]
A group **G** maintains:
- A **shared communication key** CKG (for encrypting messages among group members).
- A **public/private key pair** (KG⁺, KG⁻) for communication with outsiders.

When a new process **P** wants to join the group:

![[Pasted image 20251205025650.png|500]]
##### **1. P sends a Join Request (JR)**

The message includes:
- The identity of P and the group G
- A timestamp **T**
- A **reply pad (RP)** — a one-time key
- A **secret key Kp,G** generated by P
- All encrypted with the group’s public key KG⁺
- Plus P's **certificate** and **signature**
The RP and Kp,G allow secure transfer of keys later.

##### **2. A group member Q receives JR**
Q must:
1. **Authenticate P** using P’s certificate.
2. Verify **T** to ensure the certificate was valid at the time of sending.
3. Consult other group members to decide whether P is allowed to join.
##### **3. If P is admitted**
Q sends a **Group Admittance (GA)** message containing:
- P’s identity
- A nonce **N**
- The group communication key **CKG**, encrypted with RP
- The group’s private key KG⁻ encrypted with CKG
- The entire message signed by Q using Kp,G

Because only a true group member can decrypt JR and obtain Kp,G, P can authenticate that the GA message truly comes from a legitimate group member.
##### **4. P confirms**
P returns the nonce N encrypted with Kp,G, proving it received all necessary keys and is officially part of the group.

#### **Why use the Reply Pad (RP)?**

Instead of encrypting CKG with P’s public key, the protocol uses a **one-time key (RP)**:
- RP is used only once—safer against key compromise.
- If P’s private key is ever stolen later, past group keys **cannot** be revealed, unlike with public-key encryption.

This protocol ensures that:
- Only authenticated, approved members join the secure group
- Group secrets are safely distributed
- Replicated servers can maintain both **availability** and **security**
## Authorization Management

### Capabilities and Attribute Certificates

A more effective method of enforcing access control in distributed systems is the use of **capabilities**—unforgeable data structures that specify the exact access rights a process has for a resource. The Amoeba distributed operating system uses a well-known capability-based scheme.

![[Pasted image 20251205042209.png]]

#### **Capabilities in Amoeba**

![[Pasted image 20251205042218.png]]

A capability in Amoeba is a **128-bit value** consisting of four fields:
1. **Server Port (48 bits)**
    - Identifies the object's server.
    - Used to locate the server via broadcasting.
2. **Object Identifier (24 bits)**
    - Identifies the specific object at that server.
    - Together with the server port, uniquely identifies the object across the system.
3. **Rights Field (8 bits)**
    - Lists the access rights allowed (e.g., read, write, delete).
    - Meaning varies depending on object type.
4. **Check Field (48 bits)**
    - Ensures the capability is _unforgeable_.
    - Created using a random number and a one-way function.

#### **How Capabilities Are Created and Used**

- When an object is created, the server chooses a **random check value**, stores it internally, and includes it in the capability given to the client.
- Clients present the capability when invoking operations; the server verifies the check field to ensure the capability has not been tampered with.

#### **Generating Restricted Capabilities**

Clients can request “reduced” versions of their capabilities (e.g., granting fewer rights to another user). The steps:
1. Client sends the original capability plus a **rights mask** (subset of allowed rights).
2. The server:
    - Takes the original check value.
    - XORs it with the new rights.
    - Passes the result through a **one-way function**.
    - Places this output in the check field of the new capability.

![[Pasted image 20251205043132.png|500]]

This ensures:
- **Extra rights cannot be added** by modifying a capability—the check field would not match.
- Because the one-way function cannot be reversed, the capability remains **tamper-proof**.

#### **Attribute Certificates (Generalization of Capabilities)**

Modern distributed systems sometimes use **attribute certificates**, which:
- Store (attribute, value) pairs describing permissions or roles.
- Are signed by an **Attribute Certification Authority (ACA)**.
- Do **not** have to be issued by the same server that controls the resource, unlike Amoeba’s model.

![[Pasted image 20251205043359.png]]

Attribute certificates expand the concept of capabilities beyond Amoeba by enabling decentralized, flexible assignment of access rights.
### Delegation

Delegation allows a process to temporarily pass its access rights to another process. This is useful in distributed systems—such as when a user wants a print server to print a file later—even though the server normally lacks permission to read the user's file.

#### **Why Delegation Is Needed**

If a user instructs a server to perform an action on a file they own (e.g., printing), the server often **does not have the required access rights**, which leads to permission errors. Delegation solves this by letting the user temporarily grant rights to the server.

#### **Simple Delegation Ideas**

Two basic, but limited, approaches:
1. **Named delegation:**  
    Alice creates a certificate:  
    **“Alice says Bob has rights R”**  
    But Bob must contact Alice again to pass rights to others.
2. **Bearer certificate:**  
    Alice creates:  
    **“Whoever holds this certificate has rights R.”**  
    But this must be protected against copying.

These approaches are not flexible enough for distributed systems where processes operate across different machines.
#### **Neuman’s General Delegation Scheme**

Neuman introduces **proxies**—secure tokens that allow their holder to act with some subset of another subject’s rights.

![[Pasted image 20251205043920.png]]

A proxy consists of two parts:

![[Pasted image 20251205043853.png|500]]
##### **1. Certificate (public part)**
Contains:
- **R**: the rights being delegated
- **Sₚᵣₒₓʸ (public part of a secret)**
- **Signature by Alice**: `sig(A, C)`
    - Prevents any modification

This certificate can be passed freely between processes.

##### **2. Secret (private part)**
- **Sₚᵣₒₓʸ' (private secret)**
- Only the legitimate holder knows it
- This lets the holder _prove_ the certificate belongs to them

#### **How Delegation Works**

If Alice wants to delegate rights to Bob:
1. She sends Bob:
    - The signed certificate `[R, Sₚᵣₒₓʸ]ᴬ` (in plaintext)
    - The private secret encrypted using the shared key `K_AB(Sₚᵣₒₓʸ')`
2. Bob now holds a proxy proving Alice delegated rights to him.

Bob can further delegate rights to Charlie or Dave without contacting Alice—he simply passes the certificate and the secret.

![[Pasted image 20251205044044.png|500]]

#### **How Rights Are Exercised**

When Bob invokes an operation at a server:
1. Bob sends the certificate `[R, Sₚᵣₒₓʸ]ᴬ`.
    - The server verifies the signature to ensure no tampering.
2. The server must verify Bob is the legitimate holder.
    - It uses **Sₚᵣₒₓʸ (public challenge)** to test whether Bob knows the secret **Sₚᵣₒₓʸ'**.

Example implementation:
- Treat Sₚᵣₒₓʸ as a **public key** and Sₚᵣₒₓʸ' as its **private key**.
- Server sends a nonce encrypted with Sₚᵣₒₓʸ.
- Bob decrypts it using Sₚᵣₒₓʸ' and returns the nonce.
- This proves Bob is the rightful holder.
#### **Key Insight**

Delegation requires **two things**:
1. **Tamper-proof rights list**
    - Achieved via Alice’s signature
2. **Proof of legitimate ownership**
    - Achieved via the secret (challenge–response)

Neuman’s scheme achieves flexible, secure delegation without requiring the delegator (Alice) to know all future recipients or be involved in further delegation steps.