# Index

## Introduction
### Reasons for Replication
- Reliability: redundant copies protect against failures and corruption  
- Performance:  
  - Scale in number of clients  
  - Reduce latency via geographic replication  
- Introduces consistency challenges  
- Example: browser caching and staleness  

### Replication as a Scaling Technique
- Reduces read latency and server load  
- Synchronization required to maintain tight consistency → expensive  
- Relaxed consistency improves scalability  
- Trade-off: performance vs correctness  

---

## Data-Centric Consistency Models
### Continuous Consistency
#### Dimensions of inconsistency
- Numerical deviation  
- Staleness deviation  
- Ordering deviation  

#### Conits (consistency units)
- Define scope of consistency tracking  
- Fine vs coarse granularity  
- Developer APIs: AffectsConit(), DependsOnConit()  

### Consistent Ordering of Operations
- Sequential consistency  
- Causal consistency  
- Global ordering without real-time clocks  

### Sequential Consistency
- Single global interleaving  
- Per-process program order preserved  
- No real-time requirements  

### Causal Consistency
- Causally related writes must be seen in same order  
- Concurrent writes may be seen in different orders  
- Implemented using vector timestamps  

### Grouping Operations
- ENTER_CS / LEAVE_CS define atomic sections  
- Synchronization variables (exclusive and non-exclusive ownership)  
- Acquire, Exclusive, and Nonexclusive rules  

### Entry Consistency
- Associate data with specific synchronization variables  
- Explicit or implicit associations  

### Consistency vs Coherency
- Consistency → set of data items  
- Coherency → single data item  
- Coherence example: sequential coherence  

---

## Client-Centric Consistency Models
### Eventual Consistency
- If no updates occur, replicas converge  
- Works well with single-writer systems  
- Issues arise when clients move between replicas  

### Monotonic Reads
- A process never reads older data than it has seen before  
- Important for mobile users  

### Monotonic Writes
- Writes by the same process occur in program order  
- Prevents writing on outdated replicas  

### Read-Your-Writes
- A client always sees results of its own writes  
- Prevents stale reads after updates  

### Writes-Follow-Reads
- A process writes only to a version at least as recent as what it read  
- Ensures logical consistency (e.g., newsgroup replies)  

---

## Replica Management
### Replica-Server Placement
- Optimization problem: choose best K of N locations  
- Client-aware vs client-unaware placement  
- Geometric clustering (Szymaniak et al.) for fast placement  

### Content Replication and Placement
#### Permanent Replicas
- Static, authoritative copies  
- Mirror sites, distributed databases  

#### Server-Initiated Replicas
- Created dynamically based on access patterns  
- Threshold-based algorithms (replicate / migrate / delete)  

#### Client-Initiated Replicas (Caches)
- Private or shared copies  
- Useful when reads dominate  
- Layers of caches  
- Managed entirely by clients  

### Content Distribution
#### Update propagation strategies
- Invalidation (notify outdated)  
- State transfer (send new data)  
- Operation transfer (active replication)  

#### Push vs Pull Protocols
- Push → strong consistency, server tracks clients  
- Pull → client checks when needed, higher latency  
- Hybrid leases:  
  - Age-based  
  - Renewal-frequency-based  
  - Server-load-based  

#### Unicasting vs Multicasting
- Unicast → client requests  
- Multicast → efficient push updates to many replicas  

---

## Consistency Protocols
### Continuous Consistency Protocols
#### Bounding Numerical Deviation
- Writes have weights  
- Servers maintain logs TW[i,j]  
- Each server tracks others’ progress  
- Forward updates when deviation exceeds per-server bound  

#### Bounding Ordering Deviations
- Limit length of tentative-write queue  
- Switch to global ordering when limit exceeded  

### Primary-Based Protocols
#### Remote Write Protocols (Primary-Backup)
- Primary orders all writes  
- Blocking vs nonblocking versions  

#### Local-Write (Migrating Primary) Protocols
- Primary moves to the writer  
- Useful for mobile or disconnected operations  
- Nonblocking propagation to backups  

### Replicated-Write Protocols
#### Active Replication
- Each replica executes all operations  
- Requires totally ordered multicast  
- Sequencer or hybrid ordering mechanisms  

#### Quorum-Based Protocols
- Reads: contact Nr replicas  
- Writes: contact Nw replicas  
- Must satisfy Nr + Nw > N and Nw > N/2  

---

## Cache-Coherence Protocols
### Inconsistency Detection
- Static detection (compile-time)  
- Dynamic detection (runtime validation):  
  - Before use  
  - During execution (optimistic)  
  - At commit  

### Coherence Enforcement
- Disallow shared-data caching  
- Invalidate caches upon writes  
- Update caches with new values  

### Handling Writes in Caches
- Read-only caches  
- Write-through caches (immediate update to server)  
- Write-back caches (buffered updates, commit later)  

# Introduction

## Reasons for Replication

Data are replicated for two main reasons: **reliability** and **performance**.

**1. Reliability:**  
Replication allows a system to continue operating even if one copy fails. Multiple replicas help protect against corrupted data. For example, if three copies of a file exist and each read/write is performed on all of them, the system can use majority voting to detect and correct a single faulty update.

**2. Performance:**  
Replication improves performance in two ways:

- **Scaling in numbers:** When many processes need access to the same data, replicating a server allows the workload to be split among replicas.
- **Scaling in geographical area:** Placing replicas closer to users reduces access time. However, this can increase network traffic due to updates that must be propagated among replicas.

Although replication provides benefits, it also introduces **consistency problems**. When one copy is updated, the others become outdated unless updates are applied everywhere. Deciding _when_ and _how_ updates are propagated determines the cost of maintaining consistency.

A practical example is **web browser caching**:  
Browsers store local copies of visited web pages to improve access speed. But this can result in **stale pages** if the original page changes and cached copies are not updated. Preventing caching avoids staleness but worsens performance; having servers track and update all cached copies ensures freshness but increases server load.

### Replication as a Scaling Technique

Replication and caching are common techniques for improving scalability by reducing access time. However, keeping replicas up to date can add significant network overhead. If a process accesses its local replica infrequently compared to how often the replica is updated, many updates are wasted, and maintaining a local copy may not be worthwhile.

A more serious challenge is **consistency**: ensuring all copies always return the same data. This requires that updates be propagated to all replicas before any further operations occur, a property sometimes called _tight consistency_ or _synchronous replication_. Achieving this is difficult because it requires global synchronization—replicas must agree on the timing and order of updates, which is expensive in large, distributed networks.

This creates a dilemma: replication improves scalability and performance, but maintaining strict consistency reduces performance due to synchronization costs. The typical solution is to **relax consistency requirements**. By avoiding strict atomic updates, systems can reduce synchronization overhead, though replicas may temporarily differ. The degree to which consistency can be relaxed depends on data access patterns and the purpose of the replicated data.

# Data Centric Consistency Models

![[Pasted image 20251202210703.png|500]]

Consistency is discussed in systems where processes read and write shared data, such as distributed memory, databases, or file systems. A **data store** may be distributed across multiple machines, with each process having access to a local or nearby replica. When a process performs a write operation, that update must be propagated to all other replicas.

A **consistency model** defines the agreement between processes and the data store: if processes follow certain rules, the store guarantees correct behavior. Typically, a process expects a read to return the value of the “most recent” write. However, without a global clock, determining the last write is difficult, so different consistency models define what values a read is allowed to return.

Models that strongly restrict possible read values (i.e., are strict and easy for developers) usually perform poorly. Models with fewer restrictions perform better but are harder to reason about. The trade-off between usability and performance is unavoidable.

## Continuous Consistency

There is no universally best way to replicate data because strong consistency is expensive and loosening consistency depends heavily on the needs of specific applications. Yu and Vahdat propose describing acceptable inconsistencies along **three independent dimensions**, forming _continuous consistency ranges_:

1. **Numerical deviation:**  
    Replicas may differ slightly in value. This works for data with numerical meaning—for example, stock prices where replicas can differ by a small absolute or relative amount. Numerical deviation can also represent how many updates a replica has missed (its _weight_).
2. **Staleness deviation:**  
    A replica may return older data as long as it is not too outdated. Some applications, like weather reports, tolerate this, allowing updates to be propagated periodically rather than immediately.
3. **Ordering deviation:**  
    Different replicas may temporarily apply updates in different orders, as long as these differences remain limited. Updates may be tentatively applied and later rolled back or reordered after global agreement.

These dimensions help applications specify the types and degrees of inconsistency they can tolerate in exchange for better performance.

#### Notion of a conit

- **A conit (consistency unit)** defines the specific piece of data over which consistency is measured (e.g., one stock record, one weather report, one message queue).
- Yu & Vahdat define **three kinds of allowable inconsistency** for each conit:
    1. **Numerical deviation** – how much replicas may differ in value (e.g., stock price allowed to differ by $0.02).
    2. **Staleness deviation** – how old a replica is allowed to be (e.g., data can be up to 60 seconds old).
    3. **Ordering deviation** – how many updates may be applied in different orders or be tentative.
- Replicas track these deviations (often with vector clocks) to measure how far they are from each other.
- **Choosing conit size is important**:
    - Large conits (e.g., entire database) cause **too much synchronization**, making replicas inconsistent too easily.
    - Very small conits increase **management overhead** because the system must track many conits.
    - The conit granularity must balance accuracy and performance.
- Programmers specify acceptable consistency limits using simple calls:
    - `AffectsConit()` marks which conit an update modifies.
    - `DependsOnConit()` declares the maximum numerical, ordering, and staleness deviations acceptable before an operation is allowed.
- The middleware uses these limits to automatically **synchronize data only when needed**, enabling **continuous consistency** rather than strict consistency.

![[Pasted image 20251202211938.png]]

## Consistent Ordering of Operations

A large body of work on data-centric consistency focuses on how to order operations on shared, replicated data. These models come from concurrent programming, where multiple processes access shared resources at the same time. When updates must be committed across replicas, these models specify how replicas agree on a **consistent global ordering** of those updates.

### Sequential Consistency

- A system is sequentially consistent if:
    - The combined execution of all read and write operations is equivalent to _some_ single sequential order.
    - Each process’s operations appear in that global sequence **in the exact order written in its program**.
- Different interleavings of operations are allowed, but **all processes must observe the same interleaving**.
- Sequential consistency **does not rely on real time**; it does not require reading the most recent actual write, only a value consistent with the chosen global order.
- Delays in propagating updates across replicas are allowed, as long as the final ordering seen by all processes matches one valid sequential ordering.

![[Pasted image 20251202225310.png]]
![[Pasted image 20251202225322.png]]

### Casual Consistency

**Causal consistency** is a weaker consistency model than sequential consistency. Its key idea is to distinguish between:
- **Causally related events** – where one operation may influence another  
    (e.g., P₁ writes x → P₂ reads x → P₂ writes y)
- **Concurrent events** – operations that occur independently and cannot affect each other  
    (e.g., two processes independently writing different data items at the same time)

##### **Rule of Causal Consistency**

A data store is causally consistent if:
- **All processes must see causally related writes in the same order.**
- **Concurrent writes may be observed in different orders on different machines.**

This makes causal consistency weaker (more flexible) than sequential consistency, which requires _all_ processes to see _all_ writes in the same order.

![[Pasted image 20251202225904.png|500]]

##### **Examples**
- If two writes are concurrent (no causal relationship), different replicas may see them in different orders — allowed under causal consistency.
- If one write depends on another (e.g., a read influences a later write), all processes must observe them in the same order. If not, it violates causal consistency.

![[Pasted image 20251202230009.png]]

##### **Implementation**
Implementing causal consistency requires tracking which operations depend on others. This is usually done using **vector timestamps**, which record what each process has seen.

#### Grouping Operations

Sequential and causal consistency operate at the level of individual **read and write operations**, a fine granularity that comes from early shared-memory multiprocessor designs. However, real applications typically operate at a coarser level, using **critical sections** or **transactions** to control concurrency.

In practice, programs group multiple reads and writes between **ENTER_CS** and **LEAVE_CS**, treating them as an **atomic unit**. When a process successfully enters a critical section, it must see up-to-date data; it then performs its reads/writes safely, and leaves the critical section afterward.

To support this, systems use **synchronization variables**:

- Each synchronization variable has an **owner**, the process that last acquired it.
- A process must request ownership when it needs the variable, along with the current data protected by it.
- Synchronization variables may also be held in **nonexclusive mode** by multiple processes (read-only access).

To maintain correctness, three rules must be followed:

1. **Acquire rule:**  
    Before a process can complete an acquire operation, it must receive all updates to the data guarded by that synchronization variable.
2. **Exclusive rule:**  
    To update shared data, a process must have exclusive access — no other process may hold the synchronization variable, even in non-exclusive mode.
3. **Nonexclusive rule:**  
    After a process performs an exclusive update, any other process acquiring the variable in read-only mode must first obtain the latest data from the owner.

Together, these rules ensure that critical sections behave atomically and that processes always see the correct, up-to-date data when entering them.

![[Pasted image 20251202233422.png]]

### Entry Consistency

![[Pasted image 20251202233514.png]]

A challenge in **entry consistency** is correctly associating data with the synchronization variables that protect it. This can be done explicitly—by declaring which data will be accessed (similar to specifying affected database tables in a transaction)—or implicitly, for example by giving each object its own synchronization variable, which effectively serializes all accesses to that object.

#### Consistency vs Coherency

- **Consistency models** describe the expected behavior of an entire _set of data items_ when multiple processes read and write them concurrently. A system is consistent if all data items behave according to the rules of the chosen consistency model.
- **Coherence models** apply to **a single data item** replicated in multiple places. A data item is coherent if all its replicas follow the ordering rules of the chosen coherence model.

A common coherence model is **sequential coherence**, which ensures that all replicas of a single item eventually observe the same order of updates, even under concurrent writes.

# Client Centric Consistency Models

Earlier consistency models (like sequential, causal, and entry consistency) aim to give a **systemwide consistent view** of a distributed data store, assuming that multiple processes may update data at the same time. These models guarantee that when a process accesses shared data, it sees all updates made so far and that no other process interferes during the operation (mutual exclusion).

However, guaranteeing strong consistency under concurrency is expensive. To improve performance, strong consistency is often enforced **only when applications use synchronization mechanisms** like locks or transactions.

The section now shifts to a special class of distributed data stores where:

- **Simultaneous updates rarely occur**, or are easy to resolve.
- **Most operations are reads**.

Such systems can use a much weaker—but more efficient—consistency model called **eventual consistency**. With additional **client-centric consistency models**, these stores can hide inconsistencies cheaply and still provide acceptable behavior to users.

## Eventual Consistency

The degree of concurrency in distributed systems varies, and in many real systems, **most processes only read data**, while very few perform updates. In such cases, strong global consistency is often unnecessary.

Examples include:
- **Databases** where updates are rare and mostly performed by a small number of processes.
- **DNS**, where each domain can be updated only by its authoritative owner, eliminating write-write conflicts; only read-write conflicts must be handled.
- **The Web**, where pages are typically updated by a single authority, and browsers/proxies cache pages even if they become slightly outdated. Users generally tolerate some staleness.

These systems tolerate a **high degree of temporary inconsistency** because:
- Only one (or few) writers exist → write-write conflicts are rare.
- Updates can be propagated lazily.
- Users or clients often accept slightly stale data.

All these examples fit the model of **eventual consistency**:

> If no new updates occur, all replicas will gradually converge to the same state.

Eventual consistency requires only that updates eventually reach all replicas. Since there are few writers and conflicts are rare, it is **cheap and efficient** to implement.

![[Pasted image 20251202235726.png]]

Eventual consistency works well when a client always connects to the **same replica**, because that replica will gradually reflect all updates. Problems occur when a client switches between different replicas in a short time—something common for **mobile users**. If the user’s earlier updates have not yet propagated to the new replica, it may appear as though none of their changes happened, causing inconsistency from the client’s perspective.

To address this, **client-centric consistency** is introduced. Instead of guaranteeing global consistency across all clients, it guarantees that **each individual client** sees consistent behavior when accessing a data store, even if they move between replicas. It focuses on giving clients a stable, predictable view of their own past operations.

![[Pasted image 20251203000057.png|500]]

This approach comes from the Bayou system (designed for mobile and weakly connected environments like wireless networks and the Internet), where updates can be delayed and network connectivity is unreliable.

Bayou defines four client-centric consistency models (explained later), and uses the idea that each data item has a single **owner** allowed to modify it, avoiding write-write conflicts. Reads and writes operate on the nearest local replica, and updates are eventually propagated elsewhere.

The notation used:
- $Xi[t]$: version of data item x at replica $Li$​ at time $t$
- $WS(Xi[t])$: set of writes applied to that version
- If the writes at replica $L_i$ by time $t_1$​ are also present at replica $L_j$​ by time $t_2$​, we write:  
    $WS(Xi[t1])\rightarrow[t2]$
    (meaning the updates have been seen by $L_j$.

## Monotonic Reads

**Monotonic-read consistency** is the first client-centric consistency model. A data store provides monotonic reads if:

> Once a process reads a value of a data item x, any later read by that same process will never return an older value—only the same or a more recent one.

This prevents a client from “going back in time” when accessing different replicas.

**Example:**  
In a distributed email system where updates propagate lazily, a user might read their mailbox in San Francisco, then later check it in New York. Monotonic-read consistency ensures that all messages the user saw earlier will still be present—no older mailbox version will appear.

**How it works:**  
If a client reads x1​ at replica L1​, then later reads x2​ at replica L2​, monotonic-read consistency requires that all writes included in WS(x1)  (the set of updates behind the first read) must have been applied at L2​ before the second read. That is:

WS(x1)⊆WS(x2)

If this condition is not met, the client may see an outdated value at L2​, violating monotonic-read consistency.

![[Pasted image 20251203000935.png]]
![[Pasted image 20251203001135.png]]


## Monotonic Writes

Monotonic-write consistency ensures that a single process’s write operations are always applied in the order the process issued them, even if the process writes to different replicas over time.

This means:
- If a process performs one write on a data item at one replica,
- And later performs another write on the same item at a different replica,
- The second replica must first apply the earlier write before carrying out the new one.

In other words, a process’s writes are never applied “out of order.” Each new write must see the effects of all previous writes done by the same process.

### **Key Points**

- A process should never write to an outdated replica.
- Before performing a new write, the replica must be updated with all earlier writes by that same process.
- This model is similar to FIFO consistency but applies only to a _single_ process’s writes, not the whole system.
- It is important when updates depend on previous ones (for example, when updating part of a software library).

### **When it is violated**

A violation occurs when:
- A process writes at replica 1,
- Then writes again at replica 2,
- But replica 2 has not yet received the earlier write.

The second write is then applied to an outdated version, causing inconsistency.

### **Weaker version**

Note that, by the definition of monotonic-write consistency, write operations by the same process are performed in the same order as they are initiated. A weaker form exists when write operations can be applied in any order (for example, when they are commutative). In that case, all previous writes must be applied, but the exact order may not matter.

![[Pasted image 20251203001535.png]]

## Read Your Writes

**Read-your-writes consistency** ensures that when a process writes to a data item, any later read of that same item by the same process will always return a version that includes the effects of that write. In other words, a client should never read an _older_ value after having written a newer one.

A system without read-your-writes consistency can cause confusing behavior. For example:
- When a user edits and updates a Web page, the Web browser or server may return a cached (old) version instead of the updated one. Read-your-writes consistency would ensure the updated version is always shown.
- When a user changes a password, the update may take time to propagate across servers, causing the system to temporarily reject the new password. Read-your-writes consistency would prevent this by guaranteeing that the user sees the effects of their own updates immediately.

This model is particularly important in environments with caching or delayed update propagation, ensuring users never see stale data immediately after they modify it.

![[Pasted image 20251203002432.png]]

## Writes Follow Reads

**Writes-follow-reads consistency** ensures that if a process reads a data item and later writes to it, the write is applied to a version of the data that is at least as recent as the one it read. In other words, a process will never perform a write on an older version of the data than the one it just saw.

This model guarantees that updates made after reading data are based on an up-to-date copy. It's especially useful in systems like network newsgroups. For example:
- A user reads article A.
- They then post a response B.

Writes-follow-reads consistency ensures that the response B is stored only on replicas where article A is already present. This prevents situations where replies appear before the original article.

Users who only read data do not require this model; it applies only when a read is followed by a write.

![[Pasted image 20251203002533.png]]

# Replica Management

A central challenge in distributed systems with replication is determining **where, when, and by whom** replicas should be created, and how to keep them consistent. This placement issue consists of two distinct subproblems:
1. **Replica-server placement**  
    Selecting the optimal physical locations for servers that can host parts of the data store.
2. **Content placement**  
    Deciding which replica servers should store specific pieces of content (often one data item at a time).  
    Content placement cannot occur until replica servers have already been positioned.

After addressing these placement problems, the system must choose appropriate mechanisms to manage and maintain consistency among the replicas.

## Replica Server Placement

Replica-server placement is not heavily researched because it is often driven by management and business decisions rather than pure optimization. Still, analyzing client behavior and network structure can help guide placement choices.

The technical problem is an optimization task: selecting the best **K** out of **N** possible locations. This is computationally difficult, so heuristic methods are typically used.

Two main approaches exist:
1. **Client-aware placement**
    - Uses client-to-server distance (latency or bandwidth).
    - Places servers one at a time, always choosing the location that minimizes average client distance.
2. **Client-unaware placement**
    - Ignores client locations and uses Internet topology through Autonomous Systems (AS).
    - Places a server in each large AS on the router with the most links.
    - Performs similarly to client-aware methods if clients are uniformly distributed (though this assumption is uncertain).

Both methods are computationally expensive (worse than O(N²)), making them slow for large systems—especially problematic during flash crowds where quick replication is needed.

To address this, **Szymaniak et al. (2006)** proposed a fast method:
- Nodes are mapped into an m-dimensional geometric space.
- The space is divided into cells (hypercubes).
- The K densest cells (clusters of nodes with low latency to each other) are chosen.
- A replica server is placed in each selected cell.

Cell size is crucial:
- Too large → merges distinct clusters → too few replicas.
- Too small → splits clusters → too many replicas.

Their approach computes an appropriate cell size automatically and achieves near-optimal results while being dramatically faster—about **50,000× faster** than earlier methods for large inputs (e.g., 64,000 nodes). This enables **real-time replica-server placement**.

![[Pasted image 20251203010222.png]]

![[Pasted image 20251203010152.png|500]]

## Content Replication and Placement

![[Pasted image 20251203013507.png]]

### Permanent Replicas

Permanent replicas are the original, statically configured copies of data in a distributed system. In web systems, these replicas appear either as clusters of servers at one location that share the same site content, or as geographically distributed _mirror sites_ that each host a full copy of the website for users to choose from. Both approaches involve only a small, fixed number of replicas. Similarly, distributed and federated databases use static replication across several servers—either within a shared-nothing cluster or across multiple distant sites—to provide distributed storage and access.

### Server-Initiated Replicas

Server-initiated replicas are temporary copies created by the data-store owner to improve performance, unlike permanent replicas, which are fixed. These dynamic replicas are useful when demand unexpectedly increases from certain regions. For example, if a Web server in New York suddenly receives many requests from distant users, new replicas can be placed closer to those clients to reduce latency and server load.

Web hosting services make extensive use of server-initiated replication. They maintain a distributed set of servers and can automatically replicate files to the locations where they are most frequently accessed. This improves performance for clients and balances server load. Because the physical servers already exist, the main challenge is deciding where specific files should be stored.

Rabinovich et al. (1999) describe a dynamic replication algorithm that treats Web pages as mostly read-only data. Each server monitors how often a file is accessed and groups access counts based on the client’s nearest server. For each file **F** stored at server **Q**, the value **cntQ(P, F)** tracks the number of requests coming from clients whose closest server is **P**.

![[Pasted image 20251203013327.png|500]]

The algorithm uses two thresholds:
- **Deletion threshold (del(S, F))** – If requests for a file fall below this level, a server may delete the file, as long as it is not the last remaining copy.
- **Replication threshold (rep(S, F))** – If requests exceed this higher threshold, the server may replicate the file to another location.  
    Requests that fall between the two thresholds trigger _migration_ rather than replication—meaning the file is moved, not copied.

When reevaluating placement, server **Q** checks each file:
- If total access falls below the deletion threshold, the file is deleted (unless it is the only remaining copy).
- If more than half of the requests come from clients closest to another server **P**, Q tries to migrate the file to P.
- If migration fails because P is overloaded or out of storage, and the overall requests exceed the replication threshold, Q attempts to replicate the file on another server. It checks servers starting from the farthest away, and if a server **R** accounts for a significant fraction of requests, replication is attempted there.

Server-initiated replication is increasingly popular in Web hosting environments because it improves performance and load distribution. Although such replication can technically operate without permanent replicas, permanent copies are still valuable for backup and for maintaining consistency, as they often serve as the authoritative versions. Server-initiated replicas are then used mainly as read-only copies placed close to users.

### Client-Initiated Replicas

Client-initiated replicas, commonly known as _caches_, are temporary local copies of data stored by clients to improve access speed. The cache is managed entirely by the client, and the main data store is not responsible for keeping cached copies consistent—although in some cases the data store may help notify clients when cached data becomes outdated.

Caches improve performance when clients mostly read data. After a client fetches data from the nearest replica, it stores a copy in its local cache (either on its own machine or another nearby machine). Subsequent reads can then be served quickly from this cache as long as the data has not changed. Cached data is usually kept only for a limited time to avoid using stale information and to free space.

To increase the number of cache hits, caches may be shared among multiple clients. However, shared caching is only useful when clients tend to access the same data. In many file systems, files are rarely shared, reducing the benefit of shared caches. Similarly, Web caching has become less effective over time due to improved network and server performance, making server-initiated replication more impactful.

Placing client caches is straightforward—usually on the client’s device or on a local shared machine. Additional caching layers may be created by administrators, such as shared caches for departments, organizations, or entire regions. Another strategy is to place special cache servers at strategic network locations, allowing clients to locate the nearest cache and request it to store copies of data previously accessed elsewhere. Caching mechanisms and their consistency challenges are discussed further later in the chapter.

## Content Distribution

Replica management must decide how updates are propagated to other replicas. There are three main strategies:

1. **Propagate only a notification (invalidation).**  
    This method informs replicas that their data is outdated without sending the updated content. It uses very little network bandwidth and is best when there are many writes compared to reads. If updates occur frequently, sending full data each time may be wasteful because earlier updates might never be read; invalidation avoids this overhead.
2. **Transfer the updated data (state transfer).**  
    Updated content or logs of changes are sent to replicas. This approach is effective when read operations are far more common than writes, because updates are likely to be used before being overwritten. Techniques such as batching multiple updates help reduce communication overhead.
3. **Send the update operation itself (active replication).**  
    Instead of transferring data, replicas are told what operation to perform, along with its parameters. Each replica executes the operation to update itself. This uses minimal bandwidth when operation parameters are small and allows complex operations to be applied consistently. However, it requires more processing power at each replica.

Each method has different trade-offs in bandwidth usage, processing requirements, and suitability depending on the read-to-write ratio.

#### Pull vs Push Protocols

A key design decision in replica management is whether updates should be **pushed** to replicas or **pulled** by them.

In a **push-based (server-based)** approach, updates are automatically sent to replicas without them requesting the changes. This method is suitable when replicas must remain highly consistent—such as permanent replicas, server-initiated replicas, and large shared caches. These replicas typically serve many clients and handle many reads, so pushed updates are likely to be useful and ensure data is immediately consistent.

In a **pull-based (client-based)** approach, a replica or client requests updates only when needed. This is common for client caches, such as Web caches, which check with the origin server whether their cached data is still valid before serving it. Pulling updates is efficient when the read-to-update ratio is low, such as in personal caches or when cached data is seldom shared. The main disadvantage is increased response time during a cache miss, since the client must contact the server before receiving updated data.

![[Pasted image 20251203023037.png]]

Push-based update protocols require servers to track all client caches, which creates overhead and reduces fault tolerance. A server may need to remember thousands of clients, update them whenever content changes, and receive notifications when clients discard cached data. Communication patterns differ between approaches: push-based systems mainly send updates from server to clients, while pull-based systems require clients to poll the server and retrieve modified data when needed. Response times also vary—push-based delivery gives zero delay when updated data is already cached, whereas invalidation-based or pull-based schemes require fetching data, increasing delay.

To balance the strengths and weaknesses of both approaches, systems use **leases**, a hybrid mechanism where a server promises to push updates to a client for a limited period. After the lease expires, the client must begin pulling updates or request a new lease.

Leases can be dynamically adjusted based on different criteria:
1. **Age-based leases:**  
    Items that have not been modified for a long time receive long leases, reducing unnecessary update messages.
2. **Renewal-frequency-based leases:**  
    Clients that frequently request updates receive longer leases, providing them with stronger consistency and reducing server tracking of infrequent users.
3. **State-space-based leases:**  
    When the server becomes overloaded, it shortens lease durations so it has fewer clients to track, shifting toward a more stateless, efficient mode.

These leasing strategies help systems adapt between push and pull methods to optimize performance and scalability.

#### Unicasting vs Multicasting

Another design choice in update propagation is whether to use **unicasting** or **multicasting**. Unicasting requires a server to send separate update messages to each replica, while multicasting allows the network to efficiently deliver a single message to multiple receivers. Multicasting is especially advantageous when replicas are located within the same local-area network, where broadcast or multicast delivery is as cheap as sending a single unicast message.

Multicasting works well with **push-based** update propagation: a server can push updates to many replicas using one multicast group message. In contrast, **pull-based** approaches typically involve only one server or client requesting an update at a time, making unicasting more efficient in those scenarios.

# Consistency Protocols

A consistency protocol describes an implementation of a specific consistency model.

## Continuous Consistency

### Bounding Numeral Deviation

We are trying to keep multiple replicas of a _single numeric data item x_ close to the true value, even though updates may not reach every server at the same time.

##### **1. Writes and Their Weights**

Each write operation W(x) changes x by some numeric amount.  
This amount is called the write's **weight**, written as:
- weight(W) > 0
A write is first submitted to one server, called its **origin**, written as:
- origin(W) = Si (meaning server Si first received the write)

##### **2. Logs of Writes at Each Server**

Each server Si keeps a log Li containing all writes that _Si itself_ has applied to its local copy of x.

However, many writes in the system may not yet be visible everywhere, so values may differ.

##### **3. Representing the Writes Known by Each Server**

Let TW\[i,j] represent all the writes **executed by server Si** which **originated from server Sj**.

So:
- TW\[i,i] = all writes that were originally submitted to Si
- TW\[i,j] (for j ≠ i) = writes originally submitted to Sj that Si has already learned

The total value at server Si is:
- Vi = v(0) + sum of TW\[i,k] for all k from 1 to N

In other words, Vi equals the initial value plus all the updates that Si has seen so far, grouped by their origin.

##### **4. True Value vs. Local Value**

The true value of x at time t (denoted v(t)) is:
- v(t) = v(0) + sum over all servers k of TW\[k,k]

This means the true value is based on _all writes that have ever been submitted_, regardless of which servers have received them.

Because updates propagate slowly, each server’s value Vi may differ from v(t).

##### **5. Bounding the Deviation**

Each server Si is assigned a maximum allowed deviation bi:
- v(t) – Vi ≤ bi

This means server Si must never fall more than bi behind the true value.

##### **6. How Servers Track Each Other**

When a server Sk receives a write originating from Sj, it learns Sj’s value of TW\[i,j] at that moment.

Server Sk maintains TWk\[i,j], which is Sk’s _view_ of what Si currently knows about writes from Sj.

This view always satisfies:
- 0 ≤ TWk\[i,j] ≤ TW\[i,j]

In other words, Sk’s belief about what Si knows can never exceed what Si actually knows.

![[Pasted image 20251203031012.png|300]]
##### **7. Detecting If Another Server Is Falling Behind**

Server Sk checks whether server Si is too far behind.  
If Sk sees that Si has not been receiving updates fast enough, Sk will forward the missing writes from its own log to Si.

Sk decides to forward when:
- TW\[k,k] – TWk\[i,k] > bi / (N – 1)
This condition means:
- “Si is falling too far behind on the writes that originated at Sk.”

Forwarding the writes helps Si catch up.

##### **8. Why This Works**

Forwarding updates reduces the gap between:
- TW\[i,k] (what Si actually has)
- TWk\[i,k] (Sk’s view of what Si has)

When all servers follow this rule, the deviation between Vi and the true value v(t) always stays within the required bound bi.

### Bounding Ordering Deviations

Ordering deviations occur because each replica tentatively applies updates before the final global order is known, causing each server to maintain a local queue of tentative writes. To keep ordering deviations within limits, the system sets a maximum allowed queue length. When a server’s tentative-write queue grows beyond this limit, it stops accepting new writes and instead coordinates with other servers to agree on a global execution order for the pending writes. This ensures consistent ordering across replicas. In practice, systems enforce this ordering using primary-based or quorum-based protocols.

## Primary-Based Protocols

Distributed applications tend to adopt simple and intuitive consistency models—especially those that limit staleness or numerical deviation. More complex models, especially for ordering operations, are often ignored by developers because they are harder to understand, even if they offer better performance.

For ordering consistency, **sequential consistency** remains widely used, especially when supported by mechanisms like locking or transactions. In practice, **primary-based protocols** dominate: each data item has a primary server that coordinates write operations. The primary may remain fixed at one server or may move to the location where the write is initiated.

### Remote Write Protocols

![[Pasted image 20251203032727.png]]

In a primary-backup protocol, all write operations for a data item must be sent to a single designated primary server. The primary performs the update on its local copy and then forwards it to all backup servers. Each backup applies the update and acknowledges the primary. Once all backups confirm, the primary notifies the client that the write is complete. This blocking version ensures strong consistency but may be slow.

A nonblocking variant allows the primary to acknowledge the client immediately after updating its own copy, and only afterward send updates to backups. This speeds up writes but weakens fault tolerance because the update may not yet be replicated when the client is informed.

Primary-backup protocols naturally provide sequential consistency because the primary determines a single global order of all writes. In the blocking version, clients always see the effects of their most recent writes, while the nonblocking version cannot guarantee this without extra mechanisms.

### Local-Write Protocols

![[Pasted image 20251203032902.png]]

A variation of primary-backup protocols allows the **primary copy to migrate** to the process that wants to perform a write. When a process needs to update data item x, it locates the primary and moves it to itself. This allows multiple consecutive writes to be done locally, improving performance. Read operations can still be done on local replicas, but to support this efficiency, a **nonblocking protocol** is needed, where updates are propagated to other replicas _after_ the primary finishes its local updates.

This migrating-primary approach is useful for **mobile computers** that may operate while disconnected. Before going offline, the mobile device becomes the primary for the data it expects to update. While disconnected, it performs updates locally; other processes can still read but cannot write. When the device reconnects, it pushes its updates to the backup servers to restore consistency.

As a last variant of this scheme, nonblocking local-write primary-based proto cols are also used for distributed file systems in general. A central server normally handles writes, but it can temporarily allow one replica to act as a local primary and perform several local updates to improve performance. Once finished, the replica sends its updates back to the central server, which then distributes them to all other replicas.

## Replicated Write Protocols

In replicated-write protocols, write operations can be carried out at multiple replicas instead of only one, as in the case of primary-based replicas. A distinction can be made between active replication, in which an operation is forwarded to all replicas, and consistency protocols based on majority voting.
### Active Replication

Active replication assigns each replica its own process that performs update operations. Instead of sending updated data, the system typically sends the operation itself to all replicas. To ensure correctness, all replicas must execute operations in exactly the same order. This requires a **totally ordered multicast** mechanism.

Total ordering can be implemented using Lamport logical clocks, but that approach does not scale well in large systems. An alternative is to use a **central sequencer**, which assigns a unique sequence number to each incoming operation and then multicasts it to all replicas. Replicas then perform operations in sequence-number order. This approach is similar to primary-based protocols.

However, a sequencer alone also has scalability limitations. In large systems, a hybrid approach combining symmetric multicast (Lamport timestamps) with sequencer-based methods may be needed.

### Quorum-Based Protocols

Voting-based replication requires clients to obtain permission from multiple servers before reading or writing a replicated data item. In the basic example, if a file is stored on N servers, a client must contact a majority (over half) to perform a write. These servers agree to the update, after which the file is changed and given a new, identical version number on all updated replicas.

To read the file, the client again contacts a majority and checks their version numbers. If the majority all report the same version, it must be the most recent one, because an update to a higher version would require a majority to approve it, making it impossible for only the minority to have a newer version.

More generally, Gifford extends this into _quorum-based replication_, where a read operation requires contacting any Nr servers, and a write requires contacting any Nw servers, with both numbers chosen to ensure correctness.
![[Pasted image 20251203034854.png]]

The first constraint is used to prevent read-write conflicts, whereas the second prevents write-write conflicts. Only after the appropriate number of servers has agreed to participate can a file be read or written.

![[Pasted image 20251203035110.png]]
![[Pasted image 20251203035104.png]]


## Cache-Coherence Protocols

Caches are a special form of replication managed by clients, but the mechanisms used to keep caches consistent are similar to general replication protocols. In distributed systems, cache-coherence is usually implemented in software.

Caching solutions may differ in their coherence detection strategy, that is, when **inconsistencies are actually detected**.
- **Static detection:** The compiler analyzes code before execution and inserts instructions to prevent inconsistencies.
- **Dynamic detection:** The system checks at runtime whether cached data is still valid. This is common in distributed systems.

In distributed databases, dynamic detection can occur at different points during a transaction:
1. **Before using cached data:** The client must verify the data is still valid.
2. **Optimistically during execution:** The transaction proceeds while validation happens; if data is stale, the transaction aborts.
3. **At commit time:** All cached data used is validated at the end; stale data causes an abort (similar to optimistic concurrency control).

Another design choice is the **coherence enforcement strategy**, which determines **how** caches stay consistent with server copies:
- The simplest approach is to **not allow shared data to be cached**—only private data may be cached.
- If shared data is cached, servers may either:
    - **Send invalidations** when data changes, or
    - **Propagate updated values** to caches.  
        Some systems choose dynamically between the two.

Finally, systems must decide what happens when a **client modifies cached data**:
- With **read-only caches**, only servers can perform updates; caches must pull updated data when needed.
- With **write-through caches**, clients update cached data and immediately send the update to servers. This resembles a primary-based local-write protocol where the client temporarily becomes the primary and must have exclusive write permission.
- **Write-back caches** delay sending updates to the server, allowing multiple writes to accumulate first. This improves performance and is common in distributed file systems.

Overall, caching protocols vary in when they detect inconsistencies, how they enforce coherence, and how they handle write operations.