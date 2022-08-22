
      
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
        
Now we take $k$ as a random number from $F_p$ and define it as the *private/secret key*. The public key is then defined as $P = kG$. We know that based on the [DL](https://www.doc.ic.ac.uk/~mrh/330tutor/ch06s02.html#:~:text=The%20discrete%20logarithm%20problem%20is,logarithms%20depends%20on%20the%20groups.) assumptions, It is computationally infeasible to calculate $k$ from $P$.  We define $m \in \{0,1\}^*$ as the message we want to sign(or encrypt) and for any point $R$ on the $EC$ curve, $R_x$ refers to the $x$ component of $R$ and $R_y$ referse to the $y$ component of $R$. We refer to $R_x$  as $r$ from now. Finally, the $||$ notation denotes binary concatenation.        
        
#### Signing a message    
    
 To sign a message $m$ we first pick a random $z$ from $F_p$ and calculate a nonce $R = zG$. The signing algorithm is a follows:        
        
     Sig(m) = s = z + H(r || P || m ) * k 

 
We present the tuple of $(r,s)$ as the signature.        
           
Note that instead of `r || m` , `r || P || m` is used to prevent [Related-key attack](https://en.wikipedia.org/wiki/Related-key_attack).        
            
#### Verifying a signature 

To verify a signature we can simply check given $(r,s)$ and $m$ and knowing $P$  if the equality $sG = R + H(r || P || m )*P$ holds or not. The proof is straight forward, just multiply both sides of equation (1) in the last section with $G$.         
        
Note that $R$ can be driven from $r$ since $r$ represents a point on the x-axis of an elliptic curve, However since these curves are symmetric with respect to the x-axis, then there are two $y = R_y$ possible values on the curve. To break the symmetry [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#cite_ref-6-0) has suggested picking the odd value of $y$ and that is what was implemented in the taproot upgrade.        
  
#### Schnorr VS ECDSA  
  
Schnorr Signature has many advantages over ECDSA with little to no disadvantage, apart from not being standardized.  Some main pros are listed below:  
  
 - **Provable security**: Schnorr signatures are provably secure while it is still not proven that ECDSA is secure under reasonable assumptions. An explanation of the main ideas behind th security proof of Schnorr signatures can be found [here](https://www.youtube.com/watch?v=Nad2V7eUjbs)  
 - **Non-malleability**: The ECDSA signature scheme itself is vulnerable to a form of malleability, due to the fact that for every ECDSA signature $(r,s)$, the signature $(r,-s \mod N)$ is a valid signature of the same message. This problem is mitigated in bitcoin ([BIP146](https://github.com/bitcoin/bips/blob/master/bip-0146.mediawiki)) But when using a Schnorr signature, no such problem happens in the first place.  
 - **Linearity**: This arguably the most important improvement. Schnorr signatures are linear in nature, meaning we can add multiple signatures together to construct new signatures. The simple property can have many important consequences(some parts may be ambiguous for now but will be discussed and clarified later):  
    - **Better privacy**: By making different multisig spending policies indistinguishable on chain from regular *P2PK*.  Actually with Taproot, the concept of [Scriptless Scripts](https://github.com/ElementsProject/scriptless-scripts) can be implemented in bitcoin transaction which not only improves security and privacy, but also improves bitcoins fungibility as a medium of exchange.  
    - **Faster Verification Speed**: Using Schnorr signatures, network participants can do batch/bulk verification on a blocks' transaction signatures. Furthermore, since  Schnorr signatures are inherently less computationally expensive, signing and verifying any single transaction would need less time.  
    - **Less Usage of Block Space**: Because the ability of batch/bulk verification, less block space can be given to signatures. Also since in some cases instead of using *P2SH* (smart contracts), simpler forms of transactions can be used(Scriptless Scripts) which use less space and hence consume less of a blocks space.

#### MuSig  

**Linearity** of *Schnorr signatures* can be utilized to construct a new way to have **_n_-of-_n_** [Multi-Signature](https://en.bitcoin.it/wiki/Multi-signature) contracts in Bitcoin without actually using *P2SH* or writing any Bitcoin Scripts. The specific implementation of such an algorithm in Bitcoin's Taproot Upgrade is called [MuSig](https://eprint.iacr.org/2018/068). Here we only explore a general description of it.

Suppose *Alice* (with private key $k_a$ and nonce $z_a$) and *Bob* (with private key $k_b$ and nonce $z_b$) decide to create a 2-of-2 multi-signature account.Here's how they would use Schnorr to their privilege:

Alice and Bob create a new public key $P = P_a + P_b$. For any transaction $m$, each of them separately signs the message with its own private key and nonce, the resulting signatures would be $(s_a,r_a), (s_b,r_b)$ respectively, each of these signatures is called a *partial signature*. It is easy to prove that the only way two spend from account $P$ is to use the aggregated signature $(s_a+s_b, (R_a+R_b)_x)$. 

So Alice and Bob can deposit to address $P$ (which is the sum of their own public keys) and later on use the sum of their signatures to withdraw from this address. The resulting transactions are similar to *P2PKH* scripts, as a result no observer of the blockchain could distinguish this kind of multisignature transactions from ordinary ones. This improves Bitcoin's fungibility and privacy and can result more block space available for other transactions. Note that [Lightning Payment Channels](https://docs.lightning.engineering/the-lightning-network/payment-channels) are
essentially a *2-of-2* multisignature address so Schnorr can help a lot in the performance and security of the lightning network.

#### Batch Verification

Before the Taproot upgrade, to verify all the transactions of a block, a miner had to 
individually verify each transaction. With Taproot, multiple transaction can be verified at once (provided that all of them use Schnorr). For set of $m$ transactions each with signatures $S_1,...S_m$ where $S_i  = (s_i,r_i)$ and publick keys $P_1,...,P_m$, a miner only needs to check $\sum S_i = S$ against $\sum P_i = P$.

 Note that summation is a cheap operation in terms of computational resources need but verification is quite resource intensive compared to summation, hence by reducing the number of verifications drastic preformance boost can expected - especially when all of transaction inside a block use Schnorr-. 

#### Adaptor Signatures

Contracts in Bitcoin often require a locking mechanism to ensure the atomicity of a set of payments—either all the payments succeed or all of them fail. This locking has traditionally been done by having all payments in the set commit to the same hash digest preimage; when the party who knows the preimage reveals it on-chain, everyone else learns it and can unlock their own payments. This is called a *hashlock*. 

In 2019, Andrew Poelstra published a [scientific paper](https://raw.githubusercontent.com/LLFourn/one-time-VES/master/main.pdf) describing Adaptor Signatures as a means to insure the atomicity of disjoint transactions with the signatures themselves rather on relying on Bitcoin contracts. The resulting transactions will appear to verifiers to be no different from ordinary single-signer transactions(P2PKH), except perhaps for the inclusion of lock-time refund logic. It's worth noting that the concept of Adaptor Signatures can be implemented using both Schnorr signatures and ECDSA, however we focus on Schnorr based implementations for now.

Suppose Alice and Bob want to conduct a some kind of payment between themselves. 
More details on how they can do different kind of payments (e.g. atomic swap) will be provided later but regardless of the application here how they would use *Adaptor signatures* to their benefit:

At first Bob generates a random number $t$ which we call the *payment secret*, it is a secret because at first only Bob knows it and later on Alice can calculate it but even at that time no one else on the blockchain can find what $t$ was. From $t$ the value $T = tG$ is driven. $T$ is called the *payment point* is publicly broadcast on the network. 

Now Bobs creates a partial signature $s_b$ as before, he offsets the signature by $t$ resulting in $s'_b = s_b + t$ and then calculating a new nonce $R'_b = R_b + T$ hence a new partial signature $(s'_b, r'_b)$ this new signature is called an *Adaptor signature*.  Here we assume that Bob gets hold on Alice's partial signature $s_a$ via an encrypted communication protocol. Bob use Alice's signature to calculate $(s_{ab'},r_{ab'})$ where $s_{ab'} = s_a + s'_b$ and $r_{ab'} = (R_a + R'_b)_x$, . Obviously Alice can't use this aggregated signature by itself to unlock any funds(because although it is a valid signature, it is not a correct one!), but she can compute the secret using a simple algebraic operations: $t = s_{ab'} - s_a - s_b = s_a + s'_b - s_a - s_b = s_b + t - s_b = t$. Later on she can use $t$ to find the correct signatures and use those and complete the payment.

##### Atomic Swaps with Adaptor Signatures
    
        
---    
 **References:**   
- [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki)  
 - [What The heck is Schnorr?](https://medium.com/bitbees/what-the-heck-is-schnorr-52ef5dba289f)-Rajarshi Maitra  
 - [What are the advantages of Schnorr vs ECDSA?](https://bitcoin.stackexchange.com/questions/77234/what-are-the-advantages-of-schnorr-vs-ecdsa)   
 - [](https://download.wpsoftware.net/bitcoin/wizardry/mw-slides/2017-05-milan-meetup/slides.pdf)
 - https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-January/015614.html
 - https://github.com/bitcoin/bips/blob/master/bip-0114.mediawiki
 - https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki
 - https://en.bitcoin.it/wiki/Multi-signature
 - http://coders-errand.com/malleability-ecdsa-signatures/
 - https://github.com/ElementsProject/scriptless-scripts
 - https://medium.com/crypto-garage/adaptor-signature-schnorr-signature-and-ecdsa-da0663c2adc4
 - https://medium.com/crypto-garage/adaptor-signature-on-schnorr-cross-chain-atomic-swaps-3f41c8fb221b
 - https://bitcoinmagazine.com/culture/the-who-what-why-and-how-of-the-ongoing-transaction-malleability-attack-1444253640
 - https://www.youtube.com/playlist?list=PLPrDsP88ifOVTEJf_jQGunDUS05M9GdIC
 - https://bitcoinops.org/en/topics/adaptor-signatures/#:~:text=Adaptor%20signatures%20(also%20called%20signature,the%20adaptor%20reveals%20the%20signature.
 - https://murchandamus.medium.com/2-of-3-multisig-inputs-using-pay-to-taproot-d5faf2312ba3
 - https://eprint.iacr.org/2018/472
 - https://bitcoin.stackexchange.com/questions/111169/what-is-an-adaptor-signature
 - https://medium.com/@BR_Robin/basic-taproot-wallet-with-script-path-spend-c41f3f648a5a
 - https://github.com/bitcoinops/taproot-workshop
 - https://github.com/bitcoin/bips/blob/master/bip-0086.mediawiki
 - https://bitcoinops.org/en/newsletters/2019/05/14/#overview-of-the-taproot--tapscript-proposed-bips
   
 ---