# AI Infrastructure Video Guide

Use videos to make the hardware and data movement less abstract. Pause when a diagram or formula appears and redraw it from memory.

The list mixes vendor-neutral explanations with vendor-specific implementations. Product videos are examples, not required architecture.

## A four-hour starting path

1. [Demystifying Artificial Intelligence and Machine Learning Infrastructure for a Network Engineer](https://www.youtube.com/watch?v=6kWQ7QkUR_I) — Tech Field Day, 52 min
2. [Multi-GPU Communication Libraries for Scaling HPC and AI Workloads | NVIDIA GTC 2025](https://www.youtube.com/watch?v=kyQtbyR536I) — NVIDIA Developer, 42 min
3. [Mastering LLM Inference Optimization From Theory to Cost Effective Deployment: Mark Moyou](https://www.youtube.com/watch?v=9tvJ_GYJA-o) — AI Engineer, 34 min
4. [Scale ANY Model: PyTorch DDP, ZeRO, Pipeline & Tensor Parallelism Made Simple](https://www.youtube.com/watch?v=r3DwMv84H9I) — Zachary Mueller, 30 min
5. [GPUs on Kubernetes: What Actually Happens When You Request Nvidia...](https://www.youtube.com/watch?v=nu6bLhuvlWM) — CNCF, 26 min
6. [A deep dive into NVIDIA Blackwell with SemiAnalysis' Dylan Patel](https://www.youtube.com/watch?v=Fq4SM9XZ-8U) — Together AI, 47 min

## GPU systems

- [NVIDIA Blackwell: The Journey From Die to Data Center](https://www.youtube.com/watch?v=1la6fMl7xNA) — NVIDIA, 4 min
- [A deep dive into NVIDIA Blackwell with SemiAnalysis' Dylan Patel](https://www.youtube.com/watch?v=Fq4SM9XZ-8U) — Together AI, 47 min
- [A Deep Dive on How To Leverage the NVIDIA GB200 for Ultra-Fast...](https://www.youtube.com/watch?v=Nlgp4T4kvdE) — CNCF, 28 min

Pair with [GPU Systems](../06-ai-infrastructure/24-gpu-systems.md).

## Cluster networking

- [Demystifying Artificial Intelligence and Machine Learning Infrastructure for a Network Engineer](https://www.youtube.com/watch?v=6kWQ7QkUR_I) — Tech Field Day, 52 min
- [Multi-GPU Communication Libraries for Scaling HPC and AI Workloads | NVIDIA GTC 2025](https://www.youtube.com/watch?v=kyQtbyR536I) — NVIDIA Developer, 42 min
- [Demystifying NCCL: An In-depth Analysis of GPU Communication Protocols and Algorithms](https://www.youtube.com/watch?v=UDCLvBqEPcA) — HOTI, 29 min

Pair with [Cluster Networking](../06-ai-infrastructure/25-cluster-networking.md).

## Distributed training

- [Scale ANY Model: PyTorch DDP, ZeRO, Pipeline & Tensor Parallelism Made Simple](https://www.youtube.com/watch?v=r3DwMv84H9I) — Zachary Mueller, 30 min
- [Training Models Too Big for One GPU | ZeRO & FSDP](https://www.youtube.com/watch?v=nx5yfCa4W98) — Jia-Bin Huang, 23 min

Pair with [Distributed Training Systems](../05-ai-workloads/21-distributed-training.md).

## Inference serving

- [Mastering LLM Inference Optimization From Theory to Cost Effective Deployment: Mark Moyou](https://www.youtube.com/watch?v=9tvJ_GYJA-o) — AI Engineer, 34 min
- [Fast LLM Serving with vLLM and PagedAttention](https://www.youtube.com/watch?v=5ZlavKF_98U) — Anyscale, 32 min

Pair with [Self-Hosted Inference](../05-ai-workloads/22-self-hosted-inference.md).

## Kubernetes and Slurm

- [GPUs on Kubernetes: What Actually Happens When You Request Nvidia...](https://www.youtube.com/watch?v=nu6bLhuvlWM) — CNCF, 26 min
- [DRA is GA! Kubernetes WG Device Management — GPUs, TPUs, NICs and More](https://www.youtube.com/watch?v=Op4DNDTij1U) — CNCF, 30 min
- [Choosing the right platform: Slurm vs Kubernetes](https://www.youtube.com/watch?v=cTGUZLeZmac) — Nebius, 53 min
- [Why Kubernetes Struggles with Batch Jobs (And How Kueue Fixes It)](https://www.youtube.com/watch?v=SjNFiFvSdLQ) — Ops and Odds, 5 min
- [Managed Soperator overview: Slurm on Kubernetes for scalable AI workloads](https://www.youtube.com/watch?v=iQstLUA0oiA) — Nebius, 11 min

Pair with [Kubernetes and Slurm](../06-ai-infrastructure/27-kubernetes-and-slurm.md).

## Infrastructure as Code

- [Terraform Practices: The Good, the Bad, and the Ugly](https://www.youtube.com/watch?v=L3uLFTI8wpU) — HashiCorp, 15 min
- [Block-by-Block: Understanding Terraform Module Design Through Minecraft](https://www.youtube.com/watch?v=lmghqJIK8EE) — HashiCorp, 29 min

Pair with [Infrastructure as Code](../06-ai-infrastructure/28-infrastructure-as-code.md).

## Storage

- [Kidger 2019: Parallel Filesystems for AI and HPC](https://www.youtube.com/watch?v=IvHR1QURGac) — Daniel Kidger, 55 min
- [AI Explained: Checkpointing in LLMs and the Trade-Offs between Reliability and Performance](https://www.youtube.com/watch?v=3Rwgb8s_49g) — WEKA, 4 min

Pair with [Storage and the Data Path](../06-ai-infrastructure/26-storage-and-data.md).

## Transformer refresher

- [Transformers, the tech behind LLMs | Deep Learning Chapter 5](https://www.youtube.com/watch?v=wjZofJX0v4M) — 3Blue1Brown, 27 min
- [Attention in transformers, step-by-step | Deep Learning Chapter 6](https://www.youtube.com/watch?v=eMlx5fFNoYc) — 3Blue1Brown, 26 min

Pair with [How LLMs Actually Work](../01-foundations/01-llm-mental-model.md).

## Architecture and communication

- [Solutions Architect: Discovery Process](https://www.youtube.com/watch?v=yaeSD6HJFzY) — Aced, 14 min
- [Solutions Architect Mock Customer Demo](https://www.youtube.com/watch?v=JnmpeY540Ig) — Aced, 28 min
- [How Nebius build cloud for AI workloads](https://www.youtube.com/watch?v=lrxU7Z4p3Vs) — Nebius, 34 min
- [How dedicated endpoints work on Nebius Token Factory](https://www.youtube.com/watch?v=JHNSPUTBoP0) — Nebius, 5 min

Pair with [From Proof of Concept to Production](../07-enterprise-architecture/31-poc-to-production.md).

## Python practice

- [Python for Coding Interviews — Everything you need to Know](https://www.youtube.com/watch?v=0K_eZGS5NsU) — NeetCode, 26 min
- [Data Structure and Algorithm Patterns for LeetCode Interviews](https://www.youtube.com/watch?v=Z_c4byLrNBU) — freeCodeCamp.org, 1 h 15 min

These are optional refreshers. The curriculum's practical path is [Workshop W11 — Python for AI Infrastructure](../05-workshops/W11-python-ai-infrastructure.md).
