---
layout: post
title:  "Universal Halo2 Verifier for Move Ecosystem"
cover-img: /assets/img/blog/blog6.png
thumbnail-img: /assets/img/blog/blog6.png
date:   2023-12-20 12:00:00 +0800
categories: release
---


The [zkMove](https://www.zkmove.net/) developers have recently launched a project called [halo2-verifier](https://github.com/zkmove/halo2-verifier.move). Its objective is to enhance the capability of the blockchains in the Move ecosystem by enabling halo2 zero-knowledge proofs to be verified on-chain. The project has chosen aptos as a pilot and will cover more platforms in the future.

This post includes:

- What is halo2-verifier.(A description of the project)
- Call for help (Request for feedback)
- Design considerations and implementation details
- Plan for Future. (An outline of what is planned for the future)

In essence, we invite developers interested in Move and zero-knowledge proofs to try our project ([contracts are live in Aptos devnet](https://explorer.aptoslabs.com/account/0xd46fba4dab96814e4b53601a7ff6859d3ace6defb7c23091dbe28df0eb9f0442/modules/code/verifier_api?network=devnet)). You can publish a circuit to Aptos and perform on-chain proof verification by following our [guide](https://github.com/zkmove/halo2-verifier.move?tab=readme-ov-file#give-it-a-try) here.

We particularly welcome developers with large circuits or real use cases to experiment with our halo2 verifier. Some parts of the implementation have limitations that require optimization before reaching production readiness. If you encounter any issues or have feedback on our project, please share it [here](https://github.com/zkmove/halo2-verifier.move/discussions).

## What is `halo2-verifier.move`

Halo2 is a widely used plonk implementation, with projects like Zcash, Scroll, Axiom, and Taiko developed based on it. It already has a common verification process, but blockchains can't directly utilize it due to the different language.

Several teams are developing on-chain verifier (e.g. [halo2-solidity-verifier](https://github.com/privacy-scaling-explorations/halo2-solidity-verifier)) for the EVM community. While it uses generic templates to generate code, it is not a universal verifier. Developers still need to generate verifier contracts for each circuit.

halo2-verifier.move takes a different approach, it tries to extract the information of the halo2 circuit and abstract them into a Protocol of the circuit. We call it the shape of the circuit, which includes:

- Gates. Mainly the constraints you write in the circuit.
- [Lookup arguments](https://zcash.github.io/halo2/design/proving-system/lookup.html). include the input expressions and table expressions.
- Queries on each columns.
- Commitments of fixed columns and [permutations](https://zcash.github.io/halo2/design/proving-system/permutation.html).
- Other necessary informations like, `k`, `cs_degree`, `num_of_fixed_columns`.

With these information, the general verifier can read commitments and evaluations in proofs of the circuit, and do verification accordingly using a polynomial commitment scheme.

## Request for feedback

We'd like to hear from you about your experience using this verifier.  If you have any things to share, especially of the following things that we are interested in, please let us know.

- Have you run into any bugs, errors, issues, or confusing problems? Please file an issue over at https://github.com/zkmove/halo2-verifier.move.
- The first time that you use our sdk or contract api, or try our tutorial, is there any unconventional cases?
- Do you have problems especially when publishing your circuit info to aptos?
- Do you have use cases that our halo2-verifier or rust-sdk don’t cover up?
- Do you think the cost of current proof verification api is bearable for you or your users?

Or if you would prefer to share your experiences on telegram, head over to the [#zkmove](https://t.me/+21M-767LquZhNWFl) group.

## Design considerations and implementation details

(These sections are for those particularly curious.)

The implementation of halo2-verifier.move had to consider several constraints to ensure that it works well in blockchain environments.

### Publish the Protocol

In order to verify the proof of a circuit, developers need to publish the shape of circuit onto blockchain. So one big focus was to make sure that the protocol of any circuits can be published to blockchain like aptos, because the information of a `Protocol` can be very large, it varies in a large range from   ~100 to ~100k bytes based on the complexity of the circuit.

Currently, aptos has transaction size limit of 64k. so if the size of a protocol is larger than that, it cannot be published. To address the issue, we have done a gate compression trying to minimize the storage cost that the publish transaction will consume, although it cannot solve it once and for all. If your circuits are too large to be published, please let me know! We have another ongoing research on the potential problem.

### Expressing of Gates

Another challenge is how to express gates in Move. Gates is a list of constraints whose values should be equal to `0` when evaluating. The basic form of a constraint is something like this in rust:

```rust
#[derive(Clone)]
pub enum Expression<F> {
    /// This is a constant polynomial
    Constant(F),
    /// This is a virtual selector
    Selector(Selector),
    /// This is a fixed column queried at a certain relative location
    Fixed(FixedQuery),
    /// This is an advice (witness) column queried at a certain relative location
    Advice(AdviceQuery),
    /// This is an instance (external) column queried at a certain relative location
    Instance(InstanceQuery),
    /// This is a challenge
    Challenge(Challenge),
    /// This is a negated polynomial
    Negated(Box<Expression<F>>),
    /// This is the sum of two polynomials
    Sum(Box<Expression<F>>, Box<Expression<F>>),
    /// This is the product of two polynomials
    Product(Box<Expression<F>>, Box<Expression<F>>),
    /// This is a scaled polynomial
    Scaled(Box<Expression<F>>, F),
}
```

It’s represented as enumeration in rust, but enumeration doesn’t exist in move lang. We had to figure out a new way to express gates’ constraints.

Basically, constraints are some forms of sums and products of column queries, if we replace the column queries with unknown variables like `x_1`, `x_2` , `x_3`,then we get a multivariate polynomial, like this:

```rust
0x30644e72e131a029b85045b68181585d2833e84879b9709143e1f593f0000000  * x_2 * x_3 +
0x0000000000000000000000000000000000000000000000000000000000000001  * x_0 * x_1 * x_3
```

So if we turn constraints into multivariate polynomials, and encode the multivariate polynomials, then we solve the problem. It’s exactly what we did in halo2-verifier.move !

After the transformation, evaluating the constraints are just evaluating the multivariate polynomials given the corresponding `x_1`, `x_2` , `x_3`.

It’s easy implemented in aptos move, see the code [here](https://github.com/zkmove/halo2-verifier.move/blob/main/packages/verifier/sources/multivariate_poly.move). By the way, the implementation idea is originated from arkworks’ [algebra](https://github.com/arkworks-rs/algebra/blob/master/poly/src/polynomial/multivariate/sparse.rs).

### Reading Proof and Verification

The main logic of reading proof and verification is the same as that in halo2. We’d like to keep  most logic the same for the purpose of auditing and maintenance. The halo2 communities and auditors can easily know which parts of the code do which parts of job based on the origin halo2 verifier implementation. and if there’re new features in halo2, we can easily add them to this Move implementation. If you guys have any interests of the code, please look through [here](https://github.com/zkmove/halo2-verifier.move/blob/main/packages/verifier/sources/halo2_verifier.move).

The main problem we came across and solved was the different serializations of bn254 in two different cryptography libraries, [halo2curves](https://github.com/privacy-scaling-explorations/halo2curves) and arkworks’ [algebra](https://github.com/arkworks-rs/algebra). I described the details in an [issue](https://github.com/privacy-scaling-explorations/halo2curves/issues/109) under halo2curves repo which have received many comments from core devs. We intend to track the issue and push forward to form a common standards of serialization Fields and Curve points. In our projects, we did a tricky thing to solve the serialization problem, however it incurs a performance cost which we will solve in the near future with the help of aptos developers.

One important aspect of verification is the polynomial commitment scheme. In our case, we adopted KZG following scroll and axiom. In order to make it possible to verify kzg commitments, we cooperated with aptos devs, and delivered the [bn254_algebra implementation](https://github.com/aptos-labs/aptos-core/pull/11142) to aptos core repo. It’s  jolly during our cooperation, and thanks @zjma @alinush for helping and reviewing the code!

We also found some pitfalls in the crypto_algebra of aptos, which we’re going to address in the next phase.

## Plan for Future

A major aspect of this endeavor is to optimize the gas cost(minimize the storage and computation required) with using the halo2 verifier, also with a plan to extend its features to adopt optimizations on halo2, like shuffles and mv-lookup.

One example of gas cost optimization is make `pow` function native.Current version of halo2-verifier consumes a lot computation gas due to the reason that `pow` of field is implemented in pure move. `pow` on Field is a very costly operation. We intend to create a pr to add the api into aptos stdlib.

Our ultimate goal is to make halo2-verifier usable on aptos production environment, consuming a few hundreds of gas units at most, which is an acceptable gas cost for massive adopting  of onchain proof verifications.