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
