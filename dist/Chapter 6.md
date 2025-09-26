![[Pasted image 20250925204539.png]]

# Clock Synchronization

![[Pasted image 20250925205105.png]]

## Clock Synchronization Algorithms

![[Pasted image 20250925205858.png]]

Cp(t) = dc/dt
![[Pasted image 20250925205913.png]]
![[Pasted image 20250925210030.png|500]]

### Network Time protocol

![[Pasted image 20250925221202.png]]
delay
![[Pasted image 20250925221716.png]]
![[Pasted image 20250925221209.png]]
![[Pasted image 20250925221440.png]]

### The Berkeley Algorithm

Here the time server (actually, a time daemon) is active, polling every machine from time to time to ask what time it is there. Based on the answers, it computes an average time and tells all the other machines to advance their clocks to the new time or slow their clocks down until some specified reduction has been achieved. This method is suitable for a system in which no machine has a WWV receiver. The time daemon's time must be set manually by the operator periodically.

![[Pasted image 20250926115133.png]]
#### Clock Synchronization in Wireless Networks

Reference broadcast synchronization (RBS)

![[Pasted image 20250925222906.png|600]]
![[Pasted image 20250925222845.png]]
![[Pasted image 20250925222934.png]]

# Logical Clocks

![[Pasted image 20250926050830.png]]

![[Pasted image 20250926050901.png]]

## Lamport's Logical Clock

![[Pasted image 20250926051636.png]]
![[Pasted image 20250926051703.png]]
![[Pasted image 20250926051738.png]]

## Vector Clocks

![[Pasted image 20250926052251.png]]

![[Pasted image 20250926052301.png]]
![[Pasted image 20250926052330.png]]
![[Pasted image 20250926052339.png]]

![[Pasted image 20250926052733.png]]

### Enforcing casual communication

![[Pasted image 20250926053533.png]]
# Mutual Exclusion

![[Pasted image 20250926054230.png]]

## A Centralized Algorithm

![[Pasted image 20250926054357.png]]
![[Pasted image 20250926054447.png]]

## Decentralized Algorithm

![[Pasted image 20250926054704.png]]

## Distributed Algorithms

![[Pasted image 20250926055025.png]]
![[Pasted image 20250926055042.png]]

### Token Ring Algorithm

![[Pasted image 20250926055257.png]]

### Comparison of Four Algorithms

![[Pasted image 20250926055320.png]]
![[Pasted image 20250926055522.png]]

# Election Algorithms

![[Pasted image 20250926060057.png]]

## Traditional Election Algorithms

### The Bully Algorithm

![[Pasted image 20250926060212.png]]
![[Pasted image 20250926060326.png]]
![[Pasted image 20250926060417.png]]
### A Ring Algorithm

![[Pasted image 20250926060430.png]]
![[Pasted image 20250926060646.png|400]]

## Elections in Wireless Environments

![[Pasted image 20250926060800.png]]

![[Pasted image 20250926060915.png]]
## Elections in Large Scale Systems

![[Pasted image 20250926061434.png]]
![[Pasted image 20250926061914.png]]
![[Pasted image 20250926061847.png|500]]

![[Pasted image 20250926062040.png]]