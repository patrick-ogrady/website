# Vryx PoC

<100k Sustained Image>

## Task
* 10M Accounts
* 1M Unique Accounts Active per 60s (~60k state changes per second)
* Zipf Distribution of Acitvity (s=X v=Y)
* Simple Transfers using ED25519 Keys

## Setup
* c7g.8xlarge (32 vCPU, 64GB RAM, 100GB io2 EBS with 1500 IOPS)
* 25 Validators (5 in each region)
* 5 API Nodes (1 in each region)

