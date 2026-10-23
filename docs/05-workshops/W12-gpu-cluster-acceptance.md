# Workshop W12 — GPU Cluster Acceptance Test

**Goal:** Build a repeatable acceptance suite for a GPU cluster before production workloads use it.

**Time:** Half a day for a small cluster; longer for soak testing

**Prerequisites:** Access to at least two GPU nodes for the full exercise. You can write the runbook and manifests without hardware.

---

## Safety first

Confirm that the cluster is intended for testing. Burn-in, network benchmarks, failure injection, and node drains can disrupt other users.

Record:

- Cluster owner
- Maintenance window
- Nodes in scope
- Expected GPU and network topology
- Commands that change state
- Rollback path

## Part 1: Inventory

Collect on every node:

- GPU model, count, UUID, memory, firmware, and driver
- CPU model, NUMA layout, and memory
- NIC model, speed, link state, and firmware
- Local disks
- OS and kernel
- CUDA and NCCL versions

Fail the inventory check when nodes in one pool differ unexpectedly.

## Part 2: GPU health

Run DCGM diagnostics at the level approved for the environment.

Capture:

- Pass or fail by GPU
- ECC and XID history
- Temperature and power
- Clock-throttle reasons

Do not return a failed GPU to service because a second run happened to pass. Preserve the evidence and follow the hardware process.

## Part 3: Topology

Run:

```bash
nvidia-smi topo -m
```

Confirm:

- Expected NVLink connections
- GPUs see the intended NICs
- No unexpected cross-socket paths
- Fabric Manager is healthy on NVSwitch systems

Save one known-good topology per server type.

## Part 4: Collective baselines

Run `all_reduce_perf`:

1. Inside one server
2. Across two servers
3. Across one topology block
4. Across the full test allocation

Use a broad message range. Save:

- Full command
- Environment variables
- Node list
- GPU and driver versions
- `algbw`, `busbw`, and errors

Compare with the platform baseline for the same shape. Investigate scaling cliffs rather than forcing one universal threshold.

## Part 5: Storage

Test the patterns the workload needs:

- Large sequential reads
- Parallel reads from many nodes
- Metadata operations if the dataset has many files
- Sharded checkpoint writes
- Restore reads

Calculate whether checkpoint bandwidth meets the recovery target.

## Part 6: Soak

Run a sustained mixed test long enough to see:

- Thermal throttling
- Link flaps
- Intermittent XIDs
- Memory errors
- Performance drift

Track per-node and per-rank results. A stable cluster should not hide a slow node inside an average.

## Part 7: Failure drill

With explicit approval for the test environment:

- Drain or remove one worker
- Confirm the job fails or requeues as designed
- Confirm replacement capacity appears
- Restore from the last checkpoint
- Re-run health checks before returning the node

Do not damage hardware or interrupt shared production capacity.

## Acceptance report

Produce:

1. Cluster shape and software versions
2. Test commands and data
3. Baseline results
4. Failed nodes or links
5. Recovery results
6. Open risks
7. Final accept, conditional accept, or reject decision

Put the suite in version control. Run it after hardware replacement, driver upgrades, fabric changes, and major cluster expansion.

## Go deeper

- [Chapter 31 — Reliability and Performance](../06-ai-infrastructure/31-reliability-and-performance.md)
- [NVIDIA DCGM](https://docs.nvidia.com/datacenter/dcgm/latest/)
- [nccl-tests](https://github.com/NVIDIA/nccl-tests)
