# Vryx PoC: Reproducible Devnet

<100k Sustained Image>

Don't believe me? You can reproduce it here with a single command: <TODO>

## Task
* 10M Accounts
* 1M Unique Accounts Active per 60s (~60k state changes per second)
-> 2.35M Unique Accounts Active per 60s (~X state changes per second)
* Zipf Distribution of Acitvity (s=1.0001 v=2.7)
* Simple Transfers using ED25519 Keys
* Historical blocks/chunks pruned after depth of 512

## Setup
* c7g.8xlarge (32 vCPU, 64GB RAM, 100GB io2 EBS with 1500 IOPS)
* 25 Validators (5 in each region)
* 5 API Nodes (1 in each region)

## How is this possible?
### Vryx
* Chunks
* Signatures
* Partitioning
* Filtered Chunks
* Block + Chunk Pruning

Not included:
* Bonds/Freezing

### AppendDB (TODO: new name)

Instead of merklizing state, we checksum state. This allows for verification of execution and for fast syncing but does not allow proof generation against state.

### Parallel Execution

## Learnings

### Push Certifified Chunks to Non-Validators

### Begin Verifying Chunk Authentication as Soon as Certified

### Refunds aren't compatible with chunk building

## What's Next?

