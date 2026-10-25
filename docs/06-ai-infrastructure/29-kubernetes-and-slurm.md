# Kubernetes and Slurm

GPU scheduling has a harder constraint than ordinary service scheduling: many workloads need several GPUs or nodes at the same time.

## What the managed service does not remove

A managed Kubernetes service may operate the control plane. The platform team still owns how workloads reach and use it.

| Layer | What you need to understand |
|---|---|
| Control plane | API access, versions, audit logs, availability, and upgrade policy |
| Node pools | Images, labels, taints, autoscaling, repair, capacity, and replacement |
| CNI | Pod addresses, routing, network policy, service traffic, and MTU |
| CSI | Storage classes, volume provisioning, attachment, mounting, and recovery |
| Scheduling | Requests, limits, affinity, topology, quotas, priority, and gang admission |
| Device layer | GPU drivers, device plugins or DRA, RDMA devices, and health |
| Operations | Monitoring, upgrades, backups, incidents, and cost ownership |

**CNI**, the Container Network Interface, connects pods to the network. A pod can be Running while DNS, routes, network policy, or MTU still prevents the application from working.

**CSI**, the Container Storage Interface, connects workloads to storage systems. A claim moves through storage-class selection, provisioning, attachment, and mounting. An error at any stage can leave a pod Pending.

## How Kubernetes exposes a GPU

The NVIDIA GPU Operator can manage:

- GPU drivers
- Container runtime integration
- Device plugins or Dynamic Resource Allocation drivers
- GPU feature labels
- MIG configuration
- DCGM metrics and validation

The network side may need a separate operator or device plugin for RDMA adapters.

A pod that requests a GPU still depends on the host driver, container toolkit, scheduler, device allocation, and security policy all lining up.

## Gang scheduling

A 64-GPU job cannot make progress with 32 GPUs.

The default scheduler may start some workers while the rest remain pending. Those partial workers hold resources and can deadlock the queue.

**Gang scheduling** admits all workers together or none of them.

Kueue handles admission, quotas, borrowing, priority, and topology-aware placement for Kubernetes batch workloads. Volcano provides queues, fair sharing, and PodGroups. Training operators then create the actual distributed jobs.

## Dynamic Resource Allocation

Dynamic Resource Allocation, or DRA, gives Kubernetes a richer model than a single integer such as `nvidia.com/gpu: 1`.

It can express device attributes and allocation claims. This matters for:

- GPU model and memory
- MIG profiles
- GPU and network-device co-allocation
- Topology
- Devices that need preparation before use

DRA is still an area where Kubernetes versions, drivers, and vendor implementations matter. Test the exact stack you plan to run.

## Slurm

Slurm is a batch scheduler built for HPC.

Core concepts:

- Partitions are queues of nodes.
- GRES describes generic resources such as GPUs.
- QOS and fair-share control priority.
- Reservations protect capacity.
- Job arrays repeat similar work.
- Topology plugins place jobs near each other.

Researchers often prefer Slurm because `sbatch`, `srun`, accounting, gang start, and exclusive-node jobs are established workflows.

Platform teams often prefer Kubernetes because it uses declarative APIs, containers, GitOps, autoscaling, and the same control plane as serving and data services.

## Slurm on Kubernetes

Projects such as Soperator run Slurm components and worker nodes through Kubernetes.

This can give researchers a Slurm interface while the platform team keeps Kubernetes for provisioning, health automation, and shared capacity. It also adds complexity: two control planes now need clear ownership and failure handling.

Use it when both user experiences matter. Do not add it just to collect more platform logos.

## Choosing the platform

| Workload and team | Starting point |
|---|---|
| Large synchronous training with an HPC-native research team | Slurm |
| Online inference and cloud-native services | Kubernetes |
| Mixed training, inference, and data pipelines | Kubernetes, possibly with a Slurm layer |
| Distributed Python data processing or RL | Kubernetes with Ray can fit well |
| Small team with one workload | Use the managed path with the least operational surface |

The team's operating model matters as much as the workload.

## Multi-tenancy

Useful controls include:

- Namespaces, RBAC, and service accounts
- Resource quotas and queue quotas
- Separate node pools, taints, and tolerations
- Network policies
- Dedicated fabrics or fabric partitions
- MIG for suitable small workloads
- Separate clusters for hard regulatory or noisy-neighbor boundaries

Fair sharing is not isolation. Define both.

## Common failures

**Pod is Pending**

- No GPU capacity
- Taint is not tolerated
- Node selector or affinity cannot match
- Queue quota is exhausted
- Persistent volume is not ready
- Device plugin or DRA driver is unhealthy

**Pod runs but CUDA sees no device**

- Driver and container are incompatible
- Container runtime integration is missing
- Device allocation failed
- Security policy blocked device access

**Multi-node job hangs**

- Rendezvous address or port is wrong
- Network policy blocks workers
- NCCL picked the wrong interface
- RDMA device is missing inside the pod
- Images contain different NCCL versions

## A troubleshooting order

When a workload fails, move down the stack:

1. Read the workload status and Kubernetes events.
2. Check admission, queue quota, requests, affinity, taints, and capacity.
3. Confirm the selected node reports the expected allocatable devices.
4. Check the device plugin or DRA driver and the host GPU driver.
5. Check CNI routes, DNS, network policy, service endpoints, and MTU.
6. Check CSI provisioning, attachment, mount events, and storage permissions.
7. Check the container image, runtime libraries, and application logs.
8. For multi-node training, verify rendezvous and run a small collective test.

Do not begin with application tuning when Kubernetes has not delivered the requested network, storage, or device.

## Day-two operations

Treat upgrades as a compatibility change across Kubernetes, the node image, GPU driver, container toolkit, GPU operator, CNI, CSI, and workload images.

A safe sequence is:

1. Test the version set on a small pool.
2. Upgrade the control plane within its supported skew.
3. Upgrade CPU system pools.
4. Cordon and drain a small GPU pool.
5. Run device, network, storage, and workload acceptance tests.
6. Roll through the remaining pools with a rollback point.

Draining a stateless service and interrupting a multi-hour training job are different operations. Coordinate drains with checkpoints, disruption budgets, queue state, and spare capacity.

## Things to watch for

**Autoscaling without capacity planning.** A pending pod does not create a scarce GPU in the region.

**Scheduling by GPU count only.** Model, memory, fabric, and topology can matter.

**Running system services on expensive GPU nodes.** Keep general platform workloads on CPU pools.

**Giving every team its own scheduler.** Shared capacity becomes hard to govern.

## Go deeper

- [Kubernetes DRA documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)
- [Kueue documentation](https://kueue.sigs.k8s.io/)
- [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/)
- [Slurm documentation](https://slurm.schedmd.com/documentation.html)
- [Soperator](https://github.com/nebius/soperator)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#kubernetes-and-slurm)
- [Workshop W16 — Kubernetes Operations for AI](../05-workshops/W16-kubernetes-ai-operations.md)

---

[← Previous: Storage and the Data Path](28-storage-and-data.md) · [Next: Infrastructure as Code →](30-infrastructure-as-code.md)
