# **Feistel Cipher**

### **1. Basic Idea**

A Feistel cipher splits the plaintext block into two halves called Left (L) and Right (R).  
Each round applies a round function to one half and combines it with the other half using XOR.  
After that, the two halves are swapped.

This structure guarantees that each round is _reversible_, even if the round function itself is not.

# **2. Encryption Round**

For round i:

L_(i+1) = R_i  
R_(i+1) = L_i ⊕ F(R_i, K_i)

This means:

- The right half becomes the next left half.
- The next right half is the old left half XOR the output of the round function.

# **3. Why Decryption Works Without Inverting F**

The key reason is that XOR is its own inverse.  
If you know:

C = A ⊕ B

then you can get A back by:

A = C ⊕ B

because B ⊕ B = 0.

So during decryption, we take the ciphertext halves and undo the XOR to get back the original halves.

Also, because encryption used:

L_(i+1) = R_i

we already know R_i during decryption.  
This makes it possible to recover L_i easily using the same round function F.

No matter how complicated F is, we do not need to invert it.

# **4. Decryption Round**

During decryption:

R_i = L_(i+1)  
L_i = R_(i+1) ⊕ F(L_(i+1), K_i)

This recovers the original left and right halves from the ciphertext.

The only difference from encryption is that the keys are used in reverse order.

The structure itself stays the same.

# **5. Key Points**

- Feistel networks allow encryption and decryption to use the same algorithm.
- The round function F does not need to be invertible.
- To decrypt, we simply run the rounds backwards by applying the subkeys in reverse.
- The swap of halves in each round makes the structure reversible.
- XOR is the mathematical property that makes decryption possible.

# S box -> confusion
# P box -> diffusion

![[Pasted image 20251123204821.png]]

![[Pasted image 20251123210228.png]]
![[Pasted image 20251123230559.png]]


PKI is a framework that uses certificate authorities, digital certificates, and public/private keys to provide secure communication, authentication, and trust on networks such as the Internet.
![[Pasted image 20251123232924.png]]
![[Pasted image 20251123232931.png]]
![[Pasted image 20251123233436.png]]

![[Pasted image 20251123233556.png]]

![[Pasted image 20251123233718.png]]
![[Pasted image 20251123233813.png]]

![[Pasted image 20251123235526.png]]
![[Pasted image 20251123235534.png]]
![[Pasted image 20251123235542.png]]

![[Pasted image 20251124000059.png]]

![[Pasted image 20251124001706.png]]
![[Pasted image 20251124002850.png]]
