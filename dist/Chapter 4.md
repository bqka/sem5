# Index

## Fundamentals
### Layered Protocols
#### Lower-Level Protocols
- Physical & data-link–level communication  
- Hardware-dependent protocols  

#### Transport Protocols
- TCP, UDP, flow control, reliability  

#### Higher-Level Protocols
- Application-level communication standards (HTTP, FTP, SMTP)

#### Middleware Protocols
- Provide higher-level abstractions  
- Enable platform-independent communication  

### Types of Communication
- Unicast  
- Multicast  
- Broadcast  
- Anycast  
- Synchronous vs asynchronous  
- Persistent vs transient communication  

---

## Remote Procedure Call (RPC)
### Basic RPC Operation
#### Conventional Procedure Calls
- Normal local procedure call semantics  
- Stack frame setup, arguments, return values  

#### Client and Server Stubs
- Marshaling/unmarshaling  
- Communication handling hidden from programmer  

### Parameter Passing
#### Passing Value Parameters
- Marshaling values into messages  

#### Passing Reference Parameters
- Must convert references into values or identifiers  
- Requires indirection  

#### Parameter Specification and Stub Generation
- IDL (Interface Definition Language)  
- Automatic stub generation tools  

### Asynchronous RPC
- One-way RPC  
- Deferred synchronous RPC  

---

## Message-Oriented Communication
### Message-Oriented Transient Communication
#### Berkeley Sockets
- Low-level send/receive primitives  
- Stream vs datagram sockets  

#### Message Passing Interface (MPI)
- High-performance cluster communication  
- Send/receive, collective operations  

### Message-Oriented Persistent Communication
#### Message Queuing Model
- Message queues decouple sender and receiver  
- Persistent storage of messages  

#### General Architecture of a Message-Queuing System
- Queue managers  
- Channels  
- Routers  
- Delivery guarantees  

#### Message Brokers
- Routing, transformation, filtering  
- Publish/subscribe systems  

---

## Stream-Oriented Communication
### Support for Continuous Media
- Time-dependent data (audio, video)  
- Continuous data flow requirements  

### Data Streams
- Stream types: continuous, discrete, periodic  
- Stream segmentation  

### Streams and Quality of Service (QoS)
- Latency, jitter, bandwidth requirements  
- QoS negotiation  

#### Enforcing QoS
- Resource reservation  
- Admission control  
- Traffic shaping  

### Stream Synchronization
#### Synchronization Mechanisms
- Intra-stream (audio sync)  
- Inter-stream (audio-video sync)  
- Event-based and timestamp-based methods  

---

## Multicast Communication
### Application-Level Multicasting
- Overlays used when network-level multicast unavailable  
- Tree-based overlays  
- Mesh-based overlays  

#### Overlay Construction
- Tree construction algorithms  
- Node maintenance  
- Dealing with churn  

### Gossip-Based Data Dissemination
#### Information Dissemination Models
- Push model  
- Pull model  
- Push–pull model  

#### Removing Data
- Anti-entropy  
- TTL / decay strategies  

#### Applications
- Replication  
- Epidemic updates  
- Failure detection  

# Fundamentals

## Layered Protocols

![[Pasted image 20250924032642.png]]
![[Pasted image 20250924032714.png|400]]
![[Pasted image 20250924032825.png]]

### Lower Level Protocols

![[Pasted image 20250924033003.png]]

### Transport Protocols

![[Pasted image 20250924033238.png]]

### Higher Level Protocols

![[Pasted image 20250924033430.png]]

### Middleware Protocols

![[Pasted image 20250924034056.png]]
![[Pasted image 20250924034131.png|500]]

## Types of Communication

![[Pasted image 20250924034822.png|500]]
![[Pasted image 20250924034810.png]]
![[Pasted image 20250924034838.png]]

# Remote Procedure Call

![[Pasted image 20250924035145.png]]
![[Pasted image 20250924035156.png]]
## Basic RPC Operation

#### Conventional Procedure Calls

![[Pasted image 20250924040110.png]]
![[Pasted image 20250924040130.png|500]]

#### Client and Server stubs

![[Pasted image 20250924040627.png|500]]
![[Pasted image 20250924040646.png]]

## Parameter Passing

### Passing Value Parameters

Packing parameters into a message is called parameter marshaling.

![[Pasted image 20250924042339.png]]
![[Pasted image 20250924042810.png]]

### Passing Reference Parameters

![[Pasted image 20250924043301.png]]
![[Pasted image 20250924043311.png]]

### Parameter Specification and Stub Generation

![[Pasted image 20250924043830.png]]
![[Pasted image 20250924044204.png|500]]

## Asynchronous RPC

![[Pasted image 20250924044245.png]]
![[Pasted image 20250924044832.png]]

# Message Oriented Communication

![[Pasted image 20250924195751.png]]

## Message Oriented Transient Communication

Many distributed systems and applications are built directly on top of the simple message-oriented model offered by the transport layer.

### Berkley Sockets

![[Pasted image 20250924200310.png]]

### Message Passing Interface

![[Pasted image 20250924201007.png]]
![[Pasted image 20250924202346.png]]

## Message Oriented Persistent Communication

![[Pasted image 20250924202810.png]]

### Message Queuing Model

![[Pasted image 20250924203253.png]]
![[Pasted image 20250924203315.png]]![[Pasted image 20250924203326.png|400]]
![[Pasted image 20250924203408.png]]

### General Architecture of a Message-Queuing System

![[Pasted image 20250924203844.png|500]]
![[Pasted image 20250924204242.png]]
![[Pasted image 20250924204324.png]]
![[Pasted image 20250924204332.png]]

### Message Brokers

![[Pasted image 20250924204710.png|500]]
![[Pasted image 20250924204754.png]]

# Stream Oriented Communication

![[Pasted image 20250924205257.png]]

time-dependent information
## Support for Continuous Media

![[Pasted image 20250924205422.png]]

### Data Stream

![[Pasted image 20250924210025.png]]

## Streams and Quality of Service

![[Pasted image 20250924210308.png]]

### Enforcing QoS

![[Pasted image 20250924211044.png]]
![[Pasted image 20250924211321.png]]
![[Pasted image 20250924211311.png|500]]

## Stream Synchronization

![[Pasted image 20250924211728.png]]

### Synchronization Mechanisms

![[Pasted image 20250924220729.png|500]]
![[Pasted image 20250924221004.png|500]]
![[Pasted image 20250924220750.png]]

![[Pasted image 20250924221402.png]]
![[Pasted image 20250924221415.png]]

# Multicast Communication

An important topic in communication in distributed systems is the support for sending data to multiple receivers, also known as multicast communication.

![[Pasted image 20250924234324.png]]

## Application Level Multicasting

![[Pasted image 20250924234639.png]]

![[Pasted image 20250924235213.png]]
### Overlay Construction

![[Pasted image 20250924235751.png]]
![[Pasted image 20250924235807.png]]
![[Pasted image 20250924235816.png]]

## Gossip Based Data Dissemination

![[Pasted image 20250925000152.png]]

#### Information Dissemination Models

![[Pasted image 20250925001145.png]]
![[Pasted image 20250925001232.png]]

#### Removing Data

![[Pasted image 20250925001607.png]]

#### Applications

![[Pasted image 20250925001710.png|720]]