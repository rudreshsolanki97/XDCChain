# XinFin Delegated Proof of Stake (DPOS) Consensus

Self KYC compliant Delegated Proof of Stake (DPOS) Consensus on XDC Blockchain

Delegated Proof of Stake (DPOS) is the fastest, efficient, decentralized, and flexible consensus model available. DPOS leverages the power of stakeholder approval voting to resolve consensus issues in a fair and democratic way. Self KYC feature adds more enterpise usecases and regulator friendliness to the Public network.

This document describes the specification for a XinFin DPoS (Delegated Proof of Stake) network.

# XinFin Spec
## Definitions

- __DPoS:__ Delegated Proof of Stake: a mechanism for the selection of network Validators by coin holders *delegating* their votes
- __Validator:__ a node on the network responsible for producing and validating blocks.
- __Nominator:__ a coin holder who stakes and delegates their coins to one or more Validators.
- __Epoch:__ (usually denoted $N$) corresponds to $L \in \mathbb{N}$ blocks: it is a cycle of a few blocks in which Validators create blocks in turn.


## Voting

Any network participant will be able to vote for eligible Validators.

### 1. Staking

Users can stake their coins by sending them to the `deposit` function on the staking contract.

- The staked amount must be larger than an amount `MIN_STAKE`
- The user will have to wait for 2 epochs (epoch $N+2$) before being able to vote for a delegate.

### 2. Delegating

- After 2 epochs, a Nominator can vote for a Validator with the staked coins. 
- The vote will be effective in the epoch $N + 2$, $N$ being the current epoch.
- A new vote can be cast every 2 epochs.
- In this version, the entire stake must be voted to a single Validator. If a user wants to delegate to multiple Validators they can split their stake amongst several accounts which can then individually register as Nominators.

### 3. Withdrawal

Nominators should be able to withdraw their stake.

- First they must call `dedelegate` to remove their vote.
- After a specified number of epochs `WITHDRAWAL_PERIOD` the funds are unlocked and `wihdraw` function on the smart contract can be called with a withdrawal address. 

## Validators Registration

Any network participant will be able to register as a Validator. 

- A specified value `REGISTRATION_VALUE` (in the native token) will be sent to a `register` function on the contract. It will be burnt in order to limit the number of participants.
- A hard limit to the total number of registered Validators `MAX_REGISTERED_VALIDATORS` should be specified.
- Any Validator registering will have to wait for the beginning of epoch $N+2$ (current epoch being $N$) to be eligible.
- A Validator's total stake must be greater than the `MIN_TOTAL_STAKE` in order for it to be eligible.
- The top $L$ by total stake in a given epoch are chosen as the active Validator Set: those Validators that produce blocks in the next epoch. 

## KYC

Masternode Holder need to Add Know your customer (KYC) identification as its falls under the responsibility of financial institution and/or regulated company. Validator needs to upload self KYC document and this document will be visible on open public network.

## Choosing Validators

The problem is : how to choose $L$ Validators for a certain epoch $N$?

### Balanced staked

This version of the DPoS contract should balance all the stakes by finding the minimum staked for all eligible Validators (eg. take the top 1000 Validators) and balance all the Validators stakes by refunding the users the contributions that overflowed the stake.

For example, if the minimum stake is $S$, we want to balance all stakes to $S$. If a Validator has $S+100$ stakes because of 3 contributions: $S-10$, $5$ and $100$, then the last nominator will be refunded $95$.

In this model, an epoch would be of the maximum number of Validators allowed, eg. 1,001 (+/- an hour on a 4-seconds block time chain).

## Rewards

Rewards are assigned via the [Rewards Contract](#Rewards-Contract).

- Rewards for active Validators are calculated as a percentage of total stake, `VALIDATOR_REWARD`.
- Nominators would also need to be rewarded to incentivize them to stake. There are a couple of options here:
    - The reward contract pays directly out to nominators, minus a fee payed the Validator, which could be specified when registering.
    - The Validator is resposible for calculating/paying out the rewards. This could be done by allowing the Validators to register their own Reward Contract when registering.

## Slashing

In order for the network to be secure, misbehaviours must be detected and punished.

### Off-chain

Off-chain detection of misbehaviour is easier to implement and can be used for tricky misbehaviour detections.

In the contract, there will be a `reportBenign` method (part of the Validator Set Contract) that only Validators can call, passing a message and a block-number, and a slashing will execute if more than 2/3 of the Validators agree on the misbehaviour.

These might include but are not limited to:

1. Validators consistently propagating blocks late
2. Validators being offline for more than 24 hours.

It could slash a portion of the stakes, eg. only 4%

### On-chain

A slashing condition that can be verified on-chain is that a Validator signed-off 2 blocks with the same step (*equivocation*). Anyone could call this `reportMalicious` method with the right data, being the two signatures of the Validator, and the message signed, which would contain the step of the blocks.

This method would thus include an RLP decoder.

We could also detect if a Validator hasn't signed any block for the past 256 blocks on-chain, by challenging the Validator to submit the block he signed along with the signature. However, only the last 256 block hashes are available in the EVM, so it might be limited in this context of around 1,000-blocks epochs.

There may be other on-chain slashable conditions.

## Parameters

Suggested parameter values from requirements:

`MIN_STAKE`: 1,000,000 XDC

`VALIDATOD_REWARD`: 0.01370%

`VALIDATOR_SET_SIZE`: 21

`WITHDRAWAL_PERIOD`: 7 Days

`MAX_REGISTERED_VALIDATORS`: 5000

## Upgradability: 

Contracts should be upgradeable, could be implemented with Proxy contracts. Would need to figure out a governance mechanism to determine how upgrades are authorised and enacted.

Contract state would need to be transferred to the new version of the contract, either through a migration process or a persistent storage pattern.



Programming Language: C++, Golang, Python, Rust

Consensus Related References:
Parity Tech Aura - Authority Round https://wiki.parity.io/Aura

TomoChain (DPoS Code base in Golang)

EOS

TRON

Ethereum's Proof of Work ( ethash )

Ethereum's Proof of Authority ( clique )

Ethereum's Proof of Stake ( casper )

Proof of Stake-Velocity ( POSV )

Smart contracts for PoS

Help/Questions?

Telegram Channel: https://t.me/XinFinDevelopers

Slack Channel: https://xinfin-public.slack.com/messages/CELR2M831/
Free Slack Invite LaunchPass : https://launchpass.com/xinfin-public

Forum: http://XinFin.net
