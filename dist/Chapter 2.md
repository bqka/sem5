# Index

## Components and Connectors
### Components
- Modular units with required/provided interfaces  
- Replaceable within their environment  

### Connectors
- Mechanisms mediating communication, coordination, cooperation  
- Examples:  
  - Remote Procedure Calls (RPC)  
  - Message passing  
  - Data streaming  

## System Architectures
### Centralized Architectures
#### Client–Server Model
- Client requests; server executes  
- Idempotent vs non-idempotent operations  
- Handling lost messages  

#### Application Layering
- Presentation layer  
- Application logic layer  
- Data management layer  

#### Multitiered Architectures
- 2-tier: client ↔ server  
- 3-tier: client ↔ app server ↔ database  
- N-tier: multiple middleware/services  

### Decentralized Architectures
#### Structured P2P Architectures
##### Chord
- Ring-based lookup  
- Consistent hashing  

##### Content Addressable Network (CAN)
- d-dimensional space partitioning  
- Zone-based routing  

#### Unstructured P2P Architectures
- No rigid topology  
- Search methods:  
  - Flooding  
  - Random walks  
  - Gossip-based search  

#### Topology Management in Overlays
- Neighbor selection  
- Maintenance strategies  

#### Superpeers
- Hybrid centralized/P2P structure  
- High-capacity nodes act as indexing hubs  

### Hybrid Architectures
#### Edge Server Systems
- Push computation/storage closer to users  
- Lower latency, lower server load  

#### Collaborative Distributed Systems
##### BitTorrent
- File split into chunks  
- Download from multiple peers  
- Seeders vs leechers  

##### Globule
- Cooperative web-server replication  
- Edge server–based content distribution  

## Architectures vs Middleware
### Adaptable Middleware
- Middleware must be configurable/customizable  
- Separation of policies and mechanisms  

## Interceptors
### Purpose
- Break normal control flow to insert behavior  

### Functionality
- Applied at client and server request paths  
- Used for logging, replication, optimization  

## Approaches to Adaptive Software
### Separation of Concerns
- Isolate system functionality  

### Reflection
- System can examine/modify behavior at runtime  

### Component-Based Design
- Reusable, replaceable modules  

### Adaptive Middleware Techniques
- Dynamically adjust system behavior  

# System Architectures

Deciding on software components, their interaction, and their placement leads to an instance of a software architecture, also called a system architecture

## Centralized Architectures

Client Server Model
![[Pasted image 20250919230042.png]]
![[Pasted image 20250919230029.png|400]]
When an operation can be repeated multiple times without harm, it is said to be idempotent. Since some requests are idempotent and others are not it should be clear that there is no single solution for dealing with lost messages.

for understanding
![[Pasted image 20250919230244.png]]
![[Pasted image 20250919230334.png]]

**Application Layering**
![[Pasted image 20250919230912.png]]
![[Pasted image 20250919231530.png]]

Example 1
![[Pasted image 20250919231335.png]]
![[Pasted image 20250919231302.png|500]]

**Multitiered architecture**

![[Pasted image 20250919233720.png]]
![[Pasted image 20250919233710.png]]
![[Pasted image 20250919234011.png]]

![[Pasted image 20250919234324.png]]
![[Pasted image 20250919234216.png]]

## Decentralized Architectures

![[Pasted image 20250919234959.png]]

### Structured P2P Architecture

![[Pasted image 20250919235757.png]]

Chord system
![[Pasted image 20250920000142.png|400]]
![[Pasted image 20250920000103.png]]

Content Addressable Network (CAN)
![[Pasted image 20250920001130.png]]
![[Pasted image 20250920001113.png]]

### Unstructured P2P Architecture

![[Pasted image 20250920002340.png]]
![[Pasted image 20250920002936.png]]
![[Pasted image 20250920002942.png]]
![[Pasted image 20250920002948.png]]

### Topology Management of Overlay Networks

![[Pasted image 20250920003149.png]]
![[Pasted image 20250920003312.png]]

**Superpeers**

![[Pasted image 20250920005233.png]]
![[Pasted image 20250920004656.png]]

## Hybrid Architecture

### Edge Server  Systems
![[Pasted image 20250920010501.png]]
![[Pasted image 20250920010438.png]]

### Collaborative Distributive Systems

![[Pasted image 20250926085105.png]]

example 1 bittorrent
![[Pasted image 20250920010855.png]]
BitTorrent is a peer-to-peer file downloading system. Its principal working is shown in Fig. 2-14 The basic idea is that when an end user is looking for a file, he downloads chunks of the file from other users until the downloaded chunks can be assembled together yielding the complete file

![[Pasted image 20250920011634.png]]
![[Pasted image 20250920011656.png]]

seeders and leechers
![[Pasted image 20250920011919.png]]

example 2 Globule
![[Pasted image 20250920012251.png]]

# Architectures vs Middleware

An approach that is generally considered better is to make middleware systems such that they are easy to configure, adapt, and customize as needed by an application. As a result, systems are now being developed in which a stricter separation between policies and mechanisms is being made. This has led to several mechanisms by which the behavior of middleware can be modified
![[Pasted image 20250920012700.png]]

## Interceptors

An interceptor is a software construct that will break the usual flow of control and allow other (application specific) code to be executed.
**Interceptors make middleware smarter and more flexible** by letting extra logic (like replication or optimization) run invisibly during a request or message flow.
The basic idea is simple: an object A can call a method that belongs to an object B, while the latter resides on a different machine than A.
![[Pasted image 20250920013657.png|500]]
![[Pasted image 20250920013746.png]]

## General Approaches to Adaptive Software

What interceptors actually offer is a means to adapt the middleware.
Interceptors are part of a broader push to make middleware adaptive. While separation of concerns, reflection, and component-based design provide tools, none are yet fully effective for handling the complexity of large-scale distributed systems.

![[Pasted image 20250920014030.png]]
