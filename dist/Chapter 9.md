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

# Design Issues

