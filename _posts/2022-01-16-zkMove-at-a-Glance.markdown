---
layout: post
title:  "zkMove at a Glance"
cover-img: /assets/img/blog/blog3.webp
thumbnail-img: /assets/img/blog/blog3.webp
date:   2022-01-16 22:32:44 +0800
categories: glance
---

## zkMove at a Glance



With the boom of Defi and the emergence of non-financial smart contracts, the scalability of public chains, represented by Ethereum, is being increasingly challenged. Although technologies such as POS and sharding can improve the throughput rate to a certain extent, the root cause of congestion still exists in the long run. This is because any transaction that wants to be on the chain requires most nodes across the network to verify its validity, and the method of verification is to execute the transaction repeatedly. As the number of applications grows exponentially and the logic of smart contracts becomes more complex, the computational resources required to verify their validity will increase exponentially, which is reflected in network congestion, high transaction fee, and so called "trading volume spillover".

To fundamentally improve the scalability of the blockchain, we propose zkMove - a zero-knowledge proof-based smart contract runtime environment that combines Move, the most secure smart contract programming language, with PLONK, a maturing zero-knowledge proof technology, to "Move" computation from on-chain to off-chain, significantly improving the scalability of blockchain while ensuring security.

As a new generation of smart contract runtime environment, zkMove offers the following benefits in addition to scalability.

> Security

Move is a smart contract programming language developed by Meta for the Diem blockchain. As a new generation of programming language for digital assets, Move introduces a concept of ownership similar to that of Rust, enabling the definition of resource types that satisfy linear logical semantics and guaranteeing at the language level that assets can only be transferred and can never be copied or discarded.

> Programmability

Most of the current zero-knowledge proof based scaling solutions need to write a separate circuit for each smart contract deployment, and can only support a single application scenario, so the versatility or programmability is relatively poor. Several projects are trying to build a universal zero-knowledge proof scaling solution, among which Starkware and Matter labs' solutions are already deployed on the testing net. Learning from these projects, zkMove virtual machine can automatically generate circuits when deploying Move smart contracts and generate zero-knowledge proofs when executing the contracts. This allows the blockchain to verify the validity of a transaction through zero-knowledge proofs, eliminating the need to execute the transaction repeatedly.

> Privacy

We know that the content of a blockchain ledger is completely public. A ledger with fully public content is very limited in its use. No one is going to use a will repository, a voting system, or a medical information system where the data is completely public. With zkMove, we can use sensitive information as hidden input parameters for smart contracts, so that private data is not exposed.

Next, we will take zkMove's most typical usage scenario, zk-rollup, as an example to illustrate how it works. In order to "Move" the computation from on-chain to off-chain, user transactions need to be executed off-chain in the order they are submitted, and zk (short for zero-knowledge) proof and encoded operation records are generated, and then the run results and zk proof are submitted to the chain. A smart contract on the chain verifies the zk proof, and if it passes, the user's transaction is indeed executed correctly, and then the latest Merkle root R' is recorded. The encoded operation record is uploaded to the chain as the parameter of the smart contract verifying the zk proof. Below diagram depicts a typical workflow of zkMove.



![zkmove_arch](/assets/img/blog/zkmove-arch.png)



At its core is a stack based Move language bytecode virtual machine. The witness is the input to the transaction, which typically contains the accounts involved in the transaction, the Merkle proof, and the root of the state tree before the transaction is executed. The public data is the output of the transaction, which typically contains the root of the new state tree after the transaction is executed. zkMove uses the PLONK zero-knowledge proof algorithm, which requires only one trusted initial setup, and the proving key and verification key are generated at the time of the smart contract is published.

As of this posting, we are nearing completion of the first phase of the POC for the zkMove zero-knowledge proof virtual machine, where non-Turing-complete Move smart contracts can execute correctly and their zk proofs can be generated and verified correctly. In the next phase, we will improve the existing features and reach Turing-complete, so stay tuned for more details!


