# Multi-Signature with On-Chain Proposal State Management

**author**: "Samuel Bourque" <samuel.bourque@nomadic-labs.com>

# Overview

A Multi-Signature Contract behaves as a generalized treasury for holding assets, which can be managed by multiple sets of keys.

Real-world banking analogies are: a joint account for a couple, or a corporate account.

A MultiSig is generally described by **M of N**; i.e. `M` number of signatures out required of `N` number of actors in order to approve an operation.

# Differentiation

What makes this wallet implementation distinct from others:

 * approval is an on-chain event
 * approval status is managed on-chain
 * no changes in configuration are permitted, once initialized
 * no additional features that wouldn't feature in a straightforward wallet (e.g. feeless)

# When Does One Need a MutliSig?

 * To prevent collective assets to be absconded by any single actor
 * To observe checks-and-balances within a management group
 * When assets could/would/should not be held by an individual, when it could/would/should be held by an entity

# Use Cases

Actors who participate in a MultiSig:

 * owner: one who deploys the contract and initializes the contract's parameters--may or may not be a signer
 * signers: anyone with some degree of claim/right/authority to/over the assets contained/held within the MultiSig

## Deployment

pre-condition: no contract exists yet
effect: initialized contract

1. Alice deploys the contract;  with the following paramaters:
   * identifying herself (or someone else if she chooses) as **Owner**
   * signature threshold--i.e. how many signatures are required for a proposal to be considered approved
   * signers--the list of tz addresses for all actors who may sign

## Setting Metadata

pre-condition: contract exists
effect: metadata is set 

1. **Owner** initializes metadata using `set_metadata_uri`

## Smooth Operation

pre-condition: contract exists
effect: operation is injected 
assumes: 2 of 3 signature threshold

1. Bob--a signer--proposes an operation `O1` using the `propose` entry; he sets `the_duration` to `3d`
1. Carl--another signer--approves on the next day Bob's proposed `O1` operation using the `approve` entry
1. Alice--another signer--executes the `O1` operation using the `execute` entry

## Expired Proposal

pre-condition: contract exists
effect: operation `Op2` is not injected 
assumes: 2 of 3 signature threshold

1. Bob--a signer--proposes an operation `O1` using the `propose` entry; he sets `the_duration` to `3d`
1. Carl--another signer--approves, 4 days later, Bob's proposed `O1` operation using the `approve` entry--fails with "EXPIRED"
1. Alice--another signer--executes `01`--fails with "NOT YET APPROVED"

## Insufficient Signatures

pre-condition: contract exists
effect: operation `Op3` is not injected 
assumes: 2 of 3 signature threshold

1. Bob--a signer--proposes an operation `O1` using the `propose` entry; he sets `the_duration` to `3d`
1. Alice--another signer--executes `01`--fails with "NOT YET APPROVED"

# Restrictions

1. Parameters/Configuration can not be edited once contract is in-effect; this includes:
    * signature threshold
    * signers
1. All signatures are of equal weight

# What To Do When:

1. What To Do When we need to edit the paramaters? (e.g. Add/Remove actors to signers group)

> Deploy and Initialize a new Multisig contract and transfer assets to it.

2. What To Do When enough signers lose their keys (or are unreachable) and we cannot gain enough approval to do anything?

> Nothing can be done. If you cannot ever achieve sufficient approval, this contract will never execute any operations again... then its assets are stuck forever (unless you can recover/remember the keys)... not so dissimilar to regular wallets/accounts.

Plan accordingly to mitigate this risk.
