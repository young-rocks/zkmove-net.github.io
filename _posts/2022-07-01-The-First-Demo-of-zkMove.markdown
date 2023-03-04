---
layout: post
title:  "The First Demo of zkMove"
cover-img: /assets/img/blog/blog2.webp
thumbnail-img: /assets/img/blog/blog2.webp
date:   2022-07-01 21:00:44 +0800
categories: demo
---

Last week, I gave a demo at the Move community’s weekly meeting. It was the first public appearance of zkMove. We had a comprehensive discussion about the features of zkMove, which I will summarize briefly here with this post.
<!--more-->



## The First Demo of zkMove



> Hello, everyone! This is Guangyu Zhu, the author of zkMove. Last week, I gave a demo at the Move community's weekly meeting. It was the first public appearance of zkMove. We had an active discussion about the features of zkMove, which I will summarize briefly here with this post.



### What‘s zkMove?

zkMove is a zk-proof friendly Move language runtime environment. The initial idea of zkMove is to improve the programmability and composability of zk-proof with Move language. Developers can build scaling and privacy solution based on it. Below is some highlights of zkMove.

- **A zero-knowledge proof-friendly Move bytecode virtual machine**. As a new generation of programming language for digital assets, Move ensures security of assets at the language level. Further more, zkMove is compatible with the community Move VM at bytecode level. This makes it possible to use it as a proof of validity for the transaction blocks at layer1.

- **Powered by Halo2**.  Halo2 uses Plonkish arithmetization, which support custom gate and lookup argument, is suitable for constructing complicated circuit. No trusted setup required

- **No compromise on performance while pursuing Turing completeness**. Two types of circuits are combined: VM circuits to handle conditional branches and loops, and Move circuits, which directly compiled from bytecodes, offer smaller proof size and shorter proving time.



### High-level Architecture

To better understand the demo, let's take a look at the high-level architecture of zkMove. Before we execute a transaction, it is assumed that the smart contract should have already been published on zkMove. The proving key and verification key are generated at the time of publishing.



![zkmove-arch](/assets/img/blog/zkmove-arch.png)



On zkMove, a transaction takes three steps to complete execution. The first step is to execute the transaction and generate a witness. The witness includes the transaction itself, the root of the old state tree, the accounts involved in the transaction and the associated Merkle proof. Furthermore, the witness includes the execution trace of the transaction.

In the second step, a universal or a smart contract specific circuit is selected depending on the configuration. The root of the new state tree is passed to the circuit as the public input. Then, the witness, the public input, and the circuit are sent to the prover to generate zk proof for the transaction.

Finally the zk proof and the public input will be sent to the verifier, a smart contract on the chain, to verify the zk proof. If it passes, the new state root will be recorded on the chain.



### VM circuit vs. Move circuit

As I mentioned above, one of highlights of zkMove is the performance. VM circuit provides Turing completeness, while Move circuit provides smaller proof size and shorter proving time.

Inspired by Tinyram and zkEVMs, we construct VM circuit to verify the consistency and integrity of each step in execution trace. It can be broken down into tree parts:

- The right bytecode is loaded.
- Each bytecode was executed properly.
- Each load from locals, stack and global state retrieves the last value stored there.

Halo2's plonkish arithmetization provides custom gates and lookup arguments. When a circuit encounters some expensive operations, it can outsource the work to other circuits by lookup. This makes it possible to construct a complex circuit to verify the execution trace of the Move VM.

The VM circuit is assembled from multiple sub-circuits, including the execution circuit, the bytecode circuit, and the memory circuit. To verify that the right bytecode is executed, we look up the bytecode table in the execution circuit and use the bytecode circuit to constrain the bytecode table is the same as the smart contract. To verify memory coherence, we look up the read/write table in the execution circuit and use the memory circuit to constrain each load from locals or stack retrieves the last value stored there. This is VM circuit.

Unlike the VM circuit, Move circuits are smart contract specific circuits. The bytecode of the smart contract is "compiled" directly into the circuit. Constraints are automatically applied by the virtual machine.



![move-circuit](/assets/img/blog/move-circuit.png)



To handle conditional branching, ProgramBlock was introduced to express the control flow. However, loop statements are not supported at the moment. So Move circuits are Turing incomplete and are suitable for simple transactions such as token transfers.

### Demo and Performance

The implementation of the circuits is a bit complicated, we will introduce them in a separate document, now is the demo time!

We have a [CLI](https://github.com/young-rocks/zkmove/tree/master/demo) and some examples to demonstrate the functionality of the zkMove virtual machine. Below command will first compile *add.move* into bytecode, execute the bytecode to generate an execution trace, then build the circuit and setup the proving/verification key, and then generate a zkp for the execution with the proving key and finally verify the proof with the verification key.

```
bin/zkmove run -s examples/scripts/add.move
```

The Move program consists of scripts and modules. For testing, directive 'mods' can be added to script source file to import a module. Also, we can pass arguments to a script with directive 'args'. For example,

```rust
/// call_u8.move

//! mods: arith.move
//! args: 1u8, 2u8
script {
    use 0x1::M;
    fun main(x: u8, y: u8) {
        M::add_u8(x, y);
    }
}
```
And we need tell vm where to load the module with option '-m':

```bash
bin/zkmove run -s examples/scripts/call_u8.move -m examples/modules/
```

zkMove supports two types of circuits: the vm circuit and the move circuit. By default the vm circuit is used, which is Turing complete but slow. Move circuit is faster than vm circuit, but it's not Turing-complete. We can enable the move circuit via the command option "--fast-mode".

```bash
bin/zkmove run -s examples/scripts/add.move --fast-mode
```

Below is some prelimiary performance data collected on MacOS Catalina with 2 GHz Quad-core Intel Core i5 and 16 GB RAM. We can see Move circuit is ~100 times faster than vm circuit.


![data](/assets/img/blog/data.png)

**Note** - zkMove is still under heavy development, there is lots of limitations and issues. After we support global state, the gap between the Move circuit and the VM circuit should be much larger than 100 times. Also, we don't recommend you do performance comparisons with other zk VMs as that can be misleading. I will update the data in time once we complete the missing features.

### Limitations and issues

The project is still in an early stage. There are many limitations and issues in the virtual machine. For example, there is a lack of support for global state and only simple data types are supported. Moreover, the performance of both the circuit and the underlying proof system needs further improvement. Once these issues are well taken care of, we will make the source code available. Stay tuned!

