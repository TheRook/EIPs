## Simple Summary
A ditto transaction allows for a user to duplicate any previously confirmed transaction.  A transaction has multiple fields that maybe useful to re-use, which includes script code that makes up the smart contracts. A ditto, or copy of an existing transaction is the smallest form of a signed signature that can be used to convey value across the Ethereum network. A smaller transaction size reduces gas prices and frees up more confirmation bandwidth.

## Abstract
If some previous transaction shares simulators with a transaction a user is about to make, then fewer bytes of data are needed to represent this transaction to the blockchain. Because previously confirmed transaction are immutable and global, they can be referenced and reused.

## Motivation
Users are motivated to use ditto transactions as they pay a lower gas fee.  With the advent of DeFi smart contracts, some payments can be repetitive - such a periodic loan re-payment or payment of a regular salary.  Periodic transactions will gain the most from lower gas fees with ditto transactions over other transaction types. Additionally, if another user executed a very large contract - this script could be copied for a fraction of the cost because scripts can be re-run without being forced to re-define them.

## Specification
Existing ethereum transactions conform to the following schema:
[nonce, gasprice, startgas, to, value, data, v, r, s]

A ditto transaction is similar, Using a null byte, we will signify that this RPL structure is empty and to use the values found in the referenced transaction, as shown below:

RPL
[ditto_id, nonce, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, r, s]

The ditto_id needs only to refer to a unique transaction that has occurred, and it must be a small as possible.  The ditto_id could be the transaction number, or the absolute value starting from the 0th transaction on the genesis block. Using a single integer to represent the ditto_id will result in smaller transaction sizes.

Using this ditto transaction type, a user can replay an existing script with an new source account and destination by overriding specific fields:
[ditto_id, nonce, 0x00, 0x00, to, 0x00, 0x00, v, r, s]

To reduce the transaction size even further we can remove some elements if the transaction being duplicated is owned by the user.  This transaction type would be any source that has a periodic payment. In this case, the smallest possible transaction would be just four elements; nonce, and 'r' and 's' signature values.  The 'v' key lookup parameter can be taken from the  referenced transaction:
[ditto_id, nonce, r, s]

The value 'v' is needed to recover the public key, which can obtained from the previous transaction referenced by the ditto id.

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
