# GPU Systems

A GPU server is a small distributed system. It has several memory and communication paths, each with a different speed and purpose.

## Three limits

Start with three questions:

1. Does the workload fit in GPU memory?
2. Is it limited by math throughput or memory bandwidth?
3. How much data must move between GPUs?

Those answers matter more than the model number printed on the server.

| Resource | Plain meaning | Workloads that care most |
|---|---|---|
| HBM capacity | How much working data fits beside the GPU | Large models, long context, large batches |
| HBM bandwidth | How fast the GPU reads its own memory | Token generation, embedding lookup |
| Tensor compute | How fast the GPU performs matrix math | Training and prompt prefill |
| NVLink / NVSwitch | How fast GPUs communicate inside a server or rack-scale domain | Tensor parallelism, expert parallelism |
| PCIe | Path between GPU, CPU, NIC, and local devices | Host transfer, GPU-to-NIC affinity |

**HBM** is High Bandwidth Memory, mounted close to the GPU. Capacity decides whether a workload fits. Bandwidth often decides how fast a model decodes tokens.

## A current hardware map

Exact specifications change by form factor and vendor. Use this as a mental map, then check the current datasheet.

| GPU class | Memory | HBM bandwidth | Main reason to choose it |
|---|---:|---:|---|
| H100 SXM | 80 GB | about 3.35 TB/s | Mature training and inference platform |
| H200 SXM | 141 GB | about 4.8 TB/s | More memory and faster decode than H100 |
| B200 | about 180 GB | about 8 TB/s | More compute, FP4 support, faster NVLink |
| B300 | 288 GB | about 8 TB/s | Very large memory for reasoning and inference |

The useful comparison:

- H200 is mainly a memory and bandwidth upgrade over H100.
- Blackwell adds more low-precision compute and roughly doubles per-GPU NVLink bandwidth over Hopper.
- A newer GPU is not automatically cheaper for a workload. Availability, utilization, and software maturity matter.

## Inside an eight-GPU server

A common training server contains:

- Eight SXM GPUs
- NVSwitch, making the GPUs an all-to-all NVLink domain
- One or more CPUs and large host memory
- High-speed network adapters for cross-server traffic
- A separate front-end network for control, storage, and user traffic
- Local NVMe for caches and scratch data

The fast-to-slow order is roughly:

```text
HBM → NVLink → high-speed cluster network → storage
```

Good designs keep the most frequent communication on the fastest path. Tensor parallelism usually stays within NVLink. Data-parallel traffic can cross the cluster network.

## GPU-to-NIC affinity

Network adapters sit on particular PCIe paths. A GPU may reach one NIC through fewer switches or across a different CPU socket.

`nvidia-smi topo -m` shows the relationship. Poor placement can force traffic through a slower path and cut RDMA performance.

This is why a server with eight GPUs and eight fast NICs is not enough. The topology must be visible to NCCL and the scheduler.

## Precision is a systems choice

Lower precision reduces memory and can increase throughput:

- BF16 is a common training format.
- FP8 is used for training and inference on newer GPUs.
- INT4 or FP4 can make inference much smaller and faster.

Precision changes can affect quality or numerical stability. Treat them as software changes with evaluation gates, not harmless infrastructure tuning.

## Sharing a GPU

**MIG**, or Multi-Instance GPU, splits supported GPUs into isolated slices with dedicated memory and compute resources.

It works well for notebooks, small models, and development environments. It is usually the wrong tool for large distributed training.

Time-slicing shares a GPU without the same isolation or predictability. Use it only where noisy neighbors are acceptable.

## The software stack

From bottom to top:

1. Firmware and kernel driver
2. Fabric Manager on NVSwitch systems
3. CUDA runtime and libraries
4. NCCL, cuDNN, cuBLAS, and other accelerated libraries
5. Container toolkit
6. GPU and network operators or Slurm plugins
7. Frameworks such as PyTorch or JAX
8. Training or serving code

Version skew is one of the most common failures. The host driver, container CUDA version, communication library, and framework need a tested compatibility matrix.

## Health signals

DCGM, the Data Center GPU Manager, exposes utilization, memory activity, power, temperature, ECC errors, PCIe and NVLink traffic, and diagnostic tests.

Do not rely on one utilization percentage. A GPU can look busy while tensor cores are idle, or look idle while waiting on the network.

NVIDIA **XID** messages are driver-reported GPU errors. Some point to application faults. Others, such as uncorrectable memory, NVLink, or "fallen off the bus" errors, usually need a node drain and hardware investigation.

## Things to watch for

**Comparing sparse and dense FLOPS.** Datasheets may show both. Most planning should use dense numbers unless the workload actually uses structured sparsity.

**Mixing bits and bytes.** A 400 Gb/s network link is 50 GB/s before overhead.

**Treating a CUDA error as an application bug first.** Check driver, Fabric Manager, device exposure, and version compatibility.

**Using average GPU utilization as the health metric.** Look at tensor activity, memory activity, throttling, and per-rank behavior.

## Go deeper

- [NVIDIA data center GPU products](https://www.nvidia.com/en-us/data-center/)
- [NVIDIA DCGM documentation](https://docs.nvidia.com/datacenter/dcgm/latest/)
- [NVIDIA XID error guide](https://docs.nvidia.com/deploy/xid-errors/)
- [NVIDIA GPU Operator documentation](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#gpu-systems)

---

[← Previous: Workload Sizing](../05-ai-workloads/23-workload-sizing.md) · [Next: Cluster Networking →](25-cluster-networking.md)
