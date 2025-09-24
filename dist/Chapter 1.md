A distributed system is a collection of autonomous computing elements
that appears to its users as a single coherent system.

This definition refers to two characteristic features of distributed systems.
The first one is that a distributed system is a collection of computing elements
each being able to behave independently of each other.

A second feature is that users (be they people or
applications) believe they are dealing with a single system.

# Characteristic 1: Collection of autonomous computing elements

![[Pasted image 20250919005536.png]]
![[Pasted image 20250919005522.png]]
![[Pasted image 20250919005501.png]]

Concerning the organization of the collection, practice shows that a dis-
tributed system is often organized as an overlay network. Types -

Structured overlay: In this case, each node has a well-defined set of neighbors
with whom it can communicate. For example, the nodes are organized
in a tree or logical ring.

Unstructured overlay: In these overlays, each node has a number of references to randomly selected other nodes.

In any case, an overlay network should, in principle, always be connected,
meaning that between any two nodes there is always a communication path
allowing those nodes to route messages from one to the other. A well-known
class of overlays is formed by peer-to-peer (P2P) networks.

# Characteristic 2: Single coherent system

a distributed system should appear as a single coherent system.

More specifically, in a single coherent system the collection of nodes
as a whole operates the same, no matter where, when, and how interaction
between a user and the system takes place.

![[Pasted image 20250919010109.png]]

in a sense, it is akin to the approach taken in many Unix-like operating systems in which
resources are accessed through a unifying file-system interface, effectively
hiding the differences between files, storage devices, and main memory, but
also networks.

## Middleware and distributed systems

To assist the development of distributed applications, distributed systems are
often organized to have a separate layer of software that is logically placed on
top of the respective operating systems of the computers that are part of the
system. This organization is shown in Figure 1.1, leading to what is known as
middleware.

![[Pasted image 20250919010219.png|500]]

Each application is offered the same interface. The distributed system provides the means for
components of a single distributed application to communicate with each
other, but also to let different applications communicate. At the same time,
it hides, as best and reasonably as possible, the differences in hardware and
operating systems from each application.

services of middlware - 
![[Pasted image 20250919010423.png]]

![[Pasted image 20250919010604.png]]

# Design Goals

A distributed system should make resources easily accessible; it should hide the
fact that resources are distributed across a network; it should be open; and it
should be scalable.

- **Supporting resource sharing**
Resources can be virtually anything, but typical examples include peripherals, storage facilities, data, files, services, and networks, to name just a few.
There are many reasons for wanting to share resources. One obvious reason is that of economics. For example, it is cheaper to have a single high-end reliable storage facility be shared than having to buy and maintain storage for each user separately.
![[Pasted image 20250919012722.png]]
![[Pasted image 20250919012752.png]]

- **Making distribution transparent**

![[Pasted image 20250919012836.png]]

![[Pasted image 20250919013001.png]]

**Access transparency** ensures that users and applications can interact with resources without being affected by differences in data representation, communication methods, or operating systems. For instance, a distributed file system should allow a Windows client and a Linux client to access the same file, even though each uses different file-naming conventions.

**Location transparency** hides the physical location of resources, so users only deal with logical names. For example, when you visit `http://www.prenhall.com/index.html`, you don’t need to know in which data center or server the page actually resides. The system resolves the name and delivers the content without revealing its physical location.

**Relocation transparency** makes it possible for resources or processes to be moved by the system without disrupting access. For example, when a cloud provider migrates a virtual machine from one physical server to another for maintenance, users continue to use their applications without noticing that anything changed.

**Migration transparency** supports mobility initiated by users or external events while keeping communication seamless. A classic example is a mobile phone call: even if both callers are moving between different cell towers, the conversation continues without interruption. Similarly, teleconferencing apps allow participants to switch networks (e.g., Wi-Fi to mobile data) mid-call without dropping the session.

**Replication transparency** hides the fact that multiple copies of a resource exist across the system. For example, Google’s search index is replicated across many servers worldwide, but when you search, you see consistent results without knowing which replica served your query. This boosts performance and availability while keeping the experience uniform.

**Concurrency transparency** ensures that multiple users can access shared resources at the same time without conflicts. For example, when two people update different rows of a database simultaneously, each operation should succeed without corrupting the database. Techniques like locking or transactions guarantee consistency despite concurrent access.

**Failure transparency** allows a system to mask failures so that users don’t experience disruptions when parts of the system stop working. For instance, if one server in a video streaming service crashes, another replica server can take over so playback continues smoothly. However, distinguishing between a slow server and a failed one remains a hard challenge in practice.

 **Degree of Distribution Transparency**

Although distribution transparency is generally considered preferable for any
distributed system, there are situations in which attempting to blindly hide
all distribution aspects from users is not a good idea

There is also a trade-off between a high degree of transparency and the
performance of a system.
For example, many Internet applications repeatedly try to contact a server before finally giving up. Consequently, attempting to mask a transient server failure before trying another one may slow down the system as a whole. In such a case, it may have been better to give up earlier,
or at least let the user cancel the attempts to make contact.

The conclusion is that aiming for distribution transparency may be a
nice goal when designing and implementing distributed systems, but that
it should be considered together with other issues such as performance
and comprehensibility. The price for achieving full transparency may be
surprisingly high.

- **Being open**
Another important goal of distributed systems is openness. An open dis-
tributed system is essentially a system that offers components that can easily
be used by, or integrated into other systems. At the same time, an open
distributed system itself will often consist of components that originate from
elsewhere.

- To make a distributed system **open**, its components must follow standard rules that define both the **syntax** and **semantics** of the services they provide.
- Interfaces are usually described using an **Interface Definition Language (IDL)**, which specifies the function names, parameters, return values, and exceptions.
- IDLs capture syntax precisely, but the actual **semantics** (what the services _do_) are often described informally in natural language.
- If an interface is specified properly—meaning **completely** and **neutrally**—then any implementation can interact with another as long as both follow the same specification.
    - **Complete**: all necessary details are included for building implementations; otherwise, developers may need to add their own system-specific details.
    - **Neutral**: specifications should not prescribe how implementations are built, leaving room for different designs while ensuring compatibility.
- These qualities are crucial for:
    - **Interoperability**: components from different vendors can work together using shared standards.
    - **Portability**: applications developed for one system can run on another system that implements the same interfaces.
- An open distributed system should also support **extensibility**, meaning new components can be added, existing ones replaced, or even entire subsystems (such as a file system) swapped out, all without disrupting the rest of the system.

- **Being Scalable**

![[Pasted image 20250919203806.png]]

centralized services - bottleneck for growing users and application
centralized data - single database would undoubtedly saturate all the communication lines into and out of it (imagine if dns was a single table)
centralized algo - algorithm that operates by collecting information from
all the sites, sends it to a single machine for processing, and then distributes the results.
can overload part of network

Decentralized algo characteristics
![[Pasted image 20250919203754.png|500]]

### Scalability dimensions

- **Size scalability:** A system can be scalable with respect to its size, meaning that we can easily add more users and resources to the system without any noticeable loss of performance.

- **Geographical scalability:** A geographically scalable system is one in which the users and resources may lie far apart, but the fact that communication delays may be significant is hardly noticed.
![[Pasted image 20250919204220.png]]

- **Administrative scalability:** An administratively scalable system is one that can still be easily managed even if it spans many independent administrative organizations. If a distributed system expands to another domain, two types of security measures need to be taken. First, the distributed system has to protect itself against malicious attacks from the new domain. Second, the new domain has to protect itself against malicious attacks from the distributed system

![[Pasted image 20250919020404.png]]

**Scaling Techniques**
![[Pasted image 20250919204605.png]]
![[Pasted image 20250919204631.png]]
![[Pasted image 20250919204516.png]]
![[Pasted image 20250919021109.png]]

![[Pasted image 20250919021154.png]]
![[Pasted image 20250919021159.png]]

![[Pasted image 20250919021310.png]]
![[Pasted image 20250919021325.png]]

![[Pasted image 20250919021415.png]]

**Replication** is often used to address scalability problems by improving availability, balancing load, and reducing latency in geographically dispersed systems. **Caching** is a special case of replication where clients, not resource owners, decide to keep local copies. Both techniques, however, introduce **consistency problems** since multiple copies of a resource may diverge.

The level of acceptable inconsistency depends on the application. For example, slightly outdated cached web pages are tolerable, but systems like stock exchanges require strict consistency, which demands global synchronization. Unfortunately, such mechanisms are hard to implement at scale due to network latency limits, making replication a trade-off between performance and consistency.

In practice, combining replication, caching, and different consistency models often yields workable solutions. Among scalability challenges, **size scalability** can often be solved by adding more capacity, while **geographical scalability** is harder due to latency constraints, and **administrative scalability** is the toughest, involving organizational and political issues. Peer-to-peer systems show that user-driven collaboration can bypass some of these problems, though they are not a universal solution.

# Types of Distributed Systems

## Distributed Computing Systems

High-performance distributed systems can be divided into two types:
- **Cluster computing**: Uses a group of similar workstations/PCs connected via a high-speed local network, all running the same operating system.
- **Grid computing**: A federation of different computer systems across various administrative domains, often with heterogeneous hardware, software, and networks.

**Cluster Computing** - homogenous

![[Pasted image 20250919205254.png]]
![[Pasted image 20250919205353.png]]

**Grid Computing Systems** - heterogeneous

![[Pasted image 20250919205605.png|500]]
![[Pasted image 20250919205842.png]]
![[Pasted image 20250919210350.png]]

## Distributed Information Systems

![[Pasted image 20250919210742.png]]
**Transaction Processing Systems**
![[Pasted image 20250919210722.png]]
![[Pasted image 20250919210759.png]]
![[Pasted image 20250919210809.png]]

A nested transaction is constructed from a number of subtransactions. The top-level transaction may fork off children that run in parallel with one another, on different machines, to gain performance or simplify programming. Each of these children may also execute one or more subtransactions, or fork off its own children
![[Pasted image 20250919210839.png|300]]
When any transaction or subtransaction starts, it is conceptually given a private copy of all data in the entire system for it to manipulate as it wishes. If it aborts, its private universe just vanishes, as if it had never existed. If it commits, its private universe replaces the parent's universe
![[Pasted image 20250919221648.png]]

**Enterprise Application Integration**

![[Pasted image 20250919222017.png]]
![[Pasted image 20250919222657.png]]

# Distributed Pervasive Systems

![[Pasted image 20250919222932.png]]

**Home Spaces**

![[Pasted image 20250919223250.png]]
