---
title: Staking Protocol
hide_title: true
sidebar_position: 1
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Staking Protocol

This document describes the staking protocol in Axon, which is critical for ensuring the security and stability of the network. Validators run nodes and are in charge of participating in the consensus process. Anyone can engage in the staking as either a staker, who deposits their Axon Tokens (ATs) hoping to become a validator, or a delegator, who entrusts their tokens to a validator candidate. Kicker selects the stakers with the highest stakes to become future validators. An incentivized staker selection mechanism is correspondingly developed to promote honest behaviors among all participants.

## Participants

### Seeder

Seeder is the founder who can create and issue ATs, and initialize chains based on Axon. The initialization requires the collaborative efforts of one or more individuals to determine the validators and initiate the operation of the chain.

### Validator

Validators play a critical role in Axon's consensus mechanism by packaging transactions into blocks, ensuring the smooth operation of the chain, and earning rewards. It is important to note that validators are bounded by a defined tenure and a numerical limit, which regulate their involvement within the system.

### Staker

Stakers are participants who intend to be validators by meeting specific criteria. Any individual can become a staker by staking a certain amount of ATs and successfully passing the validator selection process.

### Delegator

Considering the expenses associated with running a node, some participants prefer to earn rewards from their holdings of ATs without actively engaging in node operation. These individuals can delegate their tokens to stakers they have selected, thereby augmenting the likelihood of the latter being chosen as validators and consequently sharing in the earned rewards. By becoming delegators, participants can passively benefit from their AT holdings while supporting the network's operation through incentivized staker selection.

### Kicker

Within the Axon ecosystem, a kicker plays the permissionless role to address various critical tasks, including:

- Monitoring the liveness of any Axon-based chain and promptly reporting its status (aka., checkpoint) to Layer 1.
- Facilitating the selection for the upcoming set of validators (aka., metadata) when the previous tenure concludes
- Ensuring the accuracy and timeliness of the information regarding the latest staking and delegating allocations

The kicker is pivotal in maintaining the integrity and smoothness of network operation by actively overseeing chain status, validator selection, and staking management.

## Key Concepts

- Epoch: The duration of a validator set, lasting for a fixed number of `x` Axon blocks.
- Period: A fixed interval of `y` Axon blocks, after which one of the validators must submit the latest state to Layer 1. When choosing the parameters, it is ensured that:
  1. `x` is divisible by `y`, i.e., `y|x` , so that an epoch consists of a specific number of checkpoints.
  2. `(x / y) mod validator_len` must be `0`.
- Quorum: the maximum number of validators in an epoch.
- Metadata: includes the essential information of Axon, such as the validators’ history of proposal counts, and consensus configuration. Metadata is stored in Layer 1.
- Checkpoint: is submitted by an Axon node to a kicker at the end of each period. Checkpoint includes the latest state of Axon, such as the state root, block hash, proposal count, and BLS signature of the latest block.
- Commission Rate (per validator): the staker and delegators assign a percentage of the block rewards based on the amount of ATs.
- Delegate Threshold (per validator): the minimum amount of delegated tokens that a staker accepts from each delegator. It is set by the staker.

## Overview
<img src={useBaseUrl("img/staking/Staking protocol overview.png")} style={{ maxHeight: '800px', display: 'block', margin: '0 auto', }} />

The entire protocol can be divided into six sub-protocols according to their functionalities: initiation, staking, delegation, validator selection, checkpoint recording, and reward distribution. Initiation is executed only at the beginning of the protocol’s lifecycle, while the other five are carried out continuously and repeatedly.

## Deep Dive

### Initiation

In this stage, seeder assumes a significant role by prescribing the metadata for the first two epochs and initiating the chain. As the primary architect and visionary of the network, founder establishes the foundational parameters and sets the trajectory for the early development and progression of the chains built on Axon.

### Staking

Participants in Axon enjoy the flexibility to stake their ATs as they wish and to engage in the network at their convenience. However, it is essential to recognize that the analysis of the total staked amount is conducted on an epoch-by-epoch basis. Each epoch serves as a time unit during which all staking records are consolidated into a comprehensive sum, capturing the collective staking activities of the participants.

Notably, the effects of staking become apparent after a two-epoch duration has transpired. This deliberate delay ensures that the network can synchronize and account for all relevant staking activities before implementing changes. Meanwhile, each staker has to set a dividend ratio and a minimum delegate threshold. 

At the onset of epoch `n`, a kicker calculates the list of validators for epoch `n+2`. This selection process adheres to a well-defined and predetermined protocol, ensuring fairness and integrity. Consequently, only a maximum of quorum stakers, as determined by the protocol, will be selected as validators for epoch `n+2`. Once the staked tokens become effective, they remain valid until the validator‘s qualification expires.

It is crucial to note that if a participant's staked tokens fail to be selected for validation, the entire staked tokens will be promptly returned to the respective participants. This mechanism safeguards participants' interests and ensures that non-selected tokens are not locked or inaccessible, allowing for continuous flexibility and engagement within the Axon ecosystem.

### Delegation

Participants have the option to delegate or redeem their ATs to or from a staker. The delegated amount must exceed the minimum delegated threshold set by the corresponding staker. Similar to the staking protocol, these operations come into effect after two epochs, ensuring a consistent and synchronized process. Moreover, the statistical analysis of the total delegated amount is performed on an epoch-by-epoch basis.

If the designated staker fails to be selected as a validator for the subsequent epoch, all the delegated tokens will be promptly returned to the participant who initiated the delegation. This mechanism safeguards participants from potential losses.

### Validator Selection

To ensure the seamless operation of the ecosystem, kicker is responsible for determining the metadata for epoch `n+1` at the beginning of epoch `n`. As articulated in the above passage on Staking, the staked tokens only take effect from epoch `n+2` onwards. Consequently, the tokens effectively staked in epoch `n+1` remain frozen from the start of epoch `n`.

Upon the completion of each epoch, validators are entitled to receive rewards for their contributions within the Axon ecosystem. However, it is important to note that these rewards are initially locked and remain inaccessible for a duration of two subsequent epochs. Once the rewards are unlocked, they are distributed to both the validators and their delegators.

Since validators from epoch `n` still possess the staked tokens, the candidate set for epoch `n+1` is composed of both the validators from epoch `n` and stakers who newly joined in epoch `n-1`. To select the validators for epoch `n+1`, candidates are sorted based on their staked amounts. Stakers with the top quorum from this sorted list are chosen as validators for epoch `n+1`, and their updated validator statuses are recorded on-chain. The staked tokens of the remaining non-selected candidates are promptly returned, allowing them to withdraw and re-stake later. If the number of candidates is less than the required quorum, all candidates automatically become validators for epoch `n+1`. Smooth chain operation can be thus ensured even in such cases.

<img src={useBaseUrl("img/staking/staking protocol validator selection.png")} style={{ maxHeight: '800px', display: 'block', margin: '0 auto', }} />

This meticulous selection process guarantees the presence of a set of reliable validators for each epoch, enhancing the stability and integrity of the system while allowing for efficient participation and token management for the participants.

### Checkpoint Recording

Checkpoint represents the most recent state of the system, which is periodically reported to Layer 1. It encompasses several essential components, including:

- Epoch: The current epoch number, indicating the stage of the system's progression.
- Period: The current period number, signifying the subdivision within an epoch.
- State Root: The state root of the Axon block header at the end of a period, encapsulating the current state of the chain.
- Block Height: The Axon block number at the end of a period, numerically representing the block's position within the chain.
- Block Hash: The Axon block hash at the end of a period, serving as a unique identifier representing the integrity of the block.
- Timestamp: The exact time at which the Axon block was finalized at the end of a period.
- Propose Count: The count of blocks proposed by each validator since the beginning of the current epoch, quantifying the participation and contribution of validators during the epoch.

Together, these elements constitute the checkpoint. As a crucial reference for Layer 1, checkpoints enable periodic updates and efficient monitoring of the state and progress of Axon’s ecosystem.

### Reward Distribution

Upon the completion of each epoch, validators are entitled to receive rewards for their contributions. However, these rewards are locked and remain inaccessible for two subsequent epochs. Once unlocked, the rewards are distributed to the validators and their delegators.

To ensure fair distribution, rewards (`r`) is divided into `i` pieces, where `i` represents the number of validators within the network. Each validator and their delegators receive an equal share, `p = r/i`, representing the total rewards allocated for the epoch.

The unlocked rewards are distributed among the participants in a manner that aligns with their token amounts. Initially, the rewards are divided proportionally reflecting the participants‘ contribution and stake in Axon. After the distribution, each delegator must deduct a commission rate from their rewards and provide them to the validator. This commission compensates the validator and incentivizes their ongoing participation.

This distribution mechanism promotes fairness, transparency, and incentivizes active participation from validators and delegators, fostering a balanced and thriving Axon ecosystem.
