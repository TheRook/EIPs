---
eip: <to be assigned>
title: Authenticated Broadcast Messages
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
This standard builds off of “EIP-627: Whisper Specification” and addresses concerns around whisper messages lacking a method of authenticity. By levering the blockchain as a ground truth an authenticated messaging layer can be built on top of whisper. 

## Abstract
By the EIP-627 specification, not all whisper wire messages are created equal.  Each whisper message has an associated proof-of-work that establishes a value of the message. When the network is inundated with messages, then nodes will cull lower-value messages. Anyone can send a message to any publishing channel, and include any information they so choose. 

Blockchains are not subject to proof-of-work pre-computation attacks because newly formed blocks must contain information about the previous block.  Authenticated messages take this a step further, and also includes information about a transaction that emitted the message. 

## Motivation
This standard leverages the blockchain to create a provable author of a given whisper wire message.  We introduce singing into the message schema, as well as enable the blockchain to act as a whisper message signer on behalf of a smart contract.  Giving smart contracts the authority to send out whisper messages that signal on their behalf opens the door for more interesting side-effects that can be built from these trusted signals.

There is also a motivation to improve the security of a fledgling standard, as authenticated messages are not subject to flooding attacks or misrepresentation.

## Specification
All authenticated whisper messages that are broadcasted to the network need to include a few extra parameters.  This includes the current BlockHash, a public-key and a signature.  Authenticated whisper messages need to conform to the following schema:

[Version, BlockHash, TTL, Topic, AESNonce, Data, EnvNonce, v, r, s]

They must include the most recent blockhash, which is there to ensure that the signature could not have been pre-computed. Additionally, the 'Expiry' has been removed, because the TTL serves this purpose.  The TTL is now an integer value in the number of blocks that the message is active for (e.g. 4 block or ~24 seconds.).


`rlp([Version, BlockHash, TTL, Topic, AESNonce, Data, EnvNonce, v, r, s])`


A whisper message that contains a txn_id, must be signed by the public key of the miner who executed the smart contract, and they need to refer to which transaction emitted the whisper message, as shown below:

`rlp([Version, txn_id, BlockHash, TTL, Topic, AESNonce, Data, EnvNonce, v, r, s])`


The above massage can only be signed by the miner who mined the BlockHash, and this can be verified.  The txn_id takes the place of the proof-of-work, and we want to avoid double-taxation. The hard work has already been paid by mining the block - an additional proof-of-work is unnecessary as this does not provide any additional security guarantee.  We should assume any whisper message that has been emitted from a smart contract should get the highest priority of delivery and work despite a temporary flood in the whisper messaging relay system. These messages have a TTL which is dependent on the gas paid in the transaction.  Any smart contract should be able to send a useful message free from charge, or for a nominal fee like the Post Office. If a contract has a use case that needs a higher TTL then additional gas should be paid to the network.

A smart contract can instruct a message to be emitted on its behalf by a new opcode, BROADCAST, which accepts two elements, a data segment and a string representation of the ‘Topic’

`OPCODE Data Topic`


`BROADCAST "hello world" "new in town"`


A miner executing this smart contract, will include the necessary fields to transmit an authenticated whisper broadcast message on behalf of this transaction.  This allows the script to transmit any data segment to a dapp or 3rd party that is subscribed to this broadcast.

## Rationale

This way, we know that any message that originates from a txn_id *must* have been built by a trusted party, and this globally verifiable.  At the time of the execution of the smart contract, the author of the contract isn't available to provide his private key material - nor should he be.  In this case, the contract is being executed by a selected miner who is entrusted with building a new block. In order to perform the action of block creation and smart contract execution, they can also include attribution and identity to messages emitted from the execution of smart contracts

## Backwards Compatibility
It isn't.

## Test Cases
todo.

## Implementation
todo.

## Security Considerations
Any implementation of the Whisper protocol defined in EIP-627 is subject to a denial of service attack, and this is mentioned in the specification.  No aspect of EIP-627 prevents pre-computation of the proof-of-work, thereby an attacker is free to compute these over a period of time in order to shut down the network with very high proof-of-work values.  This is a threshold abuse condition, whereby one actor can raise the messaging proof-of-work threshold above what is reasonable for the network to function.  The proof-of-work used by whisper is different from the proof-of-work used in a blockchain for this reason - but it doesn't have to be this way.  Authenticated whisper messages require the inclusion of the current blockhash, which cannot be prior knowledge thereby preventing pre-computation down to a window of only a few blocks (6-20 seconds) instead of no limit.

By providing authenticated messages, these can be sent on a preferred channel.  The number of messages originating from the contracts present in a newly formed block are finite, and should not encumber the network.  If there is an active flood of invalid or unimportant messages with a very high proof-of-work, then smart-contract originated messages should be unpreterperted and be delivered seamlessly.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).


