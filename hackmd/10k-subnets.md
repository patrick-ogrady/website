# [DRAFT PROPOSAL] The Path to 100k Subnets: Overhauling the Relationship between the Avalanche Primary Network and Subnets

_Originally [posted to ACP GitHub Discussions](https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7373486) on 10/24/23_

Over the last few months, I've spent a lot of time thinking about what changes could be adopted on the Avalanche Network to make Subnets even more attractive to developers (especially new startups/ecosystems just getting started).

I think that the most impactful change to consider right now, as you alluded to, would be overhauling the relationship between Subnets/Subnet Validators and the Primary Network. I typed up some background on this topic for those unfamiliar and shared a three-step rollout plan that IMO would better position Subnets in the increasingly competitve "launch your own blockchain" space (could be cleaned up and turned into an ACP).

_It is worth noting that I also think native interoperability with the Primary Network is critical to attracting developers to Subnets but didn't feel it was worth elaborating on because that functionality will be provided shortly via the Avalanche Warp Messaging (AWM) integration with the C-Chain and the launch of [Teleporter](https://github.com/ava-labs/teleporter). TL;DR, this will allow Subnets to use \$AVAX or \$USDC as their native token without a trusted intermediary._

## Background

Each node operator must stake at least 2000 \$AVAX (~$20k at the time of writing) to first become a Primary Network validator before they qualify to become a Subnet Validator. All Subnet Validators, to satisfy their role as Primary Network Validators, must also allocate 8 AWS vCPU, 16 GB RAM, and 1 TB storage to sync the entire Primary Network (X-Chain, P-Chain, and C-Chain) and participate in its consensus, in addition to whatever resources are required for each Subnet they are validating. Although the fee paid to the Primary Network to operate a Subnet does not go up with the amount of activity on the Subnet, the fixed, upfront cost of setting up a Subnet Validator on the Primary Network deters new projects that prefer smaller, even variable, costs until demand is observed. _Unlike L2s that pay some increasing fee (usually denominated in units per transaction byte) to an external chain for data availability and security as activity scales, Subnets provide their own security/data availability and the only cost operators must pay from processing more activity is the hardware cost of supporting additional load._

Regulated entities that are prohibited from validating permissionless, smart contract-enabled blockchains (like the C-Chain) can't launch a Subnet because they can't opt-out of Primary Network Validation. This deployment blocker prevents a large cohort of Real World Asset (RWA) issuers from bringing unique, valuable tokens to the Avalanche Ecosystem (that could move between C-Chain<>Subnets using AWM/Teleporter).

[Elastic Subnets](https://docs.avax.network/build/subnet/elastic/elastic-parameters) enable projects to secure their Subnet and incentivize Subnet Validators with a custom staking token. However, there is no way for the broader Avalanche Community to augment the security afforded by these custom tokens with \$AVAX to help bootstrap new ecosystems (which may otherwise not have enough value at stake to secure meaningful TVL).

## Proposed Rollout of Subnet Improvements

### Step 1: Subnet-Only Validators + Subnet \$AVAX Bonding

* Remove the requirement for a Subnet Validator to also be a Primary Network Validator but do not prevent it (current behavior not deprecated)
* Introduce a new transaction type on the P-Chain for Subnet Validators that only want to validate a Subnet (`AddSubnetOnlyValidatorTx`)
* Require Subnet-Only Validators to bond X \$AVAX per validation per Subnet (to account for P-Chain Load)

Without the requirement to validate the Primary Network, the need for Subnet Validators to instantiate and sync the C-Chain and X-Chain can be relaxed. Subnet Validators will only be required to sync the P-chain to track any validator set changes in their Subnet and to support Cross-Subnet communication via AWM (see “Primary Network Partial Sync” mode introduced in [Cortina 8](https://github.com/ava-labs/avalanchego/releases/tag/v1.10.8)). The lower resource requirement in this "minimal mode" will provide Subnets with greater flexibility of validation hardware requirements as operators are not required to reserve any resources for C-Chain/X-Chain operation.

_The value of the required "bond" (X) is open for debate. To avoid impacting network stability, I think it should be at least 250-750 \$AVAX. To set this "bond" lower, I think the PlatformVM should be futher optimized (assumes that lower fees lead to a corresponding increase in Subnets)._

### Step 2: Improve PlatformVM Efficiency + Pay-As-You-Go Subnet Validation Fees

* Replace Proposal Block-based Subnet reward voting with BLS Multi-Signature votes from Subnet Validators
* Transition Subnet Validation fees to a dynamically priced, continuously charged mechanism that doesn't require locking/staking large amounts of \$AVAX upfront _(this could be broken out as a separate step but makes sense to include in the voting code refactor, so kept here)_

To increase the P-Chain capacity for processing Subnet reward distributions (which should allow fees to be parameterized more aggressively), we first should replace Proposal Block-based voting (1 Subnet reward votes per block) with BLS Multi-Signature voting (N Subnet reward votes per block). _In a future state, this mechanism may allow Subnets to implement arbitrary reward mechanisms by adding an "amount" to this signed message instead of relying on the \$AVAX reward curve that is currently used by all Elastic Subnets. @stephenbuttolph has a number of ideas about this approach 👀._

Once this vote processing change is implemented, it would be possible to just transition to a lower required "bond" amount, but many (myself included) think that it would be more competitve to transition to a dynamically priced, continuous payment mechanism. This new mechanism would target some $Y nAVAX fee that would be paid by each Subnet Validator per Subnet per second (pulling from a "Subnet Validator's Account") instead of requiring a large upfront lockup of \$AVAX.

_The rate of nAVAX/second should be set by the demand for validating Subnets on Avalanche compared to some usage target per Subnet and across all Subnets. This rate should be locked for each Subnet Validation period to ensure operators are not subject to suprise costs if demand rose significantly over time. All fees would still be paid to the network and burned, like all other P-Chain, X-Chain, and C-Chain transactions. The optimization work outlined here should allow the min rate to be set as low as ~512-4096 nAVAX/second (or 1.3-10.6 \$AVAX/month)._

### Step 3: \$AVAX-Augmented Subnet Security

* Enable unstaked \$AVAX to be locked to a Subnet Validator on an Elastic Subnet and slashed if said Subnet Validator commits an attributable fault (i.e. proposes/signs conflicting blocks/AWM payloads)
* Reward locked \$AVAX associated with Subnet Validators that were not slashed with Elastic Subnet staking rewards

Currently, the only way to secure an Elastic Subnet is to stake its staking token. Many have requested the option to use \$AVAX for this token, however, this could easily allow an adversary to takeover small Elastic Subnets (where the amount of \$AVAX staked may be much less than the circulating supply).

\$AVAX-Augmented Subnet Security would allow anyone holding \$AVAX to lock it to specific Subnet Validators and earn Elastic Subnet reward tokens for supporting honest participants. Recall, all stake management on the Avalanche Network (even for Subnets) occurs on the P-Chain. Thus, staked tokens (\$AVAX and/or custom staking tokens used in Elastic Subnets) and stake weights (used for AWM verification) are secured by the full \\$AVAX stake of the Primary Network. \$AVAX-Augmented Subnet Security, like staking, would be implemented on the P-Chain and enjoy the full security of the Primary Network. This approach means locking \$AVAX occurs on the Primary Network (no need to transfer \$AVAX to a Subnet, which may not be secured by meaningful value yet) and proofs of malicious behavior are processed on the Primary Network (a colluding Subnet could otherwise chose not to process a proof that would lead to their "lockers" being slashed).

_This approach is comparable to the idea of using \$ETH to secure DA on [EigenLayer](https://www.eigenlayer.xyz/) (without reusing stake) or \$BTC to secure Cosmos Zones on [Babylon](https://babylonchain.io/) (but not using an external ecosystem)._

## Security Considerations

* Any Subnet Validator running in "Partial Sync Mode" will not be able to verify Atomic Imports on the P-Chain and will rely entirely on Primary Network consensus to only accept valid P-Chain blocks. This ONLY affects Subnet Validators running in "Partial Sync Mode" (opt-in); Subnet Validators (or any node for that matter) that sync the entire Primary Network can and will verify all state transitions.
* Validators running in "Partial Sync Mode" that want to import a C-Chain AWM message into a Subnet (like an \$AVAX transfer) will need to rely on an external oracle to verify a message was accepted by the Primary Network (Subnet Validators just send a message to themselves if they see something interesting on the C-Chain but won't be able to do this anymore). This can eventually be remedied by requiring the Primary Network to sign all C-Chain AWM messages (then Subnet Validators could just verify an external BLS Multi-Signature).

## Acknowledgements

Thanks to @luigidemeo1, @stephenbuttolph, @aaronbuchwald, @dhrubabasu, and @abi87 for their feedback on these ideas.

# Feedback to Draft Proposal

Thanks again for taking the time to share detailed feedback on this proposal!

I did my best to collate all questions from GitHub/Twitter and responded to them in no particular order below. I'll continue to update this list as more questions come in.

## Why do this before the Avalanche Community can vote?

_Source: https://x.com/contraband001/status/1717544681565745481_

> Do you believe "deep changes" in tokenomics should be implemented before the community can vote?

The [README](https://github.com/avalanche-foundation/ACPs#what-is-an-avalanche-community-proposal-acp) specifies how ACPs are "adopted/acitivated" here:

```
The Avalanche Foundation may, from time to time, recommend specific ACPs that it believes benefit the Avalanche Network/Community but it is ultimately up to members of the Avalanche Network/Community to adopt ACPs they support by running a compatible Avalanche Network Client (ANC), such as AvalancheGo. The Avalanche Foundation's recommendation is not binding and is made without representations, warranties or guarantees of any kind.
```

When enough active consensus weight supports a feature (in this case an ACP), nodes should activate it on the Avalanche Network. "Enough" should at least be the % of stake such that the network would not become unstable if nodes that opposed an ACP dropped off the network during activation (but I would imagine the community would aim for much higher agreement than this). [BIP9](https://en.bitcoin.it/wiki/BIP_0009), for example, requires 95% of mining power over a some duration to support a "Bitcoin Soft Fork" before it is activated.

Right now, there aren't great signaling mechanisms for indicating ACP support on AvalancheGo but that is something that should be added IMO (BIP9 is an example of that this could look like). In the meantime, the README linked above provides a ["Straw Poll"](https://github.com/avalanche-foundation/ACPs#acp-straw-poll) mechanism for signaling support/opposition before code gets to the implementation phase (if there is significant opposition, it is obvious the feature won't be activated and it isn't worth the time to implement).

Lastly, I've heard some folks say "there is only one ANC right now, AvalancheGo, and I don't have any recourse to oppose". That is not true. If you oppose an ACP, simply not upgrading would prevent activation. Alternatively, AvalancheGo could be modified to support ACP signaling so the community could still take advantage of the latest optimizations while signaling opposition. The latter approach would also easily allow the opposition to peaceful concede (and run the code they opposed) if the network participants overruled their preference.

## How will BLS Multi-Signature uptime voting work?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7374316_

> If there are any additional docs or thoughts about this I'd love to read up on it. Specifically around Step 2 which talks about replacing proposal block-based subnet reward voting with BLS multi-signature votes from subnet validators.

To clarify for all readers, the transition to BLS Multi-Signature Voting is an optimization that shouldn't change the semantic behavior of staking for Elastic Subnets (validators that are online for more than the configured [`UptimeRequirement`](https://docs.avax.network/build/subnet/elastic/elastic-parameters#uptimerequirement) will still be rewarded and validators that aren't will not). It is primarily listed in this proposal because it will likely require the introduction of a new transaction type (which is an execution-breaking change that must be deployed via mandatory upgrade).

Migrating to BLS Multi-Signature voting allows the P-Chain to support more than one uptime vote in a single block. Currently, the P-Chain uses `ProposalBlocks` to vote on a node's uptime (relying on Snowman Consensus for "voting" with preset preferences per node biased on their observations of the node in question). This approach, while cool, limits voting to one node at a time, which just isn't scalable enough for where Avalanche needs to go. BLS Multi-Signature voting allows votes to be collected async (in parallel) and then just included in a block when sufficient stake has signed (many votes can be included in a single block). This also makes votes attributable, which may be useful for other mechansims.

If Primary Network validators are mandated to register BLS keys, then the same technique could be applied there as well. That would concievably open the door to much lower minimum validator stake as well (or at least make it possible to handle the uptime voting of many more consensus participants).

I don't have any code I can point to on this because the [ACP README](https://github.com/avalanche-foundation/ACPs/blob/main/README.md) first recommends getting buy-in before implementing anything but it wouldn't be too far away from some of the [polling done in the HyperSDK](https://github.com/ava-labs/hypersdk/blob/f6588c52f6982dfa0412e7679dd7e6d3006b56ee/examples/tokenvm/tests/e2e/e2e_test.go#L586-L799).

## Should PAYG fees be rebated to Primary Network validators?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7374934_

> As the load on pchain increases so should the reward for running the primary chain. I think it should be considered that payg fees are partially burnt and partially rebated to primary chain validators. As the payg subnets grow, if the primary network doesn't grow with it the incentive to run validators on the primary network would increase. There should be a situation where an entity has the opportunity to turn a cost into a revenue stream.

I think this is a very reasonable idea and love the way it directly aligns Primary Network validator P&L with the growth of Subnets and the usefulness of the Primary Network to Subnets.

Specifically, the more Subnets/Subnet Validators the more Primary Network validators get "boosted" with extra rewards (that are not inflationary). This ensures the services the Primary Network provides to the Avalanche Network (which in practice are carried out by validators) drive value for Subnets (which should increase the amount they are "boosted").

I actually had included this in an earlier draft of my proposal but removed it just because there was already so much going on lol. These ideas will likely be rolled out in multiple ACPs anyways and this should definitely be considered for the PAYG one.

## C-Chain Staking + What if the Primary Network assigned validators?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7375232_

> For subnets, a staking mechanism can be created within the C chain for a lower amount, such as 500 AVAX. Validators in the P-chain provide subnet validation. In return, the subnet owner annually delegates a fixed fee (e.g. 50 AVAX and 20% commission fee) to a minimum of 5 validators (varies depending on subnet size). Subnet staking is done with subnet tokens that are given to validators in an incentivized manner.

If an approach like this was adopted, would the P-Chain somehow be expected to provide security for the Subnets to which it assigned validators? If so, adding a mechanism that does that generically would be extremely complex/heavyweight (for one, you'd need to have some sort of standard execution result that could be validated by the chain or some sort of group of "checker" nodes). Currently, Subnet load is never processed by any nodes other than the nodes that opt-in to participate in the Subnet (why their scalability is so exciting).

Generally, Subnets are oriented around providing their own security and only use the Primary Network for lose orchestration (like looking up public keys for validators of other subnets). I think creating some feature that assigns validators to Subnets, given this context, is best done outside of the core protocol by some third-party (which could implement some sort of execution proof mechanism check on top of the protocol if they wish to slash bad behavior on the Primary Network for malicious execution).

## Will bonds be slashable?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7378010_

> Will the AVAX bond in Step 1 be slashable or will it be the native subnet token that is slashable and/or rewards withheld?

I did not envision that bond/deposit/lock in "Step 1" would be slashable. There would be no rewards to withhold, as far as the P-Chain is concerned, because it isn't securing any validation (and isn't being rewarded like Primary Network validation). Any rewards withheld on an Elastic Subnet would be in the Subnet stake token and that is independent of this lock amount.

I thought of "Step 1" as the lightest step to start moving in the direction of decoupling Primary Network/Subnets (both from a conceptual understanding and from the perspective of how much coding work would be required to pull it off). "Step 1" more or less says, "lock up X \$AVAX and get it back when you are done staking (no \$AVAX rewards, no slashing)".

## Can we use the P-Chain as an intermediary for AWM messages?

_Source: https://x.com/devilish_flux/status/1717088637219725514_

> I am worried about relying on external oracle to pass through AWM messages from C chain. Could we load P chain to be an intermediary for all AWM messages? I realize there's a host of implications to do that but the possibility seems viable.

I completely acknowledge your concern with this approach and it should not be dismissed. Secure bridges rely on secure oracles, secure key management, and secure on-chain logic (i.e. deposit contract)...if any are not present, a bridge is insecure (there are a myriad of other considerations but these are the main strata IMO).

Subnet Validators currently use their own node as a C-Chain oracle, which verifies all state transitions on the Primary Network and performs its own consensus. This approach is pretty close to optimal (from a security standpoint) for a "per validator" bridge oracle other than introducing a more conservative parameterization of k/alpha/beta (which can be done via config).

The naïve response to your concern is that Subnets that don't want to outsource oracle functionality can continue to run as they always have (rather than the new optional, partial sync mode) and validate the entire Primary Network + serve as their own oracle. This will just increase the infra cost of running such a node (because they'll need to sync X/C).

It has been suggested a number of times that the Primary Network could serve as an optional, guaranteed delivery mechanism for AWM messages for those that wish to pay. With the topology suggested in the draft proposal, the only chain where that would make sense would be the P-Chain (as you alluded to because all Subnet Validators would still process it). The tradeoff there is that the more the P-Chain is used for things unrelated to Subnet staking, the less Subnet staking can happen (capacity drops). For this reason, I've been pretty against putting more functionality on the P-Chain (and if so, making it very expensive).

The right approach to resolve this issue IMO is to require Primary Network validators periodically produce BLS signatures over the accepted tip of the P/X/C-Chains (that are periodically included in the P-Chain as BLS Multi-Signatures). This would allow AWM messages to be authenticated without syncing X/C (while still retaining security of the Primary Network) and open the door for lite clients that don't actually run Avalanche Consensus (and as such, don't need to be connected to all validators). If this technique is adopted, we'd probably want to introduce some penalty for validators that sign conflicting checkpoints (ex: deep reorg).

While it would be most impactful when all Primary Network validators have registered BLS keys (because you can wait for a simple % of stake to sign to recognize), it could be enabled as an "optional mechanism" before then to at least get some % of stake weight to attest to the validity of an outgoing message (where % could be set to some multiplier of the byzantine adversary you are defending against).

## Will there be more specifics?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7374995_

> Adding more technical specifics on the protocol changes required would help assess feasibility. For example, how would subnet-only validators sync state?

Totally agree. More specifics will need to come in any ACP that incorporates these ideas (it should be specific enough that anyone could implement it with the ACP alone). My goal with this initial post was just to outline some initial ideas and gauge interest before kicking off more focused specification/engineering work.

## Does this lower the burden to be a Primary Network validator?

_Source: https://x.com/AvaxJesus/status/1717120974959661105_

> Very nice, would this entail that anyone could validate subnet + become a 'mainnet' validator with less of a monetary pledge?

These ideas, in their current form, propose lowering the \$AVAX requirement to validate a Subnet (through the introduction of a bond-based mechanism and a PAYG-based mechanism) but do not touch on adjusting the requirement to become a Primary Network (i.e. "mainnet") validator.

## Why not just lower the minimum staking amount?

_Source: https://x.com/vpn_bro/status/1716910102689763381_

> Seems a lot simpler to instead reduce the amount of AVAX required to validate. What are the downsides here?

That's a good question. Very fair approach to push back against the obviously simple tweak.

While the community could vote to just lower the $AVAX min staking amount (it could even be considered concurrently), doing so alone does not comprehensively address the issues folks have shared about Subnets. Recall, the proposal advocates for relaxing the requirement for validating the Primary Network to directly address the following feedback, excluding "on-chain cost too high" claim: 
1) Infra Cost/Complexity: Syncing the X/C-Chains increases the cost of running a Subnet (the infra costs to run a Subnet may be ~90% lower without X/C at the beginning when state/load is small) and increases the time it takes to start running a Subnet Validator (more chains to sync on startup). Others have also reported that working with larger databases (required to support X/C), makes it more difficult to run Subnet Validators (Subnet APIs can already run in "partial-sync-mode").
2) Chain Opt-Out not Possible: Some companies that want to launch Subnets are prohibited from doing so because they aren't legally allowed to validate a permissionless, smart contract blockchain (i.e. the C-Chain).

Outside of what has been previously discussed (I'll touch on the following in any official ACP), just lowering the $AVAX min staking amount doesn't improve "fault isolation" between Primary Network <> Subnets. With the current architecture, a popular Subnet could destabilize the Primary Network if usage spikes unexpectedly (cause an OOM, disk contention, or CPU exhaustion on a Primary Network validator) or the inverse on the Primary Network (where some undefined behavior could bring a Subnet offline). Allowing optional separation IMO is a step in the right direction to making Primary Network/Subnets more resilient to regressions in the other.

## Is this Polkadot 2.0?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7393225_

> @iamnathanwindsor FYI, currently DOT is developing pay-as-you-go or "Agile Coretime" model, a sale-based method for assigning block-space, to replace auctions and parachains. This is available in their testnet I believe.Here the initial RFC: https://github.com/polkadot-fellows/RFCs/blob/main/text/0001-agile-coretime.md. It's quite different from what is presented here, so I'm not sure how useful this information is for this topic.

TL;DR, I didn't find any information in the link related to this draft proposal. I took the time to clarify the differences between this proposal and Polkadot 2.0 because I saw a few questions about it.

After this proposal was shared, I saw a lot of comments on Twitter more or less saying "isn't this Polkadot 2.0?". I wasn't familiar with their proposal and took the time to read it. Polkadot 2.0 is **very different** (as you alluded to) than what is being proposed here (other than the usage of the moniker Pay-As-You-Go, which is widely used across many industries).

The primary difference between Avalanche's PAYG and Polkadot 2.0's PAYG is the "subject" of the PAYG fee. In Avalanche, PAYG refers to the fee to register as a Subnet Validator on some Subnet on the P-Chain. In Polkadot 2.0, PAYG refers to the fee to purchase "Coretime" for the execution of "Tasks" by some subset of Polkadot Validators assigned by the "Relay Chain" on the "Relay Chain". PAYG in Avalanche is paying for the "right to participate in validation" and PAYG in Polkadot 2.0 is paying for "block-space" that is secured by Polkadot Validators.

Recall, Avalanche Subnets do not pay the P-Chain for any execution they perform (i.e. "block-space" in Polkadot 2.0) because they provide their own security. Polkadot 2.0 charges for all execution (i.e. "block-space") that happens on a "Coretime" because the "Relay-Chain" provides security and that security is paid for with \$DOT (Subnets compensate validators with custom token emissions configured by the creator).

## Are there any additional trade-offs for "Partial Sync Mode"?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7378010_

> Looking it at from the subnet owner's perspective, are there any additional trade-offs for partial sync mode, other than the ones highlighted in "Security Considerations"? Considering that C-chain will likely be the main source of liquidity on Avalanche for the foreseeable future, the requirement for the Primary Network to sign all C-Chain AWM messages should be implemented immediately to prevent reliance on third parties.

I'm not aware of any additional trade-offs but can elaborate on some additional considerations. I totally agree that having some sort of preiodic BLS Multi-Signature over the Primary Network is the right way to go (and will entirely avoid the introduction of unwanted third-party oracles). I commented on that [here](https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7425661):
```
The right approach to resolve this issue IMO is to require Primary Network validators periodically produce BLS signatures over the accepted tip of the P/X/C-Chains (that are periodically included in the P-Chain as BLS Multi-Signatures). This would allow AWM messages to be authenticated without syncing X/C (while still retaining security of the Primary Network) and open the door for lite clients that don't actually run Avalanche Consensus (and as such, don't need to be connected to all validators). If this technique is adopted, we'd probably want to introduce some penalty for validators that sign conflicting checkpoints (ex: deep reorg).
```

In terms of other considerations, @StephenButtolph called out in the most recent [Developer Community Call](https://www.youtube.com/watch?v=37sgTe0iPrs&t=3085s) that some changes to the P2P layer will be required to support the management of non-Primary Network validator IPs (AvalancheGo assumes that all IPs it needs to manage belong to a Primary Network validator) and validator rate limiting may need to be modified to introduce an allocated message handling allocation for nodes that are Subnet Validators but have no stake (above nodes that aren't registered to do anything on the Primary Network).

One additional benefit I failed to mention in the proposal has to do with better ["fault isolation"](https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7429396):
```
With the current architecture, a popular Subnet could destabilize the Primary Network if usage spikes unexpectedly (cause an OOM, disk contention, or CPU exhaustion on a Primary Network validator) or the inverse on the Primary Network (where some undefined behavior could bring a Subnet offline). Allowing optional separation IMO is a step in the right direction to making Primary Network/Subnets more resilient to regressions in the other.
```

## Should Primary Network validators only worry about "comms and consensus"? 

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7382980_

> A large pain point right now for truly "permissionless" subnets, is that each subnet is most likely going to have specialized hardware requirements, and a subnet must give a binary to a Primary Network Validator (PNV) and teach them how to operate it. This is a non-starter. All the PNV should be doing for a subnet is network comms and consensus. Can we keep the existing architecture mainly the same but allow subnets to have their own hardware (most likely in same data center as PNV) and just "connect" via RPC to their chosen validator node? Subnets could choose X distributed, unaffiliated entities as their validators. The market will figure out the best way for PNVs to connect and do deals with subnets wanting "space", or a "slot" on their PNV.

![](https://hackmd.io/_uploads/H1Wb6T6fT.jpg)


AFAIK it is not possible for a PNV to provide "comms and consensus" without also running a Subnet's Custom VM binary themself.

Recall, the VM binary informs the consensus engine, amid a bunch of other things, whether or not a block is properly formatted (i.e. "ParseBlock") and whether or not a block is valid (i.e. "VerifyBlock"). It would be possible, in this structure, for a rogue RPC to incorrectly communicate the result of either of these calls and the PNV could end up voting or even accepting an invalid block (which shouldn't be possible). The issue, more concisely put, is that consensus invariants are currently upheld by both AvalancheGo and Custom VMs (and I'm not sure it would be possible to decouple them because some checks require access to Subnet state). _Note, I'm not addressesing the scenario where a rogue PNV modifies the responses with something incorrect because that would be the same as a plain consensus failure (whereas the example I provided is a consensus failure that the validator unknowingly commits)._

Assuming that the Subnet creator is running this RPC, you may argue that they would never do such a thing. However, the whole point of the Subnet architecture is that an operator couldn't do such a thing (assuming code is not malicious). When the creator controls all the "VM hardware", it doesn't matter how many PNVs running the Subnet as security is not improved via replication (as "VM hardware" still can corrupt consensus).

If you instead intended that some collection of pseudo-Subnet Validators would run these hardware instances (so the Subnet creator doesn't control them all), it seems odd to circumvent the built-in uptime mechanism with an out-of-protocol, market-based mechanism that intends to do the same thing (assuming it would also attempt to penalize this sort of behavior). Not to mention that fault attribution would be twice as complex and require checking both the RPC operator and on the PNV, rather than just a single entity (so all messages would likely need to be signed by the RPC operator and wrapped by the PNV).

It seems cleaner/safter to just minimize the non-Subnet related responsibilites of a Subnet Validator such that it gets close to mirroring "just the hardware" and the on-chain fee as close to what the "PNV" could conceivably charge without losing money (goal of Step 1+2).

I think there should be additional work put into better distribution (Subnet creator + n others could upload a signed binary to some source and it would auto-download) and isolation (run in a Docker container with scoped access to disk or on a different machine entirely, still controlled by the validator) of Custom VM binaries (seemingly some of the inspiration for this feedback). However, I think that convo is best reserved for a separate thread.

## Can you use staking rewards for PAYG fees?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7387137_

> I'd like to recommend adding an option for validators to deduct PAYG fees from their base chain rewards, if it applies. That would remove (reduce) the taxation events incurred 1) when receiving rewards 2) when paying the subnet fee. It would also reduce the complexity involved in topping up several subnet-specific wallets.

This is an interesting suggestion that I hadn't yet considered.

Rewards, at least right now, are distributed "all or nothing" at the end of each validator's staking period. There is no notion of "accrued rewards" that can be used for this payment (it is unknown whether a reward will be distributed until the end). I guess some % of this reward could be "held" for this purpose but I'm not sure that would reduce taxable events (could be seen as a reward + transfer handled by the network).

If the reward mechanism is changed such that rewards are "partially locked in" as the staking period progresses, it would be possible to introduce something like this (complexity aside) where the required contribution is stripped from the reward before it is "distributed". Again, I'm no lawyer so not sure if this would actually achieve the goals you alluded to.

A more practical idea could be to allow these fees to be pulled from a Primary Network validator's stake. This would allow all idle $AVAX to be staked without needing to worry about manually reloading a "fee paying" account on some regular cadence (reminds me of a yield-earning checking account that you hook up to a credit card).

## Can you create Subnets for free if they use \$AVAX?

_Source: https://github.com/avalanche-foundation/ACPs/discussions/10#discussioncomment-7392935_

> Our main concern is how to make devs/ small companies launch subnets with the less ammount of entry barriers (monetary mainly) while also taking care about the value of the Avax token. What do you guys think about letting people deploy subnets without any staking requirements (0 avax instead of 2000 Avax), but only if they are using Avax as gas on their subnet.

While this is an interesting idea, there is no way currently to enforce (via the Primary Network) that a Subnet adheres to any set of rules/performs state transitions a certain way (in this case, using "AVAX" to pay fees). This is one of the trade-offs of the Primary Network not providing security to Subnets; the Primary Network can neither prove/disprove anything about any Subnet. Recall, this Subnet independence is also why data from Subnets doesn't need to be sent to the Primary Network and why Subnets don't need to pay fees as a % of their total usage.

Even if you were to require proofs of some sort to be posted to the Primary Network from Subnets to take advantage of this "promo", it would not be possible to verify those proofs weren't forged (post proofs saying you were using $AVAX when you didn't). To roll this out (and ensure the Primary Network wasn't "ripped off"), you'd probably need to randomly assign validators to this network to avoid collusion by some group of "cheaters". This is a pretty far departure from how Subnets work today.
