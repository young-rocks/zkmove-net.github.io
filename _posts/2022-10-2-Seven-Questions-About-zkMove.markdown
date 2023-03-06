---
layout: post
title:  "Seven Questions About zkMove"
cover-img: /assets/img/blog/blog1.webp
thumbnail-img: /assets/img/blog/blog1.webp
date:   2022-10-04 21:32:44 +0800

categories: Questions

---
zkMove has been increasingly attracting people’s attention recently, and as the founder of the project, I get misc questions from time to time. Although the backgrounds of the questioners varied, the questions raised up were quite similar. So I would like to take this chance to answer those questions in one post and I hope it could help to illustrate a big picture of zkMove.
<!--more-->

First of all, please allow me to start with a self-introduction. I have taken a full time job in the crypto domain for three years, focusing on the development of virtual machines for smart contracts. Before that, I worked in Intel and Alibaba for many years, mainly engaged in the development and optimization of programming language virtual machines and system software.

## Q1. What is zkMove?
zkMove is a smart contract runtime environment. Just like the Java runtime environment is used to run Java programs, zkMove is used to run smart contracts written in the Move language. Like the Java runtime environment, zkMove also relies on the underlying operating system. The difference is that instead of Windows, MacOS or Linux, the operating system here refers to different types of blockchains. zkMove is the execution layer to carry the computation, and the blockchains serve as the settlement and consensus layer.

Unlike the native Move runtime environment, zkMove is a zero-knowledge proof friendly runtime environment that generates zero-knowledge proofs while executing Move contracts. Zero-knowledge proofs can be used to quickly verify that the execution of a smart contract has not been tampered with for the purpose of instant settlement.

## Q2. The concept of zero-knowledge proof is too difficult to understand, can you explain it in an easy-to-understand way?
Zero-knowledge proofs are the most cutting-edge area in cryptography, which is mainly used for verifiable computations. An analogue of zk proof to computation is a digital signature to data message. We all know the use of the Hash function, which compresses a message of arbitrary length into a fixed-length digest, with which it is possible to verify that the message has not been tampered with, with very little overhead. Zero-knowledge proofs is similar to Hash in that it targets “computation” rather than “data”. Once the proof is generated, it is possible to verify that the computation has not been “tampered with” with very little overhead, without having to run the program again.

There is a common doubt about zero-knowledge proofs. The verification of zero-knowledge proofs is very fast, but the process of generating proofs is very time-consuming, and using it does not seem to be worthwhile from the energy consumption point of view. This question can be thought of in a different way. The blockchain consists of thousands of nodes that need to execute all transactions on each machine in order to reach consensus. If zero-knowledge proofs is used to verify the validity of a transaction, there is no need to execute the transaction on each machine. Although it is more time consuming to generate proofs on one machine, it is still cost effective compared to the total energy saved on all machines.

There are two main barriers to the popularity of zero-knowledge proofs, one is performance and the other is programmability. The performance problem can be solved by parallelism and hardware acceleration; the programmability problem can be solved by zero-knowledge virtual machines. For arbitrary programs, zero-knowledge virtual machines can automatically generate arithmetic circuits. zkMove is a typical zero-knowledge virtual machine.

## Q3. Why Move and not other languages?
This is related to my work experience. I was one of the first group of Move users, starting from 2019, when Facebook released Libra for the first time. The first app I wrote with Move was a Gobang game. For the first e-book about Move published by Damir Shamanaev,I am the Chinese translator. I was the core developer of Starcoin, the first Move public chain launched one year ago. Having been involved in the evolution of the Move language up close and personal for so long, I’ve gained a deep understanding of how great Move is. Every time I see news about a crypto coin being stolen, I wish one day these applications would run in the Move runtime environment. That’s why I chose Move over other languages.

## Q4. What features have been completed in the project? What are the next steps?
zkMove is still in its early stages and a stack-based bytecode virtual machine has been implemented. The objects stored in the operand stack, local variables and global state are finite field elements. On this basis we have implemented a VM circuit. It is assembled from several sub-circuits, including an execution circuit, a bytecode circuit, and a memory circuit. The execution circuit is used to constrain each instruction to be executed correctly, the bytecode circuit verifies that the correct bytecode is loaded, and the memory circuit ensures memory consistency. The execution circuit already supports stack operations, local variable reading and writing, arithmetic operations, logical operations, jumps, function calls, and some other instructions.

After we finish supporting more instructions, we will try to access representative public chains in the Move ecosystem in the form of Rollup, and we will also try to connect to the EVM ecosystem. The specific solution is still under investigation and design, and we will focus on real user needs and be able to support financial, gaming, and some privacy-sensitive application scenarios.

## Q5. What are the advantages of zkMove compared to its competitors?
I haven’t heard of any other team working on a zero-knowledge virtual machine for the Move language yet. I know a project which tries to convert Move to Yul, then convert Yul to Miden assembly (Move -> Yul -> Miden). The zero-knowledge proofs will be generated with the Miden VM. This solution leverages Miden and eliminates the need to develop Move virtual machines. Compared to this solution, zkMove has its unique value. In fact, the security of the Move language is guaranteed by the Move virtual machine. If Move is converted to Miden assembly, how much of the security features will remain? Instead, zkMove is fully compatible with the native Move virtual machine and all of Move’s security features will be inherited.

## Q6. Can you tell us a little bit about the team?
I wrote the first line of code for zkMove last July, and it’s been over a year now. As the Move language continues to climb in attention, it’s time to build zkMove into a product. We have a strong team of advisors with top experts from the programming language virtual machine, blockchain, and zero-knowledge proof fields, as well as industry leaders from the market strategy, industry partnership, and investment areas. We are building our core technical team and welcome developers in the field of blockchain, smart contracts, programming language virtual machines, and zero-knowledge proofs to join us!


## Q7. What is your vision for zkMove?
Looking ahead, uncertainty in the world is unavoidable, but some things are certain, I believe, the digitization of the world is certain, and it could probably start accelerating. Smart contracts and zero-knowledge proof technology will be at the very core of both Web3 and the Metaverse. We are committed to building the safest and most efficient runtime environment for smart contracts in the long run.

Last but not least, welcome like-minded talents to [JOIN US](/joinus/)! 
