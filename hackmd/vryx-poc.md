# Vryx PoC: Reproducible Devnet (Mark I)

_The First Y B Transactions on Vyrx_

A few months ago, I posted a [blog post](https://hackmd.io/@patrickogrady/rys8mdl5p) about Vryx, a fortified decoupled state machine replication construction. Vryx claimed to unblock
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

## Task
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

### Vilmo: Verifiable State Transition Application and Sync without Merklization

Most blockchains [merklize their state](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/) once per block. Merklization, which incurs a cost of `O(log(n))` for each state update/read (not including any additional cost to update the underlying database used to persist the merkle structure to disk), enables a proof to be constructed for arbitrary key/values in state. A nice byproduct of this approach is that it also allows all participants in a blockchain with merklized state to quickly verify that they executed all state transitions the same as everyone else (for the same set of key/values, everyone will generate the same merkle root). Merklization also allows for efficient, verifiable sync between nodes where a node can request a portion of the merkle trie at a given root with a proof that the provided state is correct. They can repeat this process until they have fetched all state (this can even be done [on-the-fly as roots are updated](https://github.com/ava-labs/avalanchego/blob/7975cb723fa17d909017db6578252642ba796a62/x/merkledb/README.md?plain=1#L194) with some clever tricks).

The primary downsides of merklizing state are this `O(log(n))` complexity for each operation and the need to manage a substantial number of intermediate (inner) nodes in the merkle trie (which contain no actual workload data but just hashes to other nodes). Most blockchains get around a `O(log(n))` complexity for each read by maintaining a [flat KV store for values](https://blog.ethereum.org/2021/05/18/eth-state-problems) that sits in front of the merkle trie, however, this approach doesn't solve for the cost to update state. Others, have opted to only store intermediate (inner) nodes in memory to avoid excessive disk writes or have opted to only update the merkle trie every `n` blocks to better amortize the cost of each update. Ava Labs has gone as far as building a [new database](https://www.avax.network/blog/introducing-firewood-a-next-generation-database-built-for-high-throughput-blockchains) to better avoid the overhead of managing a merkle trie on-disk (which has typically been done in a standard embedded database like LevelDB or RocksDB).

When I began testing the Vryx Proof-of-Concept, the HyperSDK merklized state at each block using the [MerkleDB](https://github.com/ava-labs/avalanchego/blob/7975cb723fa17d909017db6578252642ba796a62/x/merkledb). Very quickly, however, this became the bottleneck to increasing throughput. To find more headroom, I began to only generate a root every 60 seconds, however, this still caused instability (at 100k TPS with 10M accounts, this meant writing ~2.5M keys to disk in a single batch). Upon further review, I found the root cause of this was inserting tens of thousands of keys into [Pebble](https://github.com/cockroachdb/pebble), a RocksDB/LevelDB inspired key-value database, in a single batch every second. Pebble allows entries to be iterated over in-order, something that isn't needed to service a merkle trie. This got dramatically worse as I increased the number of keys in the database and compaction increased dramatically.

However, even with an optimal disk management (what Firewood is attempting to solve), the cost of updating a merkle trie will never be free. Maybe there is some other set of tradeoffs to explore?

Are we saying Merkilzation is the issue or existing KV databases (Pebble)? or both?

To maximize efficiency and still allow for verifiable state transition application and syncing, I ditched both merklization and Pebble for a new key-value database I created called Vilmo. Vilmo, like the HyperSDK, is opinionated and optimized for this particular use case. While it doesn't allow for proofs of arbitrary state to be generated, it does allow for...

Vilmo is a new key-value database that doesn't provide the ability to iterate over keys in-order that is optimized for massive write throughput. It is an append-only database model on-disk that can still maintain a set of deterministically checksummed log files that can be persisted in the chain and synced.

For teams that still want merklization, they can build on top of Vilmo.

 once per block or once per set of blocks.

Vilmo, unlike most state management mechanisms employed by blockchains, does not merklize state. It sacrifices the ability to prove arbitrary state at each block for performance. Even without merklization, provides
one useful property we can use to verify execution results between nodes and perform a verifible sync: deterministic checksums.

Started with MerkleDB but eventually bottlenecked on writes. After a number of optimizations to both PebbleDB and MerkleDB, I decided to try a different approach to drive more performance (and that it did).

Future work: could layer a merkle trie on top of Vilmo (which is just a KV store)

<TODO: include diagram of Vilmo (batch files with layers of content)>

Instead of merklizing state, we checksum append-only batches. This allows for verification of execution and for fast syncing but does not allow proof generation against state.

Write a series of logs (very similar to WAL approach) but we make compaction deterministic/tied to the block. This sets the stage for trivially charging rent on recycled keys.

Batches are written to append-only files that are eventually compacted when they have an overwhelming amount of useless data.

Open question: unequal log file sizes?

### Parallel Execution

To expedite transaction execution, non-conflicting transactions are processed in parallel. In this Devnet, this meant that ~80% of txs were
immediately executable.

## Learnings

### Push Certifified Chunks to Non-Validators

### Begin Verifying Chunk Authentication as Soon as Certified

### Refunds aren't compatible with chunk building

## What's Next?

## Acknowledgements

Thanks to Stephen for being a great sounding board throughout this PoC.
