# Reliability and Performance

The first workload on a new cluster should be a test, not a training run that costs six figures.

Before asking application code to prove the platform, establish a baseline for each layer.

## Acceptance tests

Run these in order:

1. **Node inventory:** correct GPU, CPU, memory, disks, NICs, firmware, and driver.
2. **GPU diagnostics:** DCGM diagnostics and a sustained compute test.
3. **Topology:** expected NVLink, PCIe, NUMA, and GPU-to-NIC relationships.
4. **Single-node collectives:** prove NVLink and NVSwitch.
5. **Multi-node collectives:** prove the compute fabric while increasing scale.
6. **Storage:** test the real read, write, and metadata patterns.
7. **Soak:** run long enough to catch thermal, link, and intermittent faults.
8. **Failure drill:** remove a worker and prove drain, replacement, and restart.

Save the results with the cluster configuration. A baseline without context is hard to compare later.

## Observability by layer

| Layer | Useful signals |
|---|---|
| Workload | Step time, tokens/s, TTFT, TPOT, queue time, errors |
| Framework | Compute time, communication time, data-loader wait, checkpoint time |
| GPU | Tensor activity, memory activity, HBM use, power, temperature, XIDs |
| Interconnect | NVLink traffic and errors, collective bandwidth |
| Fabric | Link speed, errors, congestion, retransmits or pause signals |
| Storage | Throughput, IOPS, metadata rate, latency, cache hits |
| Scheduler | Pending time, fragmentation, quota, preemption, failed jobs |

Correlate by job, node, rank, and time. Cluster-wide averages erase the evidence you need.

## The fault-domain method

Ask where the problem follows:

- The application
- The container image
- The model or dataset
- One GPU
- One server
- One rack, leaf, or rail
- One storage target
- The whole cluster

Move one variable at a time. A job that stays slow when moved to healthy nodes points toward software or data. A slow node that affects every job points toward hardware or topology.

## Stragglers

Synchronous jobs run at the speed of the slowest rank.

Compare per-rank distributions, not only averages. Median absolute deviation is a simple robust way to find outliers when a few ranks are much slower than the rest.

Possible causes:

- Thermal or power throttling
- A degraded link
- Different NUMA placement
- Storage hot spots
- Background work
- Error recovery
- A bad GPU or NIC

Drain suspicious nodes and rerun before tuning the framework around faulty hardware.

## Failure handling

Define an automated response for:

- Uncorrectable memory errors
- GPU fallen off the bus
- NVLink or fabric faults
- Slow collective performance
- Repeated job crashes
- Storage exhaustion
- Scheduler control-plane failure

A common pattern is:

```text
detect → cordon or drain → preserve evidence → reset or replace
       → re-run acceptance test → return to service
```

Failing hardware should not return to the pool because a reboot made the alert disappear once.

## Performance regressions

When a workload is slower than its previous environment, make the comparison reproducible:

- Same model and code
- Same precision
- Same batch and sequence sizes
- Same container and library versions
- Same data path
- Same node count and topology

Then bisect:

1. Single GPU
2. Single server
3. Network collectives
4. Storage and data loader
5. Full workload

This turns "your cloud is slower" into a list of measurable differences.

## Capacity reliability

Auto-repair and autoscaling do not guarantee replacement capacity. A platform can detect a failed GPU correctly and still miss the recovery target because no spare node is available.

Plan:

- Hot or warm spare capacity
- Reservations
- Failure-domain spread where the workload permits it
- A lower-capacity degraded mode for inference
- Queue policy for training

Capacity is part of reliability.

## Things to watch for

**Alerting on GPU utilization alone.** It says little about why the workload is slow.

**Comparing benchmarks with different message sizes or topology.** Save the full command and environment.

**Running only a five-minute burn-in.** Intermittent link and thermal issues need time.

**Returning a node after a reboot without retesting it.** Recovery includes the acceptance suite.

## Go deeper

- [NVIDIA DCGM documentation](https://docs.nvidia.com/datacenter/dcgm/latest/)
- [NVIDIA XID error guide](https://docs.nvidia.com/deploy/xid-errors/)
- [PyTorch profiler](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html)
- [Workshop W12 — GPU Cluster Acceptance Test](../05-workshops/W12-gpu-cluster-acceptance.md)

---

[← Previous: Infrastructure as Code](28-infrastructure-as-code.md) · [Next: Secure AI Platforms →](../07-enterprise-architecture/30-secure-ai-platforms.md)
