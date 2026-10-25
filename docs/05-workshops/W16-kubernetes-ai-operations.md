# Workshop W16 — Kubernetes Operations for AI

**Goal:** Build and operate a small Kubernetes platform that has the same control points as an AI cluster.

**Time:** One weekend

**Prerequisites:** Docker, `kubectl`, Helm, and [Kubernetes and Slurm](../06-ai-infrastructure/29-kubernetes-and-slurm.md)

Use `kind`, `k3d`, or an existing development cluster. A GPU is optional. The local version uses labels and taints to represent accelerator pools.

---

## What you're building

Create a cluster with:

- Separate system and workload nodes
- Team namespaces and service accounts
- RBAC and resource quotas
- Network policies through a CNI
- Persistent storage through a CSI driver
- Queued batch workloads
- A small inference-style service
- Monitoring, failure exercises, and an upgrade runbook

Keep all manifests, Helm values, test commands, and findings in version control.

## Part 1: Create and inspect the cluster

Create one control-plane node and at least three workers.

Label the workers:

```text
pool=system
pool=inference
pool=training
```

Taint the training node so ordinary pods cannot use it:

```text
workload=training:NoSchedule
```

Record:

- Kubernetes version
- Node operating system and container runtime
- Internal addresses
- Labels, taints, capacity, and allocatable resources
- CNI and CSI components

Be able to explain which parts a managed provider operates and which remain yours.

## Part 2: Add tenancy and access

Create:

- `platform-system` namespace
- `team-a` and `team-b` namespaces
- One service account per team
- Roles that allow each team to manage its own jobs and read its own logs
- Resource quotas and default limits

Verify that:

- Team A cannot read Team B's secrets or workloads.
- A team service account cannot create cluster-wide resources.
- Platform operators can inspect both namespaces.

Do not solve the exercise by granting `cluster-admin`.

## Part 3: Exercise the CNI

Use a CNI that enforces NetworkPolicy. Install one if the local cluster's default does not.

Apply a default-deny policy to `team-a`, then allow only:

- DNS
- Traffic from an approved gateway
- Egress to a small mock object-storage or model API

Test an allowed and denied connection from inside a pod.

Then break one rule deliberately. Use pod events, DNS tools, routes, policies, and service endpoints to find the cause.

The pod network is part of the application path, not control-plane plumbing to ignore.

## Part 4: Exercise the CSI path

Install or use a dynamic storage provisioner.

Create:

- A StorageClass
- A persistent volume claim
- A job that writes an artifact
- A second pod that reads the artifact after the first pod is gone

Inspect the claim, volume, provisioner, attachment, and mount events.

Explain the difference between:

- ReadWriteOnce storage for one node
- ReadWriteMany storage for shared access
- Ephemeral local storage for caches and scratch data

Inject one failure, such as a missing StorageClass or an impossible access mode, and diagnose the Pending pod.

## Part 5: Control workload placement

Create a training-style Job that requires:

- `pool=training`
- The matching toleration
- Explicit CPU and memory requests

Create an inference-style Deployment that requires `pool=inference`.

Then test:

1. Missing toleration
2. Wrong node label
3. Request larger than node capacity
4. Namespace quota exhausted

For each Pending workload, find the scheduler's reason in events before changing anything.

## Part 6: Queue batch work

Install Kueue and create:

- A ResourceFlavor for the training pool
- A ClusterQueue with limited quota
- A LocalQueue for each team
- Two jobs that cannot run together

Observe one job being admitted while the other waits without holding part of the requested resources.

Add priority or fair-sharing policy, then explain:

- Who may borrow idle capacity
- What happens when the lender needs it back
- Whether running work can be preempted
- How topology and gang requirements would affect a multi-node GPU job

## Part 7: Operate an inference-style service

Deploy a small HTTP service with:

- Readiness and liveness probes
- Resource requests and limits
- A Service
- At least two replicas
- A PodDisruptionBudget
- A rolling-update policy
- Horizontal scaling from a CPU or request metric

Run load while performing an update. Measure errors and latency.

Then make the readiness probe fail and confirm that Kubernetes stops routing new traffic to the unhealthy pod.

## Part 8: Troubleshoot injected failures

Create and diagnose at least six failures:

- Image pull error
- CrashLoopBackOff
- Pending because of scheduling
- DNS or NetworkPolicy failure
- Persistent-volume failure
- Failed readiness probe
- Queue admission delay
- Node drain with insufficient spare capacity

For each failure, record:

1. User-visible symptom
2. First useful command
3. Evidence that identified the layer
4. Fix
5. Test that prevents recurrence

## Part 9: Write the day-two runbook

Document:

- Node replacement
- Cordon and drain
- Control-plane and node upgrades
- CNI, CSI, and operator upgrades
- Certificate and secret rotation
- Capacity exhaustion
- Backup and restore
- Workload-owner notification

Define an upgrade order and rollback point. Run one simulated worker replacement and rerun the platform tests.

## Optional: use real GPUs

On a GPU cluster:

- Install or inspect the GPU Operator.
- Compare a device plugin with Dynamic Resource Allocation.
- Confirm `nvidia.com/gpu` or the expected resource claim.
- Run `nvidia-smi` inside a pod.
- Inspect DCGM metrics.
- Expose the RDMA device where required.
- Run a small NCCL test across two nodes.

Record the tested Kubernetes, driver, CUDA, operator, CNI, CSI, and NCCL versions as one compatibility set.

## Optional: run the same work through Slurm

On an existing Slurm or Slurm-on-Kubernetes environment:

1. Write an `sbatch` script requesting nodes, tasks, GPUs, time, and an output path.
2. Launch the distributed process with `srun`.
3. Inspect allocation, placement, state, and reason codes with `squeue` and `scontrol`.
4. Review completed-job accounting with `sacct`.
5. Cancel, requeue, and restore one checkpointed job.
6. Compare Slurm partitions, QOS, reservations, and fair-share with the Kubernetes queue controls from this workshop.

The goal is to understand the user's workflow and the platform boundary, not to memorize every `slurm.conf` option.

## What you should have

- Reproducible cluster setup
- Namespaces, RBAC, quotas, and network policies
- Working persistent storage
- Controlled workload placement
- A queued batch workflow
- A resilient inference-style service
- Six diagnosed failures
- An upgrade and recovery runbook
- Optional GPU and RDMA validation results
- Optional Slurm job, accounting, and recovery notes

## Go deeper

- [Kubernetes concepts](https://kubernetes.io/docs/concepts/)
- [Kubernetes NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Kubernetes persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Kueue documentation](https://kueue.sigs.k8s.io/docs/)
- [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/)
