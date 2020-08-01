---
eip: <to be assigned>
title: Ditto Transactions
author: Michael Brooks <m@ib.tc>
discussions-to: <URL>
status: Draft
type: <Standards Track | Meta | Informational>
category (*only required for Standard Track): <Core | Networking | Interface | ERC>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <EIP number(s)>
replaces (*optional): <EIP number(s)>
---

## Simple Summary
A ditto transaction allows for a user to duplicate any previously confirmed transaction.  A transaction has multiple fields that maybe useful to re-use, which includes script code that makes up the smart contracts. A ditto, or copy of an existing transaction is the smallest form of a signed signature that can be used to convey value across the Ethereum network. A smaller transaction size reduces gas prices and frees up more confirmation bandwidth.

## Abstract
If some previous transaction shares simulators with a transaction a user is about to make, then fewer bytes of data are needed to represent this transaction to the blockchain. Because previously confirmed transaction are immutable and global, they can be referenced and reused.

## Motivation
Users are motivated to use ditto transactions as they pay a lower gas fee.  With the advent of DeFi smart contracts, some payments can be repetitive - such a periodic loan re-payment or payment of a regular salary.  Periodic transactions will gain the most from lower gas fees with ditto transactions over other transaction types. Additionally, if another user executed a very large contract - this script could be copied for a fraction of the cost because scripts can be re-run without being forced to re-define them.

## Specification
Building from EIP 2718, we can create a new typed transaction envelope.   Below is the basis transaction type defined in EIP 2718, and is the basis for ditto transactions: 

`rlp([0, rlp([nonce, gasPrice, gasLimit, to, value, data, v, r, s])])`

A ditto transaction represents all of the same values above, however it allows for the reuse of previous transaction's information which is referenced by a "transaction identifier" (txn_id).  To referrer to a transaction we need a unique ID that is as a small as possible.  The txn_idx could be implemented as the absolute transaction number starting from the 0th transaction on the genesis block.

With ditto transaction's parameter overriding, order matters because it allows a transaction to avoid a filler-byte.  Below is a small ditto transaction that will inherit all other parameters from the original request:

`rlp([d, rlp([txn_idx, s, nonce]))`

The above transaction exactly replays a previous request that this user has made. The public key information used to sign the transaction 's' can be obtained from the previous transaction referenced by the ditto id.  This is for example a regular repayment which is supported by a minified request. In a ditto transaction, any given parameter could be overridden, the full schema is shown below:

`rlp([d, rlp([txn_idx, s, nonce, to, v, r, value, gasPrice, gasLimit, data]))`

The above would have all fields populated, which wouldn't happen in real life, the user would want to "ditto" an existing script or key material from the blockchain. As an example, using the ditto transaction type, a user can replay an existing script with an new source account and destination:

`rlp([d, rlp([txn_idx, s, nonce, to, v, r]))`

The order above was chosen to support variability of what parameters are controlled without relying on a filler byte. If a parameter is missing, then it will be filled in by the copied request - except for the nonce.  If the nonce is missing then we will assume it is zero.  This is necessary to avoid replaying the nonce in the old request, which would happen with a strict duplication.  By leaving this value blank we interpret the users intension to use the Zeroth place for the nonce, and produce the smallest possible request of two elements, shown below:

`rlp([d, rlp([txn_idx, s]))`

With parameters overriding, the smallest possible translation only needs two parameters; a transaction index and a signature.  In order to replay this request a 2nd time, a nonce value will need to be supplied - which will increase the transaction by 2 bytes. 

## Rationale
We need to do whatever we can to reduce transaction sizes.  Adopting compression techniques of using existing transaction as a kind of code book that can be re-used by future transactions will reduce the repetitive nature of blockchain transactions.  Key material and binding smart contract data can be referenced with a kind of pointer, or "ditto id" that frees up blockchain bandwidth to encode unique data instead of repetition.

## Backwards Compatibility
It isn't.

## Test Cases
todo.

## Implementation
todo.

## Security Considerations
Many of the security fundamentals of Ethereum remain unchained.  Transactions still need to be signed by the authoritative private key, and a re-organization could result in a change in the perceived consensus. Users should not make ditto transactions if there is a threat of a re-org.

In order to build a ditto transaction, the user will need to be able to observe a previous transaction to copy - in practice this is likely one of their own transactions which was tracked by their client wallet - and they are simply hitting reply.   There maybe some concern if a client wallet will have to depend on a central authority to inform what ditto id to use. These types of threats are client-implementation based, and outside of the scope of this EIP.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
