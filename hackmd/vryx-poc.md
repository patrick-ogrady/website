# Vryx PoC: Reproducible Devnet
_The First Y B Transactions on Vyrx_

> THIS IS A POC AND NOT PRODUCTION-READY CODE. IF YOU RUN INTO ANY BUGS, PLEASE POST THEM ON THE REPO!

> This is not just a consensus implementation but an integration into the HyperSDK.

<100k Sustained Image>

Don't believe me? You can reproduce it here with a single command (just make sure you are signed into AWS): <TODO>

Don't want to do it? You can watch me do it on YouTube <TODO: link to deploy demo video>

Sustained for X hrs (could keep going, no increase in disk)

> Is this maxxed out?

no. this viability exercise was just undertaken to test out our assumptions and ensure
we get hit our throughput targets in a multi-regional setting.

<TODO: include images of CPU/RAM/DISK>

## Task
* 10M Accounts
* ~2.5M Unique Accounts Active per 60s (~100k state changes per second)
* Simple Transfers using ED25519 Keys
  * Zipf Distribution of Acitvity (s=1.0001 v=2.7)
* Historical blocks/chunks pruned after depth of 512

* Finalized Transaction Data = ~20MB/s
  * bandwidth used by each validator is X MB/s inbound and X MB/s outbound (bandwidth dominated by chunk distribution which is symmetric)
    * TODO: amount is less than this to the node because of compression
  * at no point does any validator exceed Y MB/s of inbound/outbound bandwidth (no hotspots)
  * API nodes are ~Z MB/s (get subset of txs and fetch chunks from validators)
* TTC Tx Attested (once tx is on validator) = 230ms
* Time-to-Chunk Attestation (Issuer -> API -> Validator -> Chunk Produced (every 333ms) -> Chunk Attested) = 125ms + 125ms + 330ms (worst) + 230ms = ~710ms
  * 0% expiry/failure rate for transactions, chunks, and blocks
* E2E Time-to-Finality (issuance to ordered on-chain but before executed and notified) = 2.2-2.7s (1s block time)
* E2E Time-to-Execution (notifyied issuer) (TTF + TTE) = 3-3.5s

## Setup
* c7g.8xlarge (32 vCPU, 64GB RAM, 100GB io2 EBS with 2000 IOPS)
* 5 Regions (us-west-2,us-east-1,ap-south-1,ap-northeast-1,eu-west-1)
* 25 Equal Weight Validators (5 in each region)
* 5 API Nodes (1 in each region)
* 1 Tx Issuer in eu-west (sends txs randomly to API nodes, doesn't use known validator partitions)

> Can we add more nodes?

Yes. Each additional node requires additional replication of all block and chunk material.

If you run the repro, post how many nodes you used! We plan to scale this up over the coming devnets (just uses Snowman Consensus behind the scenes, which has already hit ~1.8k validators on Mainnet)!

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

### Would you ever run this in production? Constraining State Growth with Multi-Dimensional Fee Markets

### Pruning chunks/blocks from node as soon as not needed anymore

### Open Question: MerkleDB or Vilmo

Started with MerkleDB but eventually bottlenecked on writes. After a number of optimizations to both PebbleDB and MerkleDB, I decided to try a different approach to drive more performance (and that it did).

Instead of merklizing state, we checksum append-only batches. This allows for verification of execution and for fast syncing but does not allow proof generation against state.

Write a series of logs (very similar to WAL approach) but we make compaction deterministic/tied to the block. This sets the stage for trivially charging rent on recycled keys.

Batches are written to append-only files that are eventually compacted when they have an overwhelming amount of useless data.

### Parallel Execution

## Learnings

### Push Certifified Chunks to Non-Validators

### Begin Verifying Chunk Authentication as Soon as Certified

### Refunds aren't compatible with chunk building

## What's Next?

