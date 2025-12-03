Names are essential in computer systems because they allow users and processes to identify, share, and access resources. A key requirement is _name resolution_, the process of mapping a name to the entity it refers to. Implementing naming systems in **distributed systems** is more complex than in centralized systems because the naming system itself is distributed across multiple machines. The way this distribution is organized affects both scalability and efficiency.

The chapter focuses on three major uses of names in distributed systems:

1. **Human-Friendly Names**  
    These include names in distributed file systems and the World Wide Web. Since such systems must operate on a global scale, designing scalable naming systems is a major challenge.
2. **Location-Independent Naming**  
    Some names must identify entities regardless of their physical or network location (e.g., mobile devices). Traditional human-friendly naming systems do not support this well, so alternative structures—such as those used in mobile telephony and distributed hash tables—are needed.
3. **Descriptive Naming**  
    Humans often describe entities using attributes rather than fixed identifiers. Resolving such descriptions to actual entities is difficult, requiring specialized support.

# Names, Identifiers and Addresses

A **name** in a distributed system is a string of characters or bits used to refer to an **entity**, which can be anything from hardware resources (printers, hosts, disks) to software resources (files, processes, users, web pages, messages, etc.). Entities provide operations through an **interface**; to access these operations, a process must use an **access point**.

An **address** is the name of an access point. For example, a server’s address may consist of an IP address and a port number. Entities can have multiple access points—similar to how a person can have several phone numbers.

Access points can change over time: mobile devices get new IP addresses, people change phone numbers or email addresses, and servers may move to different machines. Because of this, using an address _as the name of the entity_ is inflexible and unreliable. If the address changes or is reassigned, the reference becomes invalid.

To avoid these problems, distributed systems use **location-independent names**, which do not depend on the entity’s current address. This allows entities to move or change access points without breaking references. Location-independent naming is more flexible, scalable, and user-friendly.

Besides addresses, distributed systems use other types of names—most importantly **identifiers**, which uniquely and permanently refer to entities. A **true identifier** has three key properties:

1. It refers to **at most one** entity.
2. Each entity has **at most one** identifier.
3. It **never changes or gets reused**; it always refers to the same entity.

These properties make identifiers useful for unambiguous referencing. For example, if two processes compare identifiers, equality guarantees they refer to the same entity. This would not work with normal names like “John Smith,” or with addresses that can be reassigned (e.g., phone numbers).

**Addresses** and **identifiers** are both machine-oriented names, typically represented as bit strings (e.g., Ethernet addresses, memory addresses). They serve different purposes: addresses refer to _access points_, while identifiers refer to _entities themselves_.

A third category is **human-friendly names**, which are designed for human use and represented as character strings. Examples include UNIX filenames and DNS domain names. These names are readable and flexible but do not provide guaranteed uniqueness or location information.

![[Pasted image 20251201010136.png]]

# Flat Naming

Identifiers are convenient to uniquely represent entities. In many cases, identifiers are simply random bit strings. which we con veniently refer to as unstructured, or flat names. An important property of such a name is that it does not contain any information whatsoever on how to locate the access point of its associated entity.

## Simple Solutions

### Broadcasting and Multicasting

Both solutions are applicable only to local-area networks. Nevertheless, in that environment, they often do the job well, making their simplicity particularly attractive.

In distributed systems with efficient broadcasting (such as LANs or wireless LANs), locating an entity is simple: a request containing the entity’s identifier is broadcast to all machines, and only the machine that hosts the entity responds with its address. This method is used by **ARP**, where a computer broadcasts a message asking which machine owns a given IP address, and the owner replies with its data-link (e.g., Ethernet) address.

However, broadcasting becomes inefficient as the network grows, wasting bandwidth and interrupting many unnecessary hosts. To improve scalability, systems use **multicasting**, in which only a selected group of hosts receives the request. Networks like Ethernet support multicasting at the hardware level, and the Internet supports network-level multicasting through multicast groups identified by multicast addresses.

Multicast addresses can serve as a general location mechanism. For example, mobile computers can join a multicast group when they connect; a “where is A?” query sent to the group allows computer A to respond with its current IP address. Multicasting can also help locate replicated entities. When a request is sent to a multicast address, each replica responds with its IP address, and a simple (though imperfect) method to choose the nearest replica is to select the first reply received.

### Forwarding Pointers

Forwarding pointers are a method for locating mobile entities. When an entity moves from location A to location B, it leaves a pointer at A that references its new location. This makes the approach simple: once a client finds the entity using a naming service, it can follow the chain of pointers to discover the entity’s current address.

However, the method has significant drawbacks:

- The chain of pointers can become very long for highly mobile entities, making lookups slow and expensive.
- All intermediate locations must maintain their part of the chain as long as needed, increasing overhead.
- The approach is vulnerable to broken links—if any pointer in the chain is lost, the entity becomes unreachable.

To be effective, forwarding-pointer chains must be kept short and the pointers must be made robust.

When an object moves from one address space (A) to another (B), it leaves a **client stub** behind in A and installs a **server stub** in B. This makes migration **transparent to clients**, because clients only interact with the client stub, which forwards requests to the object's current location.

![[Pasted image 20251201011838.png|500]]

![[Pasted image 20251201011852.png]]

Forwarding pointers form a chain of client/server stub pairs. To shorten this chain, each request carries the **ID of the initiating client stub**. When the request reaches the object at its current location, the response is sent back directly to the initiating client stub (bypassing the chain). The response also includes the object’s current location, allowing the client stub to update its server stub and create a **shortcut**.

There is a trade-off:

- Sending responses directly is faster but updates only the initiating stub.
- Sending them back through the chain is slower but lets all intermediate stubs update their pointers.

Unused server stubs can be removed, which relates to distributed garbage collection.

If a process in the forwarding chain crashes, the chain breaks. A common solution is to maintain a **home location** (the machine where the object was created). The home location keeps a fault-tolerant reference to the object's current address. If the chain breaks, clients contact the home location to rediscover the object. A naming service may track home locations when they change.

## Home Based Approaches

Broadcasting and forwarding-pointer techniques do not scale well in large networks. Broadcasting or multicasting becomes inefficient, while long chains of forwarding pointers are slow and prone to failures.

A widely used scalable solution is the **home-based approach**, where each mobile entity has a _home location_ responsible for tracking its current address. This home location is usually the place where the entity was created and is maintained fault-tolerantly. When other methods fail (e.g., forwarding pointers break), the system falls back to this home to resolve the entity’s location.

![[Pasted image 20251201012838.png|500]]
![[Pasted image 20251201012913.png]]
![[Pasted image 20251201012922.png]]

Mobile IP is a key example of this approach. A mobile device keeps a fixed IP address, and a **home agent** on its home network keeps track of its current _care-of address_ (temporary address). Incoming packets for the mobile host first arrive at the home agent, which then forwards (tunnels) them to the host’s current location and informs the sender of the new address.

However, the home-based method has drawbacks:

- It can increase communication latency because all traffic initially goes through the home.
- The home must always be available; if it fails, the entity becomes unreachable.
- If an entity permanently moves far from its home, communication becomes inefficient.

A practical improvement is to register the home location itself in a naming service and allow clients to look it up and cache it, since the home location changes rarely.

![[Pasted image 20251201013032.png]]

## Distributed Hash Tables

Distributed Hash Tables (DHTs) are used in large-scale distributed systems to efficiently locate entities. **Chord** is a representative DHT design. It uses an **m-bit identifier space**, where both nodes and entity keys are assigned identifiers using a hash function (commonly 128 or 160 bits).

- Each entity with key **k** is stored at the node whose identifier is the smallest **id ≥ k**.  
    This node is called the **successor** of **k**, written as succ(k).

- A naïve lookup method would have each node track only its immediate successor and predecessor, but this leads to linear-time lookups and does not scale.

#### **Chord’s Finger Table**

To support efficient lookups, each node **p** maintains a _finger table_ with up to **m entries**.  

![[Pasted image 20251201014207.png]]

![[Pasted image 20251201014408.png]]

In a Chord-based DHT system, **joining and leaving** the network is simple, but **maintaining correct routing information**—especially the finger tables—is the main challenge.

### **Joining the DHT**

- A new node **p** joins by contacting any existing node and requesting the lookup of  
    **succ(p − 1)**.
- Once the successor is known, node **p** inserts itself into the ring and updates local pointers.

### **Keeping the System Consistent**

Each node must keep its routing information up to date:
#### **1. Maintaining immediate successor (finger table entry 1)**

- Entry **FT₍q₎\[1]** should always store **succ(q + 1)**.
- Node **q** periodically asks its successor for its predecessor:
    - If: **q ≥ pred(succ(q + 1))**, then q’s data is correct.
    - Otherwise, a new node **p** has joined between q and succ(q + 1):
        - q updates **FTq\[1] = p**
        - q checks if p recorded q as its predecessor; if not, it corrects this.

#### **2. Updating the entire finger table**

For each finger table entry **i**, node **q** finds the successor of $k = q + 2^{i-1}$

It does so by issuing a lookup request for **succ(k)**.  
These requests run in the background at regular intervals.

#### **3. Maintaining predecessor pointers**

- Each node regularly checks if its predecessor is alive.
- If the predecessor fails, **pred(q)** is set to _unknown_.
- When updating **succ(q + 1)**, if the successor’s predecessor is unknown,  
    q notifies it that q is likely its predecessor.

### **System-wide effect**

These simple periodic checks keep the Chord ring **mostly consistent**, even with node joins, leaves, and failures—though small, temporary inconsistencies may occur.

### Exploiting Network Proximity

DHT systems like Chord can route requests inefficiently because the logical overlay may not match the physical Internet topology. As a result, lookups may traverse long geographical distances unnecessarily. To address this, DHT design must consider the underlying network. Castro et al. (2002b) identify **three main techniques**:

---

### **1. Topology-Based Assignment of Node Identifiers**

- Node IDs are assigned so that physically nearby nodes receive numerically close identifiers.
- Difficult to apply in simple systems like Chord because:
    - Mapping a one-dimensional ID space to the Internet is complex.
    - It may group nodes from the same physical network into a narrow identifier range, causing large gaps in the ID space during failures.

### **2. Proximity Routing**

- Nodes maintain **multiple alternatives** (e.g., _r successors_) for each finger-table entry.
- For node _p_, finger table entry _FTₚ\[i]_ normally stores the first node in the interval \[p+2i−1, p+2i−1].
    With proximity routing, _p_ keeps **r nodes** in that interval.
- When forwarding a lookup, a node chooses the physically closest option that still satisfies identifier constraints.
- Benefits:
    - Reduces lookup latency.
    - Improves robustness—failures no longer break lookups immediately.

### **3. Proximity Neighbor Selection**

- Routing tables are optimized to include neighbors that are _physically closest_.
- Works best when many candidates exist (not typical in Chord).
- Used effectively in systems like **Pastry**, where nodes receive overlay information from multiple peers when joining.
- Overlaps conceptually with proximity routing when finger entries store multiple successors.

### **Lookup Styles**

- **Iterative lookup**:  
    The querying node contacts each next hop step-by-step.
- **Recursive lookup**:  
    Each node forwards the request internally to the next hop (the style used in earlier explanations).

## Hierarchical Approaches

In a **hierarchical location service**, a large network is divided into **domains**, forming a tree structure.

- The **top-level domain** covers the entire network.
- Domains may be subdivided into **smaller subdomains**.
- The smallest units, called **leaf domains**, usually correspond to local-area networks or cellular regions.
 domain **D** has a **directory node `dir(D)`**, responsible for tracking entities within that domain. Together, these directory nodes form a **hierarchical (tree-like) structure**, with the **root directory node** knowing about all entities in the network.

![[Pasted image 20251201021618.png]]
#### **Location Records**

- A **leaf domain directory node** stores a full location record (e.g., an actual address) for any entity in that domain.
- A **higher-level domain directory node** only stores a **pointer** to the directory node in the lower-level subdomain where the entity currently resides.
- The **root node** stores a pointer leading down the tree toward the entity’s current domain.

This creates a path from the root to the leaf domain containing the entity.

#### **Entities with Multiple Addresses**

If an entity has multiple addresses (e.g., due to **replication**), the directory node of the **lowest common ancestor domain** stores multiple pointers—one for each subdomain containing a replica. This produces a tree similar to the structure shown in Fig. 5-6.

![[Pasted image 20251201021741.png|500]]

## **Lookup Operation**

1. A client first sends a lookup request to its **local leaf directory node**.
2. If the entity **is not in that domain**, the request is forwarded **upward** to parent domains.
3. This continues until reaching a directory node **M** that _has_ a location record for the entity.
4. From M, the request is forwarded **downward** through pointers toward the leaf domain where the entity is located.
5. The leaf node returns the entity’s **actual address** to the client.

![[Pasted image 20251201022237.png|500]]

### Key Idea

![[Pasted image 20251201022335.png]]

## **Insert Operation**

Used when an entity creates a new address (e.g., replica) in a leaf domain:

1. The insert starts at the **leaf directory node** `dir(D)` and is forwarded **upward** until reaching the first directory node **M** that already has a record for that entity.
2. M adds a pointer for the new address, pointing to the child domain where the request came from.
3. Then the child directory creates a record and points to its own child.
4. This continues **downward** until reaching the leaf domain, which stores the **actual address**.

![[Pasted image 20251201022353.png|500]]

Two insertion strategies:

- **Top-down (described above)**
- **Bottom-up**: create the leaf record early so lookups can succeed even if parent nodes are temporarily unreachable.

![[Pasted image 20251201022625.png]]
## **Delete Operation**

Deleting an address follows the reverse logic of insertion:

5. The leaf domain removes the entity’s address from its record.
6. If the record becomes empty, it deletes the pointer and notifies the parent.
7. The parent attempts to remove its pointer; if its record becomes empty, it deletes it too and informs its own parent.
8. This continues until reaching a node whose record stays nonempty or the root.

# Structured Naming

## Name Spaces

Names in distributed systems are organized into a **name space**, which can be modeled as a **labeled, directed graph** containing two types of nodes:

1. **Leaf nodes**
    - Represent actual named entities.
    - Have **no outgoing edges**.
    - Store information needed to access the entity (e.g., its address or full state, such as a file's data in a file system).
2. **Directory nodes**
    - Contain **outgoing edges**, each labeled with a name.
    - Store a **directory table**, where each edge is represented as a pair:  
        **(edge label, node identifier)**.
    - Directory nodes are also entities and have their own identifiers.

This structure allows hierarchical and organized resolution of names to the entities they represent.

![[Pasted image 20251202003114.png|500]]

A **global name** is a name that denotes the same entity, no matter where that name is used in a system. In other words, a global name is always interpreted with respect to the same direc tory node. In contrast, a **local name** is a name whose interpretation depends on where that name is being used. Put differently, a local name is essentially a relative name whose directory in which it is contained is (implicitly) known.

In many file systems, naming graphs are represented using **string-based path names** instead of explicit sequences of edge labels. A path is written as a single string with components separated by a special character—commonly the slash (“/”). The slash also indicates whether a path is **absolute**. For example, the path represented as `<home, steen, mbox>` is written as `/home/steen/mbox`.

Because naming graphs can have multiple paths leading to the same node, a single entity may have **multiple valid path names**. For example, a node might be accessible as both `/home/steen/keys` and `/keys`.

This string-based naming applies not only to file systems but also to other resources in distributed systems. In the **Plan 9** operating system, all resources (files, processes, devices, network interfaces, etc.) are named in this unified, file-like manner, effectively forming a **single naming graph for all system resources**.

![[Pasted image 20251202004004.png]]
![[Pasted image 20251202004013.png]]

A **boot block** is a special disk block loaded into main memory during system startup; its purpose is to load the operating system.

A **superblock** stores global information about the file system, such as its total size, free disk blocks, and unused inodes.

Files are represented by **inodes**, each identified by an index number (with inode 0 reserved for the root directory). An inode stores metadata—such as ownership, timestamps, protection—and pointers to where the file’s data is located on disk.

Directories are also stored as files. A directory maps file names to inode numbers, making each inode number equivalent to a **node identifier** in the naming graph.

## Name Resolution

Name spaces allow information about entities to be stored and retrieved using names. Resolving a name means following a path name through the naming graph: starting at a given node, each label in the path is looked up in that node’s directory table to obtain the identifier of the next node. This process continues label by label until the final node is reached, whose contents are then returned.

A name lookup always returns the identifier of the next node in the resolution process. In UNIX-style file systems, node identifiers correspond to inode numbers. To access a directory table, the system must first read the inode to find the disk location of the directory data, and then read the data blocks containing the directory table.

### Closure Mechanism

Knowing how and where to start name resolution is generally referred to as a closure mechanism.

A **closure mechanism** determines _how name resolution begins_—that is, how the system selects the **starting node** in a name space before resolving a path.

Closure mechanisms can be **implicit** and vary across systems:

- **UNIX file systems:**  
    Resolution of a path like `/home/steen/mbox` starts at the **root directory’s inode**. The operating system already knows where the root inode is located (from the superblock and built-in knowledge). This built-in starting point acts as the closure mechanism.
- **Telephone numbers:**  
    A number like `"0031204430784"` becomes meaningful only if we know it is a **telephone number**. That knowledge serves as a closure mechanism and tells us how to begin resolution—by dialing it.
- **Local vs global names:**  
    Environment variables (e.g., `HOME` in UNIX) are **local names**, stored in user-specific tables. The closure mechanism ensures that when a variable is used, the system looks it up in the correct table.

Overall, the closure mechanism provides the **initial context** needed to start name resolution, regardless of the naming system.


### Linking and Mounting

Aliases are alternative names for the same entity. They are closely related to name resolution and can be implemented in two main ways in naming graphs:

1. **Multiple Path Names to the Same Node (Hard Links)**
    - Several absolute path names directly refer to the same node.
    - Example: In UNIX, both `/keys` and `/home/steen/keys` may refer to the same underlying node.
    - These multiple references are called **hard links**.
	
2. **Nodes Storing Path Names (Symbolic Links)**
    - The entity is represented by a leaf node that **stores an absolute path name**, not the actual data.
    - When this node is resolved, the stored path name is returned and resolution continues using that new path.
    - This corresponds to **symbolic links (symlinks)** in UNIX.
    - Example: `/home/steen/keys` might be a symbolic link containing `/keys`.

Both methods allow different names to refer to the same entity but behave differently in practice.

![[Pasted image 20251202005728.png|500]]

Name resolution normally happens within a single name space, but it can also be used to **merge multiple name spaces transparently**. This is done through **mounting**.

A **mounted file system** is modeled by having a directory node in one name space store the identifier of a directory node in a _different_ (foreign) name space.

- The directory node in the local name space is the **mount point**.
- The directory node in the foreign name space is the **mounting point** (often the root of that name space).  
    During name resolution, when the mount point is encountered, resolution continues in the foreign name space by accessing the mounting point’s directory table.

This concept generalizes to **any type of name space**, not just file systems.

In distributed systems—where each name space may be implemented by a different server—mounting requires additional information because the foreign name space might live on a different machine. Mounting a foreign name space requires at least:

1. **Name of an access protocol** (to know how to communicate).
2. **Name of the server** (running the foreign name space).
3. **Name of the mounting point** inside the foreign name space.

All three names must themselves be resolved (e.g., protocol → implementation; server name → address; mounting point → node identifier).

In nondistributed systems like UNIX, mounting is simpler:

- No access protocol or server name is needed.
- The mounting point is implicitly the root of the foreign file system.

In distributed contexts, these names are often bundled into a **URL**, which provides a single reference containing all necessary information.

The example shows how remote name spaces can be mounted using URLs.  
The name **`nfs`** is globally recognized and is resolved to the NFS access protocol. The **server name** in the URL is resolved to an address using **DNS**, and the **mounting point** (e.g., `/home/steen`) is resolved by the remote file server.

NFS - Network File System

On the client machine, the local file system contains a directory such as `/remote/vu`, which stores the URL:

`nfs://flits.cs.vu.nl//home/steen`

When a user accesses a name like `/remote/vu/mbox`:

1. Name resolution proceeds locally until reaching the mount point `/remote/vu`.
2. The stored URL is returned, causing the client to contact **flits.cs.vu.nl** via NFS.
3. The client then accesses the remote directory `/home/steen` and continues resolving `mbox` on the remote server.

This allows the user to treat the remote file system as if it were part of the local name space, making operations such as:

`cd /remote/vu ls -l`

behave as though the files were stored locally, with only performance differences noticeable.

![[Pasted image 20251202010326.png|500]]

## Implementation of a Name Space

A name space forms the heart of a naming service, that is, a service that allows users and processes to add, remove, and look up names. A naming service is implemented by name servers. If a distributed system is restricted to a local area network, it is often feasible to implement a naming service by means of only a single name server. However, in large-scale distributed systems with many entities, possibly spread across a large geographical area, it is necessary to distribute the implementation of a name space over multiple name servers.

### Name Space Distribution

Large-scale distributed naming systems—especially worldwide ones—are typically organized **hierarchically** and logically divided into **three layers** to make management efficient:

#### **1. Global Layer**

- Contains the **highest-level directory nodes**, including the **root node** and its immediate children.
- These nodes are **very stable**—their directory tables change rarely.
- They often represent **large organizations or groups of organizations**.

#### **2. Administrational Layer**

- Contains directory nodes managed **within a single organization**.
- Nodes represent **organizational groupings** such as departments, sets of hosts, or all users.
- More changes occur here than in the global layer, but the structure is still relatively stable.

#### **3. Managerial Layer**

- Contains nodes that **change frequently**.
- Includes:
    - Machines on local networks
    - Shared system files (libraries, binaries)
    - User-defined directories and files
- Maintained not only by system administrators but also by **end users**.

Large-scale distributed name spaces (such as DNS) are usually organized **hierarchically** and divided into **three logical layers**:

#### **1. Global Layer**

- Contains top-level directory nodes (the root and its immediate children).
- Very **stable**: updates are rare.
- Must have **high availability**, because failure prevents access to large portions of the name space.
- Lookup results remain valid for long periods, so **client-side caching** is highly effective.
- Performance emphasis: **high throughput**, not necessarily low response time.
- Implementation often uses **replication + caching**.

#### **2. Administrational Layer**

- Managed by a single organization.
- Nodes represent groups such as departments, hosts, users, etc.
- More updates than the global layer but still relatively stable.
- Must provide **fast lookup** for users inside the organization (milliseconds).
- Use **high-performance servers**, replication, and caching for availability.

#### **3. Managerial Layer**

- Represents frequently changing entities (hosts, shared files, user directories).
- Maintained by both system administrators and end users.
- **High performance** required; updates are frequent.
- **Availability** is less strict—temporary failures are tolerable.
- Caching is less effective unless specialized techniques are used.

![[Pasted image 20251202012640.png|500]]

#### **DNS Example**

- The DNS name space is partitioned into **zones**, each operated by a separate server.
- Higher layers (e.g., top-level domains) require strong availability and benefit from caching.
- Lower layers (e.g., per-organization nodes) must handle faster updates.
- A comparison (Fig. 5-14) highlights:
    - **Global + administrational layers**: hardest to implement due to replication, caching, and consistency issues.
    - **Managerial layer**: easier to implement but requires strong responsiveness.

![[Pasted image 20251202012402.png|600]]

#### Implementation of Name Resolution

In large-scale distributed naming systems, a name space is spread across multiple name servers. A client uses a **local name resolver** to resolve full path names (e.g., `ftp://ftp.cs.vu.nl/pub/globe/index.html`). Assuming no caching and no replication, there are **two main methods** for implementing name resolution:

---

### **1. Iterative Name Resolution**

- The client sends the **full name** to the **root name server**.
- The root server resolves only the part it knows (e.g., `nl`) and returns:
    - The address of the next name server.
    - The remaining unresolved path.
- The client then contacts each subsequent name server in turn (e.g., `VU`, then `cs`, then `ftp`), receiving the next server’s address each time.
- Finally, the client contacts the FTP server directly to retrieve the file.
- The client controls the whole step-by-step process.
### **2. Recursive Name Resolution**

- The client sends the full name to the root server **once**.
- The root server forwards the remaining name to the next name server.
- That server resolves what it can, then forwards the rest to the next server, and so on.
- The final server completes resolution and sends the file (or final result) back up the chain to the root, which returns it to the client.
- Servers perform the communication among themselves rather than the client doing each step.

![[Pasted image 20251202013831.png|500]]
![[Pasted image 20251202013854.png|500]]
### **Key Difference**

- **Iterative:** the client contacts each server step-by-step.
- **Recursive:** servers contact each other, and the client waits for the final result.

Recursive name resolution places a heavier load on name servers because each server must continue resolving the path name, rather than returning intermediate results to the client. For this reason, global-layer servers typically support only _iterative_ resolution.

However, recursive name resolution has **two major advantages**:
### **1. More Effective Caching**

- Each name server involved in recursive resolution learns about lower-level servers during the process.
- Returned results—such as the address of a lower-level name server—can be cached at multiple points:
    - the root server
    - intermediate servers (e.g., _nl_ node server)
    - the server returning the final result
- Because upper-layer nodes rarely change, cached entries remain valid for a long time.
- Eventually, lookups become very fast because servers already know where to forward requests.

In contrast, with iterative resolution, **only the client’s resolver** can cache results. Other clients will need to repeat the full lookup unless an organization uses a shared local caching name server.

### **2. Lower Communication Costs**

- With recursive resolution, the client communicates only with the first server (e.g., the _nl_ server).  
    All further communication happens between name servers themselves.
- With iterative resolution, the client must contact multiple servers individually (e.g., nl → vu → cs), leading to more long-distance messages.
- Therefore, recursive resolution can significantly reduce network traffic—especially across wide-area networks.

![[Pasted image 20251202014745.png]]

# Domain Name System (DNS)

The **Domain Name System (DNS)** is a large, hierarchical distributed naming service used mainly to map hostnames to IP addresses and mail servers. The name space is organized as a **rooted tree**:

- **Labels** (node names) are case-insensitive strings (max 63 characters).
    `flits.cs.vu.nl` (trailing dot for the root is usually omitted).
- Each node has exactly one incoming edge (except the root), so the label on the incoming edge is used as the node’s name.

max length for path name is 255 characters

A _subtree_ of the DNS name space is called a **domain**, and the path to its root node is the **domain name**.

Each DNS node stores **resource records (RRs)**—structured entries describing different kinds of information. Important DNS record types include:

- **SOA (Start of Authority):** administrative information for a zone (e.g., responsible admin, primary server).
- **A record (Address):** maps a host name to an IPv4 address; multiple A records may exist for multihomed hosts.
- **MX record (Mail Exchange):** identifies the mail server for a domain, similar to a symbolic link.
- **SRV record:** stores the server providing a particular service (service name + protocol). Allows service lookup without knowing the host's name.
- **NS record (Name Server):** identifies the authoritative name servers for a zone. Only zone-root nodes need NS records.
- **CNAME record (Canonical Name):** defines an alias for a host; points to the host’s primary name.
- **PTR record:** provides reverse lookup—maps IP addresses to hostnames via the `in-addr.arpa` domain (e.g., `20.20.37.130.in-addr.arpa` → hostname).
- **HINFO record:** stores host characteristics (machine type, OS).
- **TXT record:** stores arbitrary text information associated with a node.

These records collectively define how DNS stores and resolves names across the Internet.

![[Pasted image 20251202023055.png]]

#### DNS Implementations

The DNS name space is effectively split into **two layers**:

- **Global layer** (top-level domains)
- **Administrational layer** (organizational domains)

Local file systems—which would form a managerial layer—are **not part of DNS**.

Each **zone** in DNS is managed by a **name server**, and for reliability these servers are **replicated**.

- The **primary name server** handles updates by directly modifying its local DNS database.
- **Secondary servers** maintain consistency by periodically performing a **zone transfer**, copying the primary server’s data.

DNS stores its data in a small set of files; the main file contains all **resource records** for the zone. Nodes are simply identified by their **domain names**, so the “node identifier” is essentially just an implicit index into these files.

### Decentralized DNS Implementations

Traditional DNS uses a hierarchical structure with root servers receiving huge numbers of requests. Caching prevents overload, but scalability is still limited.

A more scalable alternative is to map DNS names into a **Distributed Hash Table (DHT)** by hashing each domain name and storing its DNS records under the resulting key. This loses the hierarchical structure of DNS names but gains major scalability benefits.

#### **CoDoNS: A DHT-Based DNS**

CoDoNS is a DHT-based DNS replacement that uses prefix-based routing (similar to Pastry/Tapestry).  
Example: For a node with ID **3210**, its routing table contains nodes whose IDs share prefixes of various lengths:

- prefix `0`
- prefix `1`
- prefix `2`
- prefix `30`
- prefix `31`
- prefix `33`
- prefix `320`
- prefix `322`
- prefix `323`

Node 3210 is responsible for keys starting with prefix `321`.

When a DNS name hashes to key **k**, the node responsible for prefix(k) stores its DNS records.

#### **Replication to Reduce Routing Hops**

CoDoNS replicates DNS records to nodes that share increasingly short prefixes:

- Replicating to all nodes with prefix length **i** = replication at _level i_
- Replication reduces lookup hops:
    - Level 0 ⇒ record everywhere ⇒ 0 hops
    - Level 1 ⇒ record in all nodes sharing 1 prefix ⇒ ~1 hop
    - And so on.

There is a trade-off: more replication → faster lookup but higher resource usage.

#### **Zipf Distribution for Selecting What to Replicate**

DNS queries follow a **Zipf-like distribution** (~1/nᵃ).  
This means a small number of domain names get the majority of lookups.

Probability that the **most popular Xᵢ fraction** should be replicated at level _i_ is given by:

$X_i = \left( \frac{d^i \, log(N-C)}{1 + d + ... + d^{logN - 1}} \right)^{\tfrac{1}{1-a}} \text{with} d = b^{\frac{(1-\alpha)}{\alpha}}$

Where:

- **N** = number of nodes
- **b** = base of identifier digits
- **α** = Zipf parameter (≈ 1)
- **C** = desired average lookup cost (hops)

This formula tells the system **how many (and which) DNS records to replicate**.

#### **Example**

For:

- **b = 32**, **α = 0.9**
- **10,000 nodes**
- **1,000,000 DNS records**
- **Target: average lookup = 1 hop**

Results:

- Level 0: replicate only the **top 70** most popular records everywhere
- Level 1: replicate the **next ~3,306** records
- Level 2: replicate the next **~155,769** records

Beyond that, no replication is needed.

# Attribute Based Naming

Flat and structured naming normally give each entity a unique, location-independent name. However, when users want to _search_ for entities rather than refer to a single known entity, these naming schemes are insufficient. This leads to **attribute-based naming**, where entities are described using _(attribute, value)_ pairs. A user specifies constraints on attributes, and the naming system returns one or more matching entities.

## Directory Services

These systems are known as **directory services**, in contrast to structured naming systems. Designing a useful and consistent set of attributes is difficult—especially when different people assign values inconsistently (as commonly seen in music or video databases).

To unify resource description, the **Resource Description Framework (RDF)** is used. RDF represents data as **triples**: _(subject, predicate, object)_.  
Example: _(Person, name, Alice)_.

- Subjects, predicates, and objects can all be resources.
- RDF makes heavy use of URLs to reference these resources.
- Applications can query stored RDF data, e.g., “find the record for a person named Alice.”

While RDF descriptions may be stored centrally, the actual resources may be distributed elsewhere. The major challenge is **performance**: unlike structured naming, attribute-based lookups often require **exhaustive search** through many descriptors. When descriptors are distributed across machines, special techniques are required to make search efficient.