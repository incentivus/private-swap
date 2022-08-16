
  
# Private Swap  

  Suppose Alice has some Bitcoin and Bob has some other coin(e.g., Litecoin). Alice wants to trade her bitcoins for Bobs litecoin.        
However, they both want to preserve their privacy, so an outside adversary or even an observer (e.g., Eve) could not        
distinguish Alice and Bobs transactions from ordinary bitcoin transactions like a simple transaction between         
Alice and Carol in which Alice sends some bitcoin to Carol. Note that having such a feature on bitcoin's blockchain        
not only improves bitcoin's fungibility but improves all network participants privacy due to the fact that         
anybody analyzing the blockchain must now deal with the possibility that Carol's transaction was actually a simple payment        
or an atomic swap or a smart contract(script) etc.         
        
        
To understand how the aforesaid swap can be done some prior knowledge is needed:      
      
 1. What is a swap? what is an atomic swap and why we need it?      
 2. How a tradition atomic is done using hash time-lock contracts?      
 3. What is taproot?      
 4. What are Schnor signatures?      
 5. What are Signature Adaptors?      
 6. What Are Scriptless Scripts?      
 7. What Are PTLCs (point timelocked contract)?      
 8. How to implement a realworld ptlc on bitcoin?      
      
      
## The Taproot Update

 The Taproot update encompasses three Bitcoin Improvement Proposals (BIPs), including [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki) (BIP – Schnorr), [BIP341](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki) (BIP – Taproot), [BIP342](https://github.com/bitcoin/bips/blob/master/bip-0342.mediawiki) (BIP – Tapscript).    
    
BIP-Schnorr introduces “Schnorr Signatures,” a faster, more secure and less data-intensive way to authorize transactions. BIP – Schnorr also enables BIP – Taproot, which uses a technique called “MAST” to commit less smart contract transaction data to the blockchain while also obscuring some private transaction information. Finally, BIP – Tapscript outfits Bitcoin with an upgraded transaction programming language which utilizes Schnorr and Taproot technology. Tapscript also allows developers to implement future Bitcoin upgrades more efficiently.    
    
Taproot got activated at block 709,632 on November 14, 2021.     
    
For the rest of this text, we focus on each component of Taproot to get a firm understanding on its different parts and how the come to gether to imrprove Bitcoins performance, scalability, privacy and fungibility.     
    
## Schnorr Signatures 

**Schnorr signature** is a [digital signature](https://en.wikipedia.org/wiki/Digital_signature "Digital signature") produced by the **Schnorr signature algorithm** that was described by [Claus Schnorr](https://en.wikipedia.org/wiki/Claus_P._Schnorr "Claus P. Schnorr"). It is a digital signature scheme known for its simplicity. Although this schema came around 1980s, it was guarded by the patent law until 2008. At that time    
there were no standardized way to implement the algorithm and not many people used in their systems, so satoshi decided to use [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) instead of Schnorr in Bitcoin. Fast forwarding a couple of years, an implementation of Schnoor Signatures was proposed in [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki).     
    
Here we try to dig a little deep into the actual workings of these algorithms. We assume the reader is already familiar with [Public-Key Cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) on an introductory level and has some understanding of [Digital Signatures](https://en.wikipedia.org/wiki/Digital_signature). We won't go into detailed proofs and won't use very formal mathematical notations, instead try to show a simplified outlook of Schnorr signatures in bitcoin.     
    
### Definitions and Notations 

We define $F_p$ as a prime filed where $p$ is prime number. We also define our $EC(elliptic \; curve)$ as cyclic group of order $p$ and choose the point $G$ on the elliptic curve and define $H:\{0,1\}^{*}\rightarrow {\mathbb  {F}}_{p}$ as cryptographic hash function. Note that all network participants should use the same $H$ and $G$.    
    
Now we take $k$ as a random number from $F_p$ and define it as the *private/secret key*. The public key is then defined as $P = kG$. We know that based on the [DL](https://www.doc.ic.ac.uk/~mrh/330tutor/ch06s02.html#:~:text=The%20discrete%20logarithm%20problem%20is,logarithms%20depends%20on%20the%20groups.) assumptions, It is computationally infeasible to calculate $k$ from $P$.  We define $m \in \{0,1\}^*$ as the message we want to sign(or encrypt) and for any point $R$ on the $EC$ curve, $R_x$ referse to the $x$ component of $R$ and $R_y$ referse to the $y$ component of $R$. We refer to $R_x$  as $r$ from now. Finally, the $||$ notation denotes binary concatenation.    
    
#### Signing a message

 To sign a message $m$ we first pick a random $z$ from $F_p$ and calculate a nonce $R = zG$. The signing algorithm is a follows:    
    
 Sig(m) = s = z + H(r || P || m ) * k (1)  We present the tuple of $(r,s)$ as the signature.    
       
Note that instead of `r || m` , `r || P || m` is used to prevent [Related-key attack](https://en.wikipedia.org/wiki/Related-key_attack).    
        
#### Verifying a signature 

To verify a signature we can simply check given $(r,s)$ and $m$ and knowing $P$  if the equality $sG = R + H(r || P || m )*P$ holds or not. The proof is straight forward, just multiply both sides of equation (1) in the last section with $G$.     
    
Note that $R$ can be driven from $r$ since $r$ represents a point on the x-axis of an elliptic curve, However since these curves are symmetric with respect to the x-axis, then there are two $y = R_y$ possible values on the curve. To break the symmetry [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#cite_ref-6-0) has suggested picking the odd value of $y$ and that is what was implemented in the taproot upgrade.    
    
    
    
---
 **References:**    
 ---