# Cloud ML Platform Landscape

The product names change. The architecture layers change much more slowly.

A useful comparison starts with the workload and maps each platform to the same needs:

```text
data and features
  → training or inference
  → accelerator and network
  → orchestration
  → model lifecycle
  → security and operations
```

Do not compare a managed model API with a raw GPU cluster as if they were equivalent products.

## Five levels of control

| Platform shape | You manage | Good fit |
|---|---|---|
| Managed model API | Prompts, application, evaluation, and data policy | Fast product delivery without model infrastructure |
| Managed ML platform | Data, training code, models, pipelines, and deployment policy | Teams wanting an integrated ML lifecycle |
| Managed Kubernetes | Containers, operators, scheduling policy, and services | Platform teams with cloud-native operating skills |
| Managed HPC cluster | Images, jobs, scheduler policy, data path, and recovery | Large synchronous training and HPC-native teams |
| GPU VMs or bare metal | Almost the entire software and operating stack | Maximum control or specialized performance needs |

More control is not automatically better. It creates more integration, upgrade, security, and on-call work.

## NVIDIA

NVIDIA supplies much of the accelerator software used across clouds:

- CUDA for GPU programming and runtime APIs
- cuBLAS and cuDNN for accelerated math and neural-network operations
- NCCL for communication between GPUs
- GPU Operator and Network Operator for Kubernetes
- NeMo for model development and training
- Triton Inference Server, TensorRT-LLM, and NIM for model serving
- DCGM for health and telemetry

NVIDIA is a layer in many cloud architectures, not a replacement for networking, storage, identity, orchestration, or the ML lifecycle.

## AWS

Common building blocks include:

- Amazon SageMaker AI for managed training, pipelines, registry, deployment, and monitoring
- Amazon EKS for Kubernetes-based AI platforms
- AWS ParallelCluster for HPC clusters and Slurm
- Amazon EC2 GPU instances, plus Trainium and Inferentia accelerators
- Elastic Fabric Adapter for low-latency cluster communication
- Amazon S3 for durable objects and FSx for Lustre for high-throughput shared data
- Amazon Bedrock for managed foundation-model access

The architecture question is often whether the team needs SageMaker's integrated control plane, EKS flexibility, a Slurm cluster, or only a managed model API.

## Microsoft Azure

Common building blocks include:

- Azure Machine Learning for managed jobs, pipelines, registries, endpoints, and monitoring
- Azure Kubernetes Service for Kubernetes-based platforms
- Azure CycleCloud for creating and operating HPC environments, including Slurm
- N-series virtual machines for accelerated computing
- InfiniBand-enabled VM options for tightly coupled workloads
- Blob Storage or Azure Data Lake Storage for objects and data lakes
- Azure Managed Lustre for high-throughput shared storage

Enterprise designs often begin with identity, network topology, policy, and existing data services before choosing the ML execution layer.

## Google Cloud

Common building blocks include:

- Vertex AI for managed training, pipelines, registry, endpoints, and monitoring
- Google Kubernetes Engine for Kubernetes-based AI platforms
- Cluster Toolkit for repeatable HPC and AI environments
- Compute Engine GPU instances and Cloud TPUs
- Cloud Storage for durable objects
- Parallelstore or Filestore for shared filesystem needs

The main choice is usually between Vertex AI's managed lifecycle, GKE's platform flexibility, and a more HPC-shaped environment for large training.

## Compare platforms by questions, not logos

### Workload

- Training, fine-tuning, batch inference, or online inference?
- Which framework, model size, precision, and parallelism?
- Which quality, latency, throughput, completion-time, and availability targets?

### Accelerators and fabric

- Which GPU or accelerator shapes are available in the required region?
- What is the memory, local interconnect, and cross-node fabric?
- Can the scheduler place the complete job on the right topology?
- What measured scaling efficiency does the workload achieve?

### Data path

- Where does the data live now?
- What must be copied, cached, mounted, or streamed?
- Which storage handles active datasets, checkpoints, models, and durable retention?
- What are the migration time and egress costs?

### Model lifecycle

- How are data, code, runs, environments, and models versioned?
- Which pipeline, registry, validation, deployment, monitoring, and rollback path exists?
- Can these services integrate with the customer's existing tools?

### Enterprise controls

- How do identity federation, workload identity, private access, keys, logs, and policy work?
- Which residency, compliance, tenancy, and support requirements apply?
- Who operates each layer and responds when it fails?

### Economics and portability

- What is the cost at realistic utilization, including people and migration?
- Is capacity committed, elastic, or uncertain?
- Which interfaces are portable: containers, Kubernetes, Slurm, Terraform, S3 APIs, or model formats?
- What would it cost and take to leave?

## A practical comparison exercise

Choose one training workload and one inference workload.

For each of NVIDIA's software stack, AWS, Azure, and Google Cloud:

1. Select a managed and a self-managed path.
2. Draw the compute, network, storage, identity, and MLOps layers.
3. Name what the provider operates and what the customer operates.
4. State the likely performance and adoption risks.
5. Define a proof of concept that would decide between the options.

The goal is not to declare one universal winner. It is to explain why a platform fits a particular customer and workload.

## Official starting points

- [NVIDIA AI Enterprise documentation](https://docs.nvidia.com/ai-enterprise/)
- [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/)
- [Amazon SageMaker AI](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html)
- [Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [AWS ParallelCluster](https://docs.aws.amazon.com/parallelcluster/latest/ug/what-is-aws-parallelcluster.html)
- [Azure Machine Learning](https://learn.microsoft.com/azure/machine-learning/overview-what-is-azure-machine-learning)
- [Azure Kubernetes Service](https://learn.microsoft.com/azure/aks/what-is-aks)
- [Azure CycleCloud](https://learn.microsoft.com/azure/cyclecloud/overview)
- [Vertex AI](https://cloud.google.com/vertex-ai/docs/start/introduction-unified-platform)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)
- [Cluster Toolkit](https://cloud.google.com/cluster-toolkit/docs/overview)
