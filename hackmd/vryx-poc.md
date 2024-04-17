# Processing 5 Billion Micropayments (at 100k TPS) with Vryx and Vilmo

Over the last few days, the first [Vryx-enabled](https://hackmd.io/@patrickogrady/rys8mdl5p) HyperSDK devnet processed 5+ billion micropayments at a sustained rate of 100,000 transactions per second (TPS). This test involved 10,000,000 active accounts (2,500,000 active every 60 seconds and 95,000 active every second) and 50 equal weight validators (32 vCPU, 64 GB RAM, 100 GB io2 SSD) distributed over 5 regions (us-west-2, us-east-1, ap-south-1, ap-northeast-1, eu-west-1).

![Transactions Per Second](https://patrickogrady.xyz/images/vryx-poc/transactions-rate.png)

You can view the code (Vryx Proof-of-Concept HyperSDK integration) used to run this devnet [here](https://github.com/ava-labs/hypersdk/pull/711). You can reproduce these results in your own AWS account by [running a single command](https://github.com/ava-labs/hypersdk/blob/dadbb8248d6b499eb38b14d6014a1e42a012e4d1/examples/morpheusvm/scripts/deploy.devnet.sh).

![Cummulative Transactions](https://patrickogrady.xyz/images/vryx-poc/transactions-cummulative.png)

## Task: Process 5 Billion Micropayments


Today, I couldn't be more thrilled to share the results of the Proof-of-Concept integration of Vryx with the HyperSDK.

In January, I released [Vryx](https://hackmd.io/@patrickogrady/rys8mdl5p), a fortified decoupled state machine replication construction.

 that would be integrated with the [Avalanche HyperSDK](https://github.com/ava-labs/hypersdk), to push its throughput to our initial goal of 100,000 transactions per second (TPS).


 Vryx claimed to unblock
a large increase in throughput for the HyperSDK. This blog post is a write-up of the PoC of that work.

 documentation about [Vryx](https://hackmd.io/@patrickogrady/rys8mdl5p), a fortified decoupled state machine replication construction and some detailed ideas of how that would be applied to the HyperSDK.

> THIS IS A POC AND NOT PRODUCTION-READY CODE. IF YOU RUN INTO ANY BUGS, PLEASE POST THEM ON THE REPO!

> This is not just a consensus implementation but an integration into the HyperSDK.

<TODO: 100k Sustained Image>

Don't believe me? You can reproduce it here with a single command (just make sure you are signed into AWS): <TODO>

Don't want to do it? You can watch me do it on YouTube <TODO: link to deploy demo video>

Sustained for X hrs (could keep going, no increase in disk)

> Is this maxxed out?

no. this viability exercise was just undertaken to test out our assumptions and ensure
we get hit our throughput targets in a multi-regional setting. As the system is productionized
over the coming months we will attempt to max it out (both with participants and throughput).

<TODO: include images of CPU/RAM/DISK>

## Task: Simulate Micropayment Workload
* 10M Accounts
* ~2.5M Unique Accounts Active per 60s (~100k state changes per second)
* Simple Transfers using ED25519 Keys
  * Zipf Distribution of Acitvity (s=1.0001 v=2.7)
* Historical blocks/chunks pruned after depth of 512

network latency -> including handling time
-> should divide by 2 for one-way (fix time to chunk attestation)

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
* c7g.8xlarge (32 vCPU, 64GB RAM, 100GB io2 EBS with 2500 IOPS)
* 5 Regions (us-west-2,us-east-1,ap-south-1,ap-northeast-1,eu-west-1)
* 50 Equal Weight Validators (10 in each region)
* 5 API Nodes (1 in each region)
* 1 Tx Issuer (c7gn.8xlarge, eu-west-1, sends txs randomly to API nodes, doesn't use known validator partitions)

> Can we add more nodes?

Yes. Each additional node requires additional replication of all block and chunk material.

If you run the repro, post how many nodes you used! We plan to scale this up over the coming devnets (just uses Snowman Consensus behind the scenes, which has already hit ~1.8k validators on Mainnet)!

> Can we add more regions?

Yes.

## How is this possible?
### Vryx
* Chunk-Based Transaction Dissemination
  * BLS Multi-Signatures
* Address Partitioning
* Filtered Chunks
* Block + Chunk Pruning

Not included:
* Bonds/Freezing

### Would you ever run this in production? Constraining State Growth with Multi-Dimensional Fee Markets

Unlike with a single dimensional fee where increasing capacity opens the door to using any available resource to the max usage,
the HyperSDK uses Multi-Dimensional Fees to individually constrain each resource. This allows someone, for example, to restrict
the amount of new state that can be allocated each second to a value much less than the state read/updated per second (which is often
less of the bottleneck and/or incurs little to no long-term risk).

### Pruning chunks/blocks from node as soon as not needed anymore

Validators running HyperSDK-Based chains are not expected to indefinitely persist chain data. Rather, they are just supposed
to store enough data for other Validators to sync to the network. As a result, Validators don't require much disk space and can run
at "steady state" indefinitely because they clean up after themselves.

### Vilmo: Verifiable State Transitions and Sync without Merklization

When designing the state storage layer for a blockchain, there are three primary capabilities that are typically considered:

1) proving the value of arbitrary keys in state
2) enforcing that state transitions are applied consistently amongst all parties
3) fetching the current state efficiently and verifiably from existing participants (to avoid re-execution of all state transitions)

When all these capabilities are required, most blockchains opt to [merklize their state](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/) either once per block or once every few blocks (batched state updates can be more efficient). Once state is merklized, it can be efficiently represented by a single hash, the merkle root. If any values are modified, this root will change (i.e. two states will only have the same root if they have the same state). With this root, we can (1) generate a proof that any key-value (or range of key-values) is described by a root by providing a path of hashes from the root to the key-value in question. We can (2) enforce state transitions are applied consistently by checking whether this root (typically just 32 bytes) is the same as the root others have computed. We can (3) fetch the current state from existing participants by requesting a range of key-values from a given root and verifying the proof provided (this can even be done [on-the-fly as roots are updated](https://github.com/ava-labs/avalanchego/blob/7975cb723fa17d909017db6578252642ba796a62/x/merkledb/README.md?plain=1#L194) with some clever tricks). You can read more about the interesting properties of Merkle Tries [here](https://www.avax.network/blog/from-the-labs-handling-blockchain-state).

Merklization, however, is not "free". Each state update/read (not including any additional cost to update/maintain the underlying database that persists the merkle structure to disk) incurs a cost of `O(log(n))` and intermediate (inner) nodes must be maintained although they contain no workload data (just hashes of other nodes, which may be more intermediate nodes in a large trie). Most blockchains get around the `O(log(n))` complexity for each read by maintaining a [flat, key-value store](https://blog.ethereum.org/2021/05/18/eth-state-problems) that sits in front of the Merkle Trie, however, this approach requires storing a copy of all values and does nothing to reduce the cost of updating the Merkle Trie (in fact, it increases the cost because the value must be stored in both places). Others, have opted to store intermediate (inner) nodes in memory to avoid excessive disk writes or have opted to only update the merkle trie every `n` blocks to better amortize the cost of each update. Ava Labs has gone as far as building a [new database (Firewood)](https://www.avax.network/blog/introducing-firewood-a-next-generation-database-built-for-high-throughput-blockchains) to avoid the overhead of the underlying database managing the Merkle Trie on-disk (which has typically been done in an embedded database like LevelDB or RocksDB that isn't optimized for this workload).

Vilmo, the embedded key-value database that manages state on the HyperSDK, is an answer to the question: what if we don't require all of these properties? Specifically, what if we (1) don't need to prove the value of arbitrary keys in state but still want to (2) enforce state transitions are applied consistently and that (3) new nodes can efficiently fetch the current state from existing participants?

Vilmo optimizes for large (100k+ key-values) batch writes by leveraging a collection of rotating append-only log files. The deterministic checksum of the changes written in a single batch to a log file can then be used to compare the result of execution between multiple parties.  When most of the data previously written to a log file is no longer "alive", the log file is compacted (rewritten) to only include "alive" data. When a batch is appended to an existing log file, the checksum of the last batch is added to the checksum calculation of the new batch so that the entire log file can be verified rather than just the latest batch. This allows for other parties to sync the last `n` log files from the chain to fully sync the latest state (by applying the modifications in order). Vilmo assumes that the operator keeps an index of all alive keys and the location of their values in-memory. Each log file is mmap-ed for fast access. You can review a preliminary implementation of Vilmo [here](https://github.com/ava-labs/hypersdk/tree/dadbb8248d6b499eb38b14d6014a1e42a012e4d1/vilmo). If you are familiar with WALs, you can think of Vilmo as a series of rotating WALs that are updated in a specific way.

<TODO: include diagram of Vilmo (batch files with layers of content)>

Vilmo compaction (when required) occurs during block execution and is synchronized across all nodes. The frequency of this compaction is tuneable (i.e. how much "useless" data can live in a log before cleanup), however, the timing of this compaction (during block execution) is not. This approach allows for a forthcoming implementation of state expiry and/or state rent to be applied during compaction (charging a fee to each key that is preserved during a rewrite). This fee would likely increase the larger the log file is to deter an attacker from purposely increasing the size of a single log file to increase the time compaction will take (Vilmo works best when log files are of uniform size). Exposing state compaction to the HyperSDK allows it to better charge for resource usage that is currently not accounted for in most blockchains (i.e. the cost of maintaining state on-disk).

### Parallel Execution

To expedite transaction execution, non-conflicting transactions are processed in parallel. In this Devnet, this meant that ~80% of txs were
immediately executable.

## Learnings

### Push Certifified Chunks to Non-Validators

### Begin Verifying Chunk Authentication as Soon as Certified

### Refunds aren't compatible with chunk building

## What's Next?

Productionize the PoC.

## Acknowledgements

Thanks to Stephen for being a great sounding board throughout this PoC.
