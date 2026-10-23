# Cluster Networking

Distributed training is synchronized. Each step waits for communication to finish before the next step can move on. One weak link can slow every GPU in the job.

## Two networks, two jobs

GPU platforms usually separate:

- The **compute fabric**, carrying GPU-to-GPU communication between servers
- The **front-end network**, carrying control traffic, storage, APIs, and user access

Combining them is possible at smaller scale. At large scale, storage bursts and user traffic should not compete with training collectives.

## RDMA and GPUDirect

**RDMA**, Remote Direct Memory Access, lets one machine move data directly to another machine's registered memory without the usual kernel networking path.

**GPUDirect RDMA** lets the network adapter move data directly to or from GPU memory. It avoids a copy through host memory and lowers CPU overhead.

The path only works when drivers, container device access, PCIe topology, and network configuration agree. A silent fallback to TCP can make a job work slowly instead of failing clearly.

## InfiniBand and RoCE

| | InfiniBand | RoCEv2 |
|---|---|---|
| Network | Purpose-built HPC fabric | RDMA over Ethernet and UDP/IP |
| Loss handling | Credit-based and lossless by design | Requires careful congestion and loss configuration |
| Operations | Subnet manager and InfiniBand tools | Familiar Ethernet and IP tools |
| Common use | Large synchronous training | Training and inference where Ethernet is preferred |

RoCE needs a well-designed congestion strategy. Priority Flow Control, Explicit Congestion Notification, and DCQCN are common pieces. Bad tuning can create head-of-line blocking or congestion spreading.

Do not turn this into a brand argument. Compare the workload's collective performance, isolation, operational skill, and cost.

## NCCL collectives

NCCL chooses algorithms and transports for collective operations:

- Data parallelism: all-reduce
- FSDP: all-gather and reduce-scatter
- Tensor parallelism: frequent all-reduce
- Mixture of experts: all-to-all
- Pipeline parallelism: point-to-point sends

Different patterns stress the fabric differently. A network that handles large all-reduce messages well may still struggle with latency-sensitive or all-to-all traffic.

## Reading `nccl-tests`

`nccl-tests` is the standard first check for GPU collective performance.

For all-reduce it reports:

- **algbw:** data size divided by time
- **busbw:** a normalized estimate of fabric use

Use the provider or platform team's baseline for the same GPU, node count, topology, and message range. There is no honest universal "good bus bandwidth" number.

Test in steps:

1. One server
2. Two servers
3. One rack or leaf group
4. The target job size

A sharp drop at one boundary points toward topology, oversubscription, or a weak node.

## Topology

A **Clos** or fat-tree network uses leaf and spine switches. A non-blocking design has enough uplink capacity to carry all downlink traffic at once.

Oversubscription is a ratio between edge capacity and uplink capacity. It saves money and may be fine for inference or small jobs. Large synchronized training is much less forgiving.

Rail-optimized designs connect the same NIC position from many servers into the same network rail. This matches common collective patterns and gives NCCL predictable paths.

Schedulers should understand placement boundaries such as block, rack, leaf, and fabric. A healthy job can become slower when spread across a wider topology.

## Isolation

The compute fabric is still a multi-tenant security boundary.

InfiniBand uses partition keys, or P-Keys, to separate groups on a shared fabric. Ethernet designs use controls such as VLANs, VRFs, ACLs, and host-level policy. Stronger requirements may need dedicated hardware or clusters.

Test isolation. A configuration that looks separate in an API is not enough.

## A structured debug path

1. Scope the problem: one node, one rail, one rack, or the whole cluster.
2. Check link state, speed, width, counters, and errors.
3. Run point-to-point bandwidth and latency tests.
4. Check GPU-to-NIC and NUMA affinity.
5. Confirm NCCL selected RDMA rather than sockets.
6. Run collectives while increasing node count.
7. Bisect slow nodes and drain the outliers.
8. Only then tune NCCL algorithms or application overlap.

Useful tools include `ibstat`, `ibv_devinfo`, `perfquery`, perftest, `nvidia-smi topo -m`, and `NCCL_DEBUG=INFO`.

## Things to watch for

**A link negotiated below its expected speed.** The port can be Active and still be slow.

**RDMA devices missing inside the container.** NCCL may fall back to sockets.

**MTU mismatch.** It often appears after encapsulation or mixed network paths.

**A single thermally throttled GPU.** It looks like a network straggler because the whole collective waits for it.

**Tuning environment variables before finding the fault.** Overrides can hide the real topology problem.

## Go deeper

- [NCCL documentation](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/)
- [nccl-tests performance notes](https://github.com/NVIDIA/nccl-tests/blob/master/doc/PERFORMANCE.md)
- [GPUDirect RDMA documentation](https://docs.nvidia.com/cuda/gpudirect-rdma/)
- [NVIDIA DGX SuperPOD reference architecture](https://docs.nvidia.com/dgx-superpod/)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#cluster-networking)

---

[← Previous: GPU Systems](26-gpu-systems.md) · [Next: Storage and the Data Path →](28-storage-and-data.md)
