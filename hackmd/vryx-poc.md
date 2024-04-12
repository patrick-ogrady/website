# Vryx PoC: Reproducible Devnet
_The First 10B Transactions on Vyrx_

> THIS IS A POC AND NOT PRODUCTION-READY CODE. IF YOU RUN INTO ANY BUGS, PLEASE POST THEM ON THE REPO!

> This is not just a consensus implementation but an integration into the HyperSDK.

<100k Sustained Image>

<TODO: link to deploy video>

Don't believe me? You can reproduce it here with a single command: <TODO>

Sustained for X hrs (could keep going, no increase in disk)

> Is this maxxed out?

no. this viability exercise was just undertaken to test out our assumptions and ensure
we get hit our throughput targets in a multi-regional setting.

<TODO: include images of CPU/RAM/DISK>

## Task
* 10M Accounts
* ~2.5M Unique Accounts Active per 60s (~100k state changes per second)
* Zipf Distribution of Acitvity (s=1.0001 v=2.7)
* Simple Transfers using ED25519 Keys
* Historical blocks/chunks pruned after depth of 512

* Finalized Transaction Data = ~20MB/s
  * bandwidth used by each node is 20MB/s inbound and 20MB/s outbound (bandwidth dominated by chunk distribution which is symmetric)
    * TODO: amount is less than this to the node because of compression
  * at no point does any validator exceed 21MB/s of inbound/outbound bandwidth (no hotspots)
* TTC Tx Attested (once tx is on validator) = 230ms
* Time-to-Chunk Attestation (Issuer -> API -> Validator -> Chunk Attested) = 125ms + 125ms + 230ms = ~480ms
* Time-to-Finality = 2.4-2.9s (1s block time)
* Time-to-Execution (TTF + TTE) = 3-3.5s
* 0% expiry/failure rate for transactions, chunks, and blocks

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

### Would you ever run this in production? Constraining State Growth with Multi-Dimensional Fee Markets

### Pruning chunks/blocks from node as soon as not needed anymore

### Open Question: MerkleDB or Vilmo 

Instead of merklizing state, we checksum state. This allows for verification of execution and for fast syncing but does not allow proof generation against state.

### Parallel Execution

## Learnings

### Push Certifified Chunks to Non-Validators

### Begin Verifying Chunk Authentication as Soon as Certified

### Refunds aren't compatible with chunk building

## What's Next?

