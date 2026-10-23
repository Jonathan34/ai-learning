# Secure AI Platforms

GPU infrastructure does not need a special theory of security. It does need the usual controls applied to some unusual data paths and very expensive shared hardware.

## Start with boundaries

Before choosing controls, identify:

- Who can submit workloads
- Which data classifications are allowed
- Where data may be stored and processed
- Which teams may share hardware
- Which services need internet access
- Who can administer the cloud, cluster, scheduler, and model service
- What evidence security and audit teams need

Draw the trust boundaries. A product list is not a security design.

## Identity

Use the organization's identity provider for people. Use short-lived workload identities for services.

The basic pattern:

- Federation and single sign-on for users
- Groups mapped to platform roles
- Service accounts or workload identity for applications
- Least privilege
- Separate break-glass administration
- No long-lived cloud keys in notebooks or CI variables

Kubernetes RBAC is only one layer. Cloud IAM, scheduler accounts, model gateways, object storage, and secrets systems all need a consistent ownership model.

## Network design

Keep GPU workers private by default.

Separate:

- User and API ingress
- Management traffic
- Storage and service traffic
- GPU compute-fabric traffic

Use controlled egress for package and model downloads. A private cluster that can freely call the internet is only partly private.

Compute-fabric isolation also matters. Use fabric partitions or dedicated fabrics where supported, plus host and cluster controls. Test that one tenant cannot reach another tenant's devices.

## Data protection

Classify data before it reaches the platform.

Controls usually include:

- Encryption in transit and at rest
- Customer-managed keys where required
- Private storage endpoints
- Bucket and filesystem policies
- Secrets outside images and source code
- Retention and deletion rules
- Backup and restore tests
- Logs that avoid raw prompts, outputs, and training examples by default

Training copies data into caches, checkpoints, logs, and artifacts. The data inventory must include all of them.

## Multi-tenancy

Multi-tenancy has several levels:

| Boundary | Good for | Limitation |
|---|---|---|
| Namespace and quota | Trusted teams with shared operations | Weak hardware and control-plane boundary |
| Separate node pools | Performance and operational separation | Shared cluster remains |
| MIG | Small workloads needing GPU-level isolation | Limited shapes and not suited to large training |
| Separate clusters or projects | Stronger team and policy boundary | More cost and operational work |
| Dedicated hardware or fabric | Strict performance or regulatory needs | Highest cost and lowest flexibility |

Choose the boundary from the threat model and noisy-neighbor tolerance, not convenience alone.

## Software supply chain

GPU images are large and contain deep dependency stacks.

Protect them with:

- Approved base images
- Pinned driver, CUDA, NCCL, and framework versions
- Image scanning and signatures
- A private registry
- Software bills of materials
- Controlled model and dataset sources
- Rebuild and patch procedures

Model weights are executable inputs in a broad sense. Loading code from an untrusted model repository can run arbitrary Python. Disable remote code unless it is reviewed and required.

## Model-service controls

Inference adds application concerns:

- Authentication and per-tenant authorization
- Rate and token limits
- Request-size limits
- Abuse detection
- Data-retention controls
- Model and version allowlists
- Audit events for routing and configuration changes
- Safe rollout and rollback

The model gateway is a policy boundary, not only a convenience proxy.

## Governance without freezing delivery

Create a paved path:

- Approved modules and images
- Clear data classes
- Default-private networking
- Standard logging and metrics
- A review path for exceptions
- Automated evidence collection

If every workload needs a new security design, teams route around the platform. If every workload is allowed by default, security eventually stops the platform.

## Shared responsibility

Write down who owns:

- Physical and fabric health
- Cloud identity and network controls
- Cluster upgrades
- GPU driver and base-image updates
- Scheduler configuration
- Workload code and dependencies
- Model behavior and evaluation
- Data governance
- Incident response

Ambiguous ownership shows up during the first incident.

## Things that trip people up

**Calling a shared cluster "isolated" because it has namespaces.** Namespaces are not a hardware boundary.

**Logging full prompts and datasets for observability.** Collect the minimum needed, with access and retention controls.

**Giving notebooks broad credentials.** Interactive environments are a common path to data and cloud resources.

**Treating compliance reports as the architecture.** Attestations support the control story; they do not replace workload-specific design.

## Go deeper

- [Kubernetes security overview](https://kubernetes.io/docs/concepts/security/)
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [SLSA supply-chain framework](https://slsa.dev/)
- [Workshop W14 — Enterprise AI Platform Capstone](../05-workshops/W14-enterprise-ai-platform.md)

---

[← Previous: Reliability and Performance](../06-ai-infrastructure/29-reliability-and-performance.md) · [Next: From Proof of Concept to Production →](31-poc-to-production.md)
