A key characteristic of distributed systems is **partial failure**, where one component can fail while others continue functioning normally. Unlike single-machine systems—where a failure often stops the entire system—distributed systems must handle these isolated faults gracefully. A major design goal is **fault tolerance**: ensuring the system continues operating acceptably even when some components fail, and is able to recover automatically while repairs are made.

# Introduction to Fault Tolerance

## Basic Concepts

To understand fault tolerance in distributed systems, we must first understand the broader concept of **dependability**. A dependable system maintains correct operation despite faults and is evaluated through four key properties:
1. **Availability**  
    This is the likelihood that a system is functioning correctly _at any given instant_ and ready to serve users. A highly available system tends to be operational whenever it is needed.
2. **Reliability**  
    This measures the ability of a system to run _continuously_ without failure over a period of time. Unlike availability, which concerns individual moments, reliability is about sustained, uninterrupted operation. A system can have high availability but low reliability, and vice versa.
3. **Safety**  
    Safety ensures that if the system does fail temporarily, the failure does not lead to catastrophic or dangerous consequences. Safety is critical in domains like aviation, nuclear plant control, or space missions, where even brief incorrect behavior can have disastrous results.
4. **Maintainability**  
    This property refers to how easily and quickly a system can be repaired after a failure. Systems with high maintainability can often recover faster and thus indirectly contribute to high availability. Automatic detection and repair improve maintainability but are challenging to implement correctly.

Dependability sometimes also includes **security**, particularly data integrity, which will be discussed in a later chapter.

![[Pasted image 20251203201613.png]]

A system _fails_ when it can no longer provide the services it promised to its users. The underlying cause of a failure is an _error_—a flawed part of the system’s state that could lead to incorrect behavior. For example, corrupted network packets represent errors because they may be misread or not recognized at all.

The _cause_ of an error is called a _fault_. Faults can come from many sources. Some are easy to fix—like replacing a bad cable—while others, such as weather-related interference in wireless networks, cannot be eliminated easily.

Dependable systems must address faults using several approaches: fault prevention, fault removal, and fault forecasting. However, the most crucial concept is **fault tolerance**—the ability of a system to continue providing correct service even when faults occur.

Faults generally fall into three categories:
- **Transient faults:** These occur once and then disappear. Repeating the operation usually succeeds. A brief obstruction of a transmission signal (e.g., a bird flying through a microwave link) is a transient fault.
- **Intermittent faults:** These appear, disappear, and reappear unpredictably. A loose connector that works sometimes and fails at other times is a typical example. These faults are hard to diagnose because they may vanish during troubleshooting.
- **Permanent faults:** These persist until the faulty component is repaired or replaced. Examples include burned-out hardware components, disk crashes, or software bugs.

Understanding the nature of errors and faults—and designing systems that tolerate them—is essential for building dependable distributed systems.

## Failure Models

![[Pasted image 20251203202315.png]]

Distributed systems can fail in many different ways, and not all failures are equally severe. To understand failures clearly, it helps to distinguish between different types based on how a server behaves when something goes wrong. This classification helps designers determine how serious a failure is and what kind of fault tolerance is required.

Below are the key failure types:

##### **1. Crash Failures (Fail-Stop)**

A crash failure occurs when a server **halts prematurely**, but behaves correctly until it stops. After halting, the server produces **no further output**.
- Example: An operating system freezes and the machine must be rebooted.
- This is one of the simplest failures to detect: if a server stops responding entirely, others assume it has crashed.
##### **2. Omission Failures**

The server **fails to send or receive messages**.
Types:
- **Receive omission:** The server never gets a request (maybe no thread is listening or message lost).
- **Send omission:** The server completes a request but fails to send the reply (e.g., send buffer overflow).
- Servers may also “hang” due to software issues, which effectively causes omission failures.

These failures do not necessarily halt the server, but they disrupt communication.

##### **3. Timing Failures**

The server responds **too early** or, more commonly, **too late**.
- If a response violates timing constraints, a **performance failure** occurs.
- In real-time systems (e.g., streaming or control systems), timing correctness is essential.
##### **4. Response Failures**

The server responds, but the response itself is **wrong**.
Two types:
- **Value failure:** The reply contains incorrect data (e.g., search engine returns irrelevant results).
- **State transition failure:** The server reacts incorrectly to a request, often because it reaches an unexpected internal state.

Response failures are serious because the system appears to work, but produces **incorrect results**.

##### **5. Arbitrary Failures (Byzantine Failures)**

The **most severe** type. The server may:
- Produce incorrect or random output,
- Behave inconsistently,
- Or even behave maliciously in coordination with others.

These failures are extremely difficult to handle. The name refers to complex and untrustworthy behavior historically attributed to the Byzantine Empire.

##### **Fail-Stop, Fail-Silent, and Fail-Safe Distinctions**

- **Fail-stop:** A fail-stop server will simply stop producing output in such a way that its halting can be detected by other processes. In the best case, the server may have been so friendly to announce it is about to crash; otherwise it simply stops.
- **Fail-silent:** Others must _infer_ the server has crashed, but might mistakenly assume a slow server is dead.
- **Fail-safe:** The server produces incorrect output, but the wrong output is **detectable as nonsensical** (less severe than full Byzantine behavior).

## Failure Masking by Redundancy

Fault-tolerant systems rely on **redundancy** to mask failures so that other processes do not notice them. There are **three main types of redundancy** used to tolerate faults:

#### **1. Information redundancy**
Extra bits are added to data so that corrupted information can be detected and corrected.  
Example: error-correcting codes like Hamming codes that fix bit errors caused by noise.

#### **2. Time redundancy**
An operation is performed more than once so that if it fails due to transient or intermittent faults, it can be retried.  
Example: transactions that can be safely aborted and re-executed.

#### **3. Physical redundancy**
Extra hardware components or software processes are added so the system continues working even if some fail.  
Example: replicating processes so that if a few crash, the system still operates correctly.

Together, these redundancy mechanisms help mask faults and keep the system functioning despite component failures.

![[Pasted image 20251203204843.png]]

# Process Resilience

## Design Issues

To tolerate faulty processes, distributed systems often organize multiple identical processes into a **group**. The key property of a process group is that when a message is sent to the group, **all members receive it**. This ensures that if one process fails, another can take over, improving fault tolerance.

Groups are **dynamic**—they can be created or destroyed, and processes may join or leave at any time. A single process may belong to multiple groups simultaneously, so the system needs mechanisms to manage group membership.

The idea is similar to human social groups: a person may belong to several organizations and receive messages from each one independently. Likewise, process groups let a system treat a collection of processes as a single abstraction. A process can send a message to the group without needing to know how many members it has, who they are, or where they are—details that may change over time.

This abstraction simplifies communication and provides a foundation for building fault-tolerant distributed services.

### Flat vs Hierarchical Groups

![[Pasted image 20251203214037.png|500]]

Process groups can be organized in different internal structures, which affects how they communicate and handle failures. In **flat groups**, all processes are equal and there is no central controller. This symmetry means there is **no single point of failure**—if one process crashes, the group continues functioning with fewer members. However, coordination is harder because decisions typically require **collective agreement or voting**, which adds overhead.

In contrast, **hierarchical groups** have a leader or coordinator with subordinate worker processes. Communication is centralized: requests go to the coordinator, which decides which worker should handle them. This structure simplifies decision-making and can be more efficient, but it introduces a **critical weakness**—if the coordinator fails, the entire group stops functioning. Thus, each organization offers a trade-off between efficiency and fault tolerance.

### Group Membership

Managing group membership in distributed systems requires mechanisms for creating groups, deleting groups, and allowing processes to join or leave. One approach is to use a **central group server** that tracks all group information. This is simple and efficient, but it introduces a **single point of failure**—if the group server crashes, group management collapses and groups may need to be rebuilt.

A more fault-tolerant alternative is **distributed group management**, where group membership changes are communicated directly among members, often using reliable multicast. To join, a process broadcasts a request to the existing group; to leave, it sends a goodbye message. However, this only works when departures are voluntary. If a process crashes, it sends no message, so other members must detect its absence by observing that it no longer responds. Once confirmed, the crashed process is removed.

Membership changes must also be **synchronized with message delivery**. A process that has just joined must receive all subsequent group messages, and one that leaves must stop receiving and sending messages immediately. Achieving this often means representing joins and leaves as special messages inserted into the group’s message stream.

Finally, if too many processes fail, the group may become unable to function. A recovery protocol is required to rebuild the group, including resolving conflicts when multiple processes simultaneously attempt to start the recovery.

## Failure Masking and Replication

Process groups are essential for building fault-tolerant distributed systems. By replicating a process across multiple machines and organizing those replicas into a group, the system can continue operating even if some replicas fail.

There are two main approaches to using replicated processes for fault tolerance:
1. **Primary-based replication (primary-backup)**
    - The group is organized hierarchically with one primary and several backups.
    - The primary handles all updates, and backups follow along.
    - If the primary fails, backups run an election algorithm to choose a new primary.
2. **Replicated-write protocols (active replication, quorum systems)**
    - Replicas form a flat (non-hierarchical) group.
    - All processes handle writes, often using atomic or totally-ordered multicast.
    - This avoids a single point of failure but requires distributed coordination.

A key question is how many replicas are needed.  
A system is **k-fault tolerant** if it keeps operating correctly even when k processes fail:
- If failures are **fail-silent** (crash with no incorrect output), then **k + 1 replicas** are enough.
- If failures are **Byzantine** (arbitrary or malicious behavior), then **2k + 1 replicas** are required so that the **k + 1 correct replicas outvote the k faulty ones**.

In practice, it's difficult to guarantee exactly how many failures might occur, so system designers often rely on probability and statistical analysis.

Finally, to make replicated writes work, all replicas must receive requests in the **same order**, a requirement known as **atomic multicast**. This ensures that all replicas remain consistent.

## Agreement in Faulty Systems

Replicating processes into a group improves fault tolerance, but many situations require the group to **reach agreement** (consensus), such as electing a coordinator, committing a transaction, assigning work, or synchronizing actions. Agreement is easy when everything behaves perfectly, but becomes challenging when processes or communication can fail.

Distributed agreement algorithms aim for all _nonfaulty_ processes to reach the same decision within a finite number of steps. Whether agreement is achievable depends on key assumptions about the system:

1. **Synchronous vs. asynchronous systems** — In a synchronous system, processes progress in a coordinated pace; in an asynchronous one, they do not.
	A system is synchro nous if and only if the processes are known to operate in a lock-step mode. Formally, this means that there should be some constant c, such that if any processor has taken c + 1 steps, every other process has taken at least 1 step. A system that is not synchronous is said to be asynchronous.
2. **Bounded vs. unbounded message delays** — Some systems guarantee messages arrive within a fixed maximum time; others do not.
3. **Ordered vs. unordered delivery** — Some systems guarantee messages from the same sender arrive in order; others may not.
4. **Unicast vs. multicast communication** — Agreement feasibility can depend on which communication model is used.

Agreement can be reached only in certain combinations of these assumptions. Most real distributed systems are **asynchronous**, use **unicast**, and have **unbounded delays**, meaning that special reliable, ordered communication channels (like TCP) are required. The difficulty shown in Fig. 8-4 highlights how demanding agreement becomes when failures are possible.

![[Pasted image 20251203220926.png]]

The Byzantine agreement problem, studied by Lamport, explores how distributed processes can reach a common decision even when some behave maliciously or unpredictably. Under the assumptions of synchronous processes, bounded communication delays, and ordered unicast messaging, each of the N processes begins with a value Vi. The goal is for every nonfaulty process to construct the same vector V, where V\[i] equals Vi for all correct processes.

With up to k faulty processes, the algorithm works in four steps (illustrated for N = 4, k = 1). First, each process sends its value to all others. Faulty processes may lie and send different values to different receivers. Next, each process builds a vector of the received values. Then, every process sends this vector to all others, again allowing faulty ones to lie. Finally, each process examines the received vectors; for each position i, if a majority of values match, that value is adopted, otherwise it is marked UNKNOWN. In the example, all correct processes agree on the values of the nonfaulty processes (1, 2, and 4), while the value for the faulty process (3) cannot be decided, which is acceptable. The key point is that agreement is guaranteed only for the nonfaulty processes’ values.

![[Pasted image 20251203221019.png]]

![[Pasted image 20251203221034.png]]

1. **Majority Needed to Overrule Faulty Processes**  
    To ensure correct consensus when up to **k processes may be faulty**, a system needs **at least 2k + 1 correct processes**, for a total of **3k + 1**.  
    With this number, agreement can be based on a rule that requires **more than two-thirds of all processes** to vote for the same value.  
    This guarantees that the decision reflects the majority of **nonfaulty** processes, even if faulty ones lie or mislead others.
2. **Agreement May Be Impossible in Some Systems**  
    Fischer, Lynch, and Paterson (1985) proved a fundamental limitation:  
    In fully asynchronous distributed systems—where message delivery time has no upper bound **agreement is impossible if even one process can fail**, even if that failure is just a silent crash. 
    The core issue: a very slow process cannot be distinguished from a dead one.
3. **Beyond Byzantine vs. Correct Processes**  
    Traditional models assume processes are either:
    - **Byzantine** (arbitrary, malicious) or
    - **Altruistic** (fully cooperative)
    
    However, in real systems—especially across different organizations—processes may behave **rationally**.  
    Rational processes act in their own interest, such as reporting a timeout because it is cheaper than performing the actual operation.  
    To address this, the **BAR (Byzantine, Altruistic, Rational)** fault-tolerance model was proposed, combining all three behavior types.

## Failure Detection

Failure detection is essential for fault-tolerant distributed systems. To mask failures effectively, processes must first detect when another process has failed and should no longer be considered a group member.

There are two basic ways to detect failures:
1. **Active detection** – Processes periodically send “Are you alive?” (ping) messages to others and expect replies.
2. **Passive detection** – Processes assume others are alive as long as they continue to receive messages from them. This works only when communication is frequent and reliable.

In practice, active pinging is commonly used. However, timeout-based detection has problems:
- **False positives**: A process may be falsely marked as failed due to network delays or lost messages.
- **Crudeness**: Timeouts alone are an unreliable indicator, and industrial systems often use simplistic mechanisms.

Better failure detection requires combining additional techniques:
- **Gossip-based detection**: Processes periodically gossip their status to neighbors. This information spreads, allowing nodes to infer failures when status updates become stale (as seen in Obduro).
- **Cross-checking**: When one node suspects another of failing, it can ask other neighbors to verify connectivity, helping distinguish between **node failures** and **network link failures**.
- **Forwarding alive messages**: If a node is known to be alive, that information can be propagated to others that might incorrectly suspect failures.

A final issue is **failure notification**—how to inform the rest of the system when a member is detected as failed. One aggressive approach is used in FUSE: the failure of one node triggers a cascading reaction where neighboring nodes stop responding, rapidly turning a single failure into a system-wide group failure notification. This works robustly because FUSE relies on reliable TCP connections and can distinguish link failures more easily.

Overall, designing accurate and robust failure detectors is challenging, and systems must balance network unreliability, message delays, and the risk of misidentifying healthy nodes as failed.

# Reliable Client-Server Communication

Most of the failure models discussed previously apply equally well to communication channels. In particular, a communication channel may exhibit crash, omission, timing, and arbitrary failures. In practice, when building reliable communication channels, the focus is on masking crash and omission failures. Arbitrary failures may occur in the form of duplicate messages, resulting from the fact that in a computer network messages may be buffered for a relatively long time, and are reinjected into the network after the original sender has already issued a retransmission.

## Point to Point Communication

![[Pasted image 20251203222606.png]]

## RPC Semantics in the Presence of Failures

![[Pasted image 20251203222719.png]]

#### Client cannot locate the server

A client may fail to locate a suitable server—for example, if all servers are down or if the server interface has changed while the client stub remained outdated. In such cases, the binder cannot match the client to a compatible server and reports an error. One possible response is to raise an exception or signal (e.g., a custom "SERVER_NOT_FOUND" signal), allowing the client program to handle the failure. However, this approach reduces transparency: requiring programmers to write special handlers breaks the illusion that remote procedures behave like local ones. It also depends on language features (like exceptions or signals), which may not exist in all programming languages.

#### Lost request messages

![[Pasted image 20251203223203.png]]

#### Server crashes

![[Pasted image 20251203223611.png]]

When a server crashes during an RPC (remote procedure call), it becomes impossible to know whether the requested operation was executed. Three main failure-handling philosophies exist:
1. **At-least-once semantics**  
    The client keeps retrying until it eventually gets a response.  
    _The RPC is executed at least once—but possibly multiple times._
2. **At-most-once semantics**  
    The client stops immediately when an error occurs.  
    _The RPC is executed at most once—it might not have run at all._
3. **No guarantees**  
    The system provides no help; the RPC might have been executed zero, one, or many times.  
    _This is simple to implement but unreliable._

Ideally, we want **exactly-once semantics**, where the operation is performed _once and only once_, but this is generally impossible to guarantee in real distributed systems. For example, if a client sends a request to print text and the server crashes, there is no way for the client to know whether the print job already happened.

Also assume that when a client issues a request, it receives an acknowledgment that the request has been delivered to the server. There are two strategies the server can follow. It can either send a completion message just before it actually tells the printer to do its work, or after the text has been printed.

Assume that the server crashes and subsequently recovers. It announces to all clients that it has just crashed but is now up and running again. The problem is that the client does not know whether its request to print some text will actually be carried out.

After a server crash and recovery, the client faces four possible strategies:
1. **Never retry**  
    – risk: the operation never happened.
2. **Always retry**  
    – risk: the operation happens twice.
3. **Retry only if no acknowledgment of request delivery was received**  
    – assumes the server crashed before receiving the request.
4. **Retry only if an acknowledgment _was_ received**  
    – assumes the server crashed after receiving the request, making duplication possible.

Each strategy carries a trade-off because the client cannot know the exact state of the server before the crash.

![[Pasted image 20251203225441.png]]

![[Pasted image 20251203225324.png]]

#### Lost Reply Messages

Lost replies in distributed systems create uncertainty because a client cannot know whether the server is slow, the request was lost, or the reply was lost. When the client’s timer expires, it may resend the request—but repeating a request is safe only for _idempotent operations_.
- **Idempotent requests** are operations that can be performed multiple times without changing the outcome (e.g., “send me the first 1024 bytes of a file”). These can be safely retried.
- **Non-idempotent requests**, like “transfer $1 million,” cannot be repeated without causing incorrect behavior. If a reply is lost and the client resends the request, the server may perform the transfer again, causing catastrophic errors.

To safely handle lost replies when operations are not idempotent, systems use **request sequence numbers**:
1. Each client tags every request with a unique sequence number.
2. The server keeps track of the latest sequence number seen from each client.
3. If a duplicated request arrives (same sequence number), the server **does not execute it again**, but still sends back the recorded response.
4. Servers may also use a flag bit to distinguish original requests from retransmissions.

This ensures non-idempotent operations are not performed multiple times—but it requires the server to maintain per-client state, and raises questions such as how long to store this history.

#### Client Crashes

A _client crash_ can leave behind an unwanted, still-running remote computation called an **orphan**. Orphans waste resources, may hold locks, and can cause confusion—especially if the client later restarts and repeats the RPC while the orphan eventually sends a stale reply.

Nelson (1981) proposed four approaches to dealing with orphans:
1. **Orphan Extermination**  
    Before sending an RPC, the client writes a log entry to stable storage. After rebooting, it consults the log and explicitly kills any orphans.  
    _Problems:_ Logging every RPC is expensive, orphans can create more descendants, and network partitions may prevent killing them.
	![[Pasted image 20251203232412.png]]
2. **Reincarnation**  
    Time is divided into numbered epochs. When a client reboots, it broadcasts a new epoch number. All computations from previous epochs are immediately killed. If network partitions occur, surviving orphans can still be detected because they carry old epoch numbers.
3. **Gentle Reincarnation**  
    Similar to reincarnation, but instead of immediately killing all orphaned computations, each machine first tries to find the client. Only if the client cannot be located are the computations killed.
4. **Expiration**  
    Each RPC is given a time limit T to finish. After a crash, the client waits T before rebooting, ensuring any orphans have timed out and terminated.  
    _Drawback:_ Choosing a suitable T is difficult, especially for operations with very different execution times. RPCs must also request extra time explicitly.

![[Pasted image 20251203232732.png]]

# Reliable Group Communication

## Basic Reliable-Multicasting Systems

Most transport layers provide reliable **point-to-point** communication but do **not** directly support reliable communication to a **group of processes**. When groups are small, reliability can be achieved by simply establishing multiple reliable point-to-point channels, but this becomes inefficient as the group size grows.

To scale beyond small groups, we need a clear definition of **reliable multicasting**—the guarantee that a multicast message is delivered to every member of the target group. However, complications arise when group membership changes during communication (processes joining or leaving) or when processes may **crash**.

- In failure-prone systems, reliable multicasting means: **all nonfaulty members must receive the message**, and agreement must be reached on the current group membership before delivering messages. This also interacts with ordering guarantees (e.g., atomic multicast).
    

A simpler model assumes:

- Processes do **not** fail.
    
- Group membership is **fixed** during communication.
    

Under these assumptions, reliable multicasting means each message is delivered to all current members, without necessarily enforcing identical delivery order across receivers (unless specifically required).

![[Pasted image 20251203234314.png|500]]

A basic approach for reliable multicast with a single sender and multiple nonfaulty receivers (Figure 8-9):

1. The sender labels each multicast message with a **sequence number**.  
    Receivers use this to detect missing messages.
2. The sender keeps each sent message in a **history buffer** until **every receiver acknowledges** it.
3. If a receiver detects a missing message, it sends a **negative acknowledgment (NACK)**, prompting retransmission.
4. Alternatively, the sender may automatically retransmit if acknowledgments are not received in time.
5. Design choices include:
    - Piggybacking acknowledgments on other messages.
    - Retransmitting via point-to-point unicast or a single multicast.

This simple strategy works well when the number of receivers is limited.

## Scalability in Reliable Multicasting

The main limitation of the simple reliable multicast scheme is poor scalability. With **N receivers**, the sender must handle **N acknowledgments**, which can overwhelm it—this problem is known as **feedback implosion**, and becomes worse in wide-area networks.

A common improvement is to have receivers send **only negative acknowledgments (NACKs)** when they detect a missing message. This reduces the amount of feedback and scales better, though it cannot completely eliminate the risk of implosion.

However, relying solely on NACKs introduces another issue:
- The sender can never be certain all receivers received a message.
- In theory, it must keep each message in its **history buffer indefinitely** in case a late NACK arrives.

In practice, the sender eventually discards old messages to avoid unbounded buffer growth, but doing so risks being unable to satisfy a retransmission request for a discarded message.

### Nonhierarchical Feedback Control

A major requirement for scalable reliable multicast is reducing the number of feedback messages sent to the sender. The **Scalable Reliable Multicast (SRM)** protocol addresses this using **feedback suppression**.

In SRM:
- Receivers **never send positive acknowledgments**.
- They send **negative acknowledgments (NACKs)** only when they detect a missing message (message loss detection is left to the application).
- Importantly, these NACKs are **multicast to the entire group**, not sent directly to the sender.

If several receivers miss the same message _m_, all would normally send NACKs. To prevent this, each receiver schedules its NACK with a **random delay**.
- If a receiver hears another receiver’s NACK before its own timer expires, it **suppresses** its own NACK, knowing that message _m_ will be retransmitted.
- Ideally, only **one** NACK reaches the sender, which then multicasts the retransmission.  
    This mechanism greatly reduces redundant feedback.
![[Pasted image 20251203235447.png|600]]

Although this approach scales better, it introduces several issues:
1. **Accurate timer settings are difficult** in wide-area, geographically dispersed groups. Poor timer coordination can still lead to multiple simultaneous NACKs.
2. **Multicast NACKs disturb receivers that already received the message**, forcing them to process unnecessary feedback.
    - A theoretical solution is using a **separate multicast group per missing message**, but this requires extremely efficient group management and is impractical.
    - A better approach is grouping receivers that miss the same messages together to share a feedback channel.

To further improve scalability, **local recovery** can be used:
- A receiver that has message _m_ and hears a NACK may retransmit _m_ itself before the sender does, reducing load on the sender and speeding recovery.

### Hierarchical Feedback Control

Feedback suppression is a nonhierarchical approach and does not scale well to very large receiver groups. To achieve high scalability, **hierarchical reliable multicasting** is used.

In a hierarchical scheme:
- The large group of receivers is divided into **subgroups**, arranged in a **tree structure**.
- The sender’s subgroup forms the **root** of the tree.
- Within each subgroup, any small-group reliable multicast method can be used.
- Each subgroup has a **local coordinator** responsible for:
    - Forwarding multicast messages to its subgroup members.
    - Handling **retransmission requests** locally.
    - Maintaining its own **history buffer**.
- If a coordinator misses message _m_, it requests retransmission from its **parent coordinator**.
- In acknowledgment-based schemes, each coordinator acknowledges receipt upward. After receiving acknowledgments from all its subgroup members and children, a coordinator may safely delete message _m_ from its buffer.

![[Pasted image 20251204000223.png|500]]

The major challenge of hierarchical multicast is **tree construction**, which often needs to be dynamic. Ideally, the structure could follow the network’s underlying multicast tree, but modifying routers to act as coordinators is generally impractical. As a result, **application-level multicasting** (built at the application layer rather than the network layer) has become more popular.

Overall, designing reliable multicast systems that scale across large, wide-area groups is difficult. No single approach is optimal, and each solution introduces its own complexities.

# Distributed Commit

Atomic multicasting is a specific instance of the broader **distributed commit** problem, where an operation must be performed by **all** members of a group or by **none**.
- In reliable multicast, the operation is delivering a message.
- In distributed transactions, it may be committing a transaction at multiple sites.

A common way to implement distributed commit is to use a **coordinator**, which instructs all other involved processes (participants) to perform or not perform the operation. This simple approach is known as the **one-phase commit protocol**, but it has a major limitation: participants cannot notify the coordinator if they are unable to complete the operation (e.g., due to concurrency control issues).

To address such limitations, more robust commit protocols are used:
- The **two-phase commit (2PC)** protocol is widely adopted and ensures better coordination.
- However, 2PC handles coordinator failures poorly, leading to blocking problems.
- As a result, the **three-phase commit (3PC)** protocol was developed to improve failure handling.

## Two Phase Commit

The **two-phase commit protocol (2PC)**, introduced by Gray (1978), ensures that all participants in a distributed transaction either commit or abort together. Assuming no failures, it proceeds in two phases:

#### **Phase 1: Voting**
1. The coordinator sends a **VOTE_REQUEST** to all participants.
2. Each participant replies with either:
    - **VOTE_COMMIT** (ready to commit), or
    - **VOTE_ABORT** (cannot commit).

#### **Phase 2: Decision**
1. The coordinator collects all votes:
    - If **all** voted commit → it sends **GLOBAL_COMMIT**.
    - If **any** voted abort → it sends **GLOBAL_ABORT**.
2. Participants receiving **GLOBAL_COMMIT** commit; otherwise they abort.

These steps are represented in finite state machines for both coordinator and participant.

### **Failure Issues and Blocking**

2PC is prone to blocking because several states require waiting for messages:
- Participants may block in **INIT** waiting for VOTE_REQUEST.
    - If it does not arrive after a timeout, they abort.
- The coordinator may block waiting for participant votes.
    - If some votes are missing after a timeout, it aborts and sends GLOBAL_ABORT.
- A participant may block in **READY**, waiting for the coordinator’s final decision.
    - This is the most problematic case: the participant cannot unilaterally abort and must determine what the coordinator decided.

![[Pasted image 20251204002041.png]]

To resolve the READY blocking problem:
- A participant **P** may contact another participant **Q** to infer the final decision:
    - If Q is in **COMMIT**, P can safely commit.
    - If Q is in **ABORT**, P aborts.
    - If Q is in **INIT**, the coordinator likely crashed early, so P and Q abort.
    - The worst case: if **all** participants are in **READY**, no one can deduce the decision, so they must wait for the coordinator to recover.  
        → The protocol becomes **blocked**.

![[Pasted image 20251204001922.png|500]]

For a process to recover correctly in the **two-phase commit (2PC)** protocol, it must save its state to **persistent storage**. This allows it to resume the correct behavior after a crash:
- If a participant crashed in **INIT**, it can safely abort the transaction after recovery.
- If it had already logged **COMMIT** or **ABORT**, it can immediately return to that state and resend its decision.

The problematic case is when a participant crashes in **READY**. Upon recovery, it still does not know whether the transaction should commit or abort, so it must contact other participants—just as it would after timing out while waiting for the coordinator's decision.

### **Coordinator Recovery**

The coordinator needs to persist only two important states:
1. When entering **WAIT** (after sending VOTE_REQUEST), so it can retransmit VOTE_REQUEST after recovering.
2. Its **final decision** (GLOBAL_COMMIT or GLOBAL_ABORT), so it can resend that decision when recovering.

The coordinator proceeds as follows:
- Sends **VOTE_REQUEST** to all participants.
- Logs entry into **WAIT**, then collects votes.
- If votes are missing after a timeout → assumes participant failure → logs and multicasts **GLOBAL_ABORT**.
- If all votes are received:
    - If all commit → logs and multicasts **GLOBAL_COMMIT**.
    - Otherwise → logs and multicasts **GLOBAL_ABORT**.

### **Participant Behavior**

Each participant:
1. Waits for **VOTE_REQUEST**. If it times out, it aborts (coordinator likely failed).
2. If it receives VOTE_REQUEST:
    - Decides whether to commit.
    - Logs its vote and sends **VOTE_COMMIT** or **VOTE_ABORT**.
    - If it voted commit, it waits for the global decision.
3. If it times out waiting for GLOBAL_COMMIT/ABORT:
    - It initiates a **termination protocol**:
        - Multicasts **DECISION_REQUEST** to all participants.
        - Waits for a response.
    - If it receives a decision (from any participant or the recovering coordinator), it logs and executes it.

Participants must also run a separate thread to answer DECISION_REQUESTS:
- If their log shows **GLOBAL_COMMIT** or **GLOBAL_ABORT**, they respond accordingly.
- If they are still in **INIT**, they may return GLOBAL_ABORT.
- If they are in other states (e.g., READY), they cannot help.

### **Blocking in 2PC**

2PC is a **blocking commit protocol** because certain situations cannot be resolved without the coordinator:
- If all participants are in **READY** and the coordinator crashes, none of them can deduce the final decision.
- They must block until the coordinator recovers.

### **Avoiding Blocking**

Two main approaches exist:

4. **Reliable multicast with immediate rebroadcasting**, which ensures participants can deduce the final decision without waiting for the coordinator.
5. The **three-phase commit (3PC)** protocol, designed specifically to handle coordinator failures without blocking.

## Three Phase Commit

The **two-phase commit (2PC)** protocol can block if the coordinator crashes, leaving participants unable to reach a final decision. The **three-phase commit (3PC)** protocol, proposed by Skeen (1981), is a nonblocking variant designed to avoid this problem under fail-stop failures. Although rarely used in practice, it provides important conceptual insight.

### **Key Properties of 3PC**

3PC extends 2PC by adding an extra phase and ensuring:
6. **No single state can transition directly to both COMMIT and ABORT.**
7. **No state exists from which a final decision is undecidable but a transition to COMMIT is still possible.**

These conditions guarantee **nonblocking behavior**.

### **How 3PC Works**

Like 2PC, 3PC involves a **coordinator** and multiple **participants**. The coordinator:
1. Sends **VOTE_REQUEST** to all participants.
2. If any participant votes to abort → coordinator sends **GLOBAL_ABORT**.
3. If all vote to commit → coordinator sends **PREPARE_COMMIT**.
4. After receiving acknowledgments that all participants are prepared → coordinator sends **GLOBAL_COMMIT**.

Thus, the commit decision occurs in **two steps**, giving participants more information about the transaction's progress.

![[Pasted image 20251204003037.png]]
### **Handling Failures**

There are only a few blocking states, and 3PC defines safe timeout actions:
#### **Coordinator Timeouts**
- If waiting for votes (**WAIT**) → timeout → coordinator aborts (same as 2PC).
- If waiting in **PRECOMMIT** → timeout means a participant crashed _after voting commit_.  
    → Coordinator safely broadcasts **GLOBAL_COMMIT**.

#### **Participant Timeouts**

A participant may timeout in **READY** or **PRECOMMIT**, conclude the coordinator has failed, and consult other participants.

Depending on the states of others:
1. **If a contacted participant is in COMMIT or ABORT** → participant follows that decision.
2. **If any participant is in INIT** → transaction must be aborted.
    - A participant can only be in INIT if no one reached PRECOMMIT.
3. **If all reachable participants are in PRECOMMIT (and form a majority)** → transaction can be safely committed.
4. **If all reachable participants are in READY (and form a majority)** → transaction should be aborted, since a crashed participant may recover to INIT.

### **Key Difference from 2PC**

3PC prevents the classic blocking scenario of 2PC, where:
- All participants are in READY,
- The coordinator crashes,
- and a crashed participant might later recover to COMMIT.

In 3PC, if any participant is in READY:
- No crashed participant can ever recover to COMMIT (only INIT, ABORT, or PRECOMMIT).  
    → Therefore, the operational participants can always reach a final decision without waiting for the coordinator.

### **Conclusion**

3PC ensures that **surviving processes can always resolve the transaction**, making it a nonblocking protocol. However, because the failure conditions that cause 2PC blocking rarely occur in practice, 3PC is mostly of theoretical interest.

# Recovery

gupta ki mkc bta ni rha karna h ya ni