# Storage and the Data Path

An idle GPU is usually waiting for something. Sometimes that thing is the network between GPUs. Often it is the data path.

## Four different storage jobs

| Job | Access pattern | Main concern |
|---|---|---|
| Dataset loading | Many readers, often small files or shuffled records | Metadata, read bandwidth, CPU decode |
| Checkpoint writing | Many ranks write together | Aggregate write bandwidth and pause time |
| Checkpoint restore | Burst reads after a failure | Recovery time |
| Model loading | Large sequential reads during scale-out | Cold-start time |

One storage product rarely serves all four well.

## A practical tiered design

**Object storage** is the durable system of record for datasets, checkpoints, and model artifacts. It scales well and is cheap, but it is not a POSIX filesystem.

**Shared or parallel filesystems** give many workers a common namespace and high aggregate throughput. They are useful for active datasets, shared environments, and checkpoints.

**Local NVMe** is fast and close to the workload. It is good for caches, staged shards, temporary checkpoints, and model weights. It is usually ephemeral.

**Block storage** is a good fit for boot volumes, controllers, and databases. It is rarely the main data path for a large training job.

## Small files are expensive

Millions of small files can make a fast filesystem look slow because metadata operations dominate.

Common fixes:

- Pack samples into larger shards
- Use formats designed for streaming, such as WebDataset
- Prefetch and cache locally
- Increase data-loader workers carefully
- Move decode or augmentation onto the GPU where it helps

Measure before tuning. Too many workers can create CPU, memory, or metadata pressure.

## Checkpoint bandwidth

Size checkpoint storage from the acceptable pause:

```text
required write bandwidth =
  checkpoint size / acceptable write time
```

A 1.1 TB checkpoint written in 60 seconds needs about 18 GB/s of aggregate write bandwidth, before overhead.

Use sharded checkpoints so each rank writes its own part. Where supported, write asynchronously and copy durable versions to object storage in the background.

Keep a small number of recent checkpoints on fast storage. Move older ones to cheaper durable storage with a lifecycle policy.

## Finding data starvation

Start with a synthetic-data run.

If the GPUs become much faster with generated data, the model and compute path are probably fine. Then inspect:

- Data-loader wait time
- CPU decode or augmentation
- Filesystem throughput and metadata rate
- Host-to-device copies
- Cache hit rate
- Per-worker imbalance

Low GPU use plus healthy NCCL is a strong reason to look here.

## Model loading and inference scale-out

Autoscaling a model service may require every new replica to load tens or hundreds of gigabytes.

Reduce cold starts with:

- Weights cached on local NVMe
- Pre-pulled container images
- A warm pool
- A weight format the engine loads efficiently
- A shared cache that avoids every node downloading at once
- Admission control while capacity becomes ready

Do not let a scale-out event become an object-storage denial-of-service attack from your own nodes.

## Data gravity

Moving petabytes takes time and may create egress charges. A technically good GPU platform can still be a poor choice if all data and dependent services remain elsewhere.

Plan a migration in stages:

1. Inventory data, update rate, and dependencies.
2. Move a representative subset for the benchmark.
3. Measure transfer and transformation time.
4. Define the source of truth during the transition.
5. Decide whether the final architecture is migrated or hybrid.

The network path to the data belongs in the platform design.

## Things to watch for

**Benchmarking sequential throughput only.** Real loaders may be limited by metadata or random access.

**Writing one checkpoint file from one process.** It wastes cluster parallelism and makes recovery fragile.

**Putting durable data only on local NVMe.** Fast is not the same as safe.

**Blaming the GPU fabric for low utilization.** Replace the dataset with synthetic input before guessing.

## Go deeper

- [PyTorch Distributed Checkpoint](https://pytorch.org/docs/stable/distributed.checkpoint.html)
- [WebDataset](https://github.com/webdataset/webdataset)
- [NVIDIA GPUDirect Storage](https://docs.nvidia.com/gpudirect-storage/)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#storage)

---

[← Previous: Cluster Networking](25-cluster-networking.md) · [Next: Kubernetes and Slurm →](27-kubernetes-and-slurm.md)
