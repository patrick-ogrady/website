# Vryx PoC: Reproducible Devnet

> THIS IS A POC AND NOT PRODUCTION-READY CODE. IF YOU RUN INTO ANY BUGS, PLEASE POST THEM ON THE REPO!

<100k Sustained Image>

Don't believe me? You can reproduce it here with a single command: <TODO>

Sustained for X hrs (could keep going, no increase in disk)

## Task
* 10M Accounts
* ~2.5M Unique Accounts Active per 60s (~100k state changes per second)
* Zipf Distribution of Acitvity (s=1.0001 v=2.7)
* Simple Transfers using ED25519 Keys
* Historical blocks/chunks pruned after depth of 512

## Setup
* c7g.8xlarge (32 vCPU, 64GB RAM, 100GB io2 EBS with 2000 IOPS)
* 5 Regions (us-west-2,us-east-1,ap-south-1,ap-northeast-1,eu-west-1)
* 25 Equal Weight Validators (5 in each region)
* 5 API Nodes (1 in each region)

> Can we add more nodes?

Yes. Each additional node requires additional replication of all block and chunk material.

If you run the repro, post how many nodes you used!

> Can we add more regions?

Yes.

## How is this possible?
### Vryx
* Chunks
* Signatures
* Partitioning
* Filtered Chunks
* Block + Chunk Pruning

Not included:
* Bonds/Freezing

### Open Question: MerkleDB or AppendDB (TODO: new name)

Instead of merklizing state, we checksum state. This allows for verification of execution and for fast syncing but does not allow proof generation against state.

### Parallel Execution

## Learnings

### Push Certifified Chunks to Non-Validators

### Begin Verifying Chunk Authentication as Soon as Certified

### Refunds aren't compatible with chunk building

## What's Next?

