# Workshop W17 — Automate an AI Platform

**Goal:** Build a repeatable cloud platform with Terraform, then prove that it is safe to change and hand to another team.

**Time:** One weekend

**Prerequisites:** Terraform or OpenTofu, one cloud sandbox, and [Infrastructure as Code for AI Platforms](../06-ai-infrastructure/30-infrastructure-as-code.md)

Use the cloud where you already have access. Keep the GPU node pool disabled until the rest of the platform works.

---

## What you're building

Provision:

- A network with private workload subnets
- Identity for administrators, automation, and workloads
- A managed Kubernetes cluster
- A CPU system node pool
- An optional accelerator node pool
- Object storage for artifacts
- Logging and basic monitoring
- Remote Terraform state

The exact resource names differ by provider. The design and delivery controls should remain recognizable.

## Part 1: Set cost and safety limits

Before writing resources:

- Choose one sandbox account, subscription, or project.
- Set a budget and alert.
- Confirm the target region.
- Keep `enable_accelerator_pool = false` by default.
- Define an expiration time for temporary environments.
- Decide which resources need deletion protection.

Your first successful apply should not create a GPU.

## Part 2: Structure the repository

Start with:

```text
platform/
  modules/
    network/
    identity/
    kubernetes/
    accelerator-pool/
    storage/
    observability/
  environments/
    sandbox/
    production/
  tests/
  policies/
```

Keep provider configuration and environment values in the roots. Keep reusable resources inside modules.

Pin provider and module versions. Commit the dependency lock file.

## Part 3: Configure remote state

Create or select a remote backend with:

- Encryption
- Versioning
- Locking where supported
- Restricted administrator and automation access
- Recovery procedure

Do not store credentials in variables or state intentionally. Marking an output as sensitive hides it from normal display but does not remove it from state.

Test state locking by attempting two plans or applies against the same environment.

## Part 4: Build the network

Create:

- Address ranges with room for nodes, pods, services, and growth
- Private workload subnets
- Controlled outbound access
- Security rules for the Kubernetes API and node traffic
- Flow logs where available

Produce a short data-flow diagram:

```text
operator → control plane
workload → object storage
training nodes ↔ training nodes
inference gateway → model service
```

For every path, name the source, destination, port or protocol, identity, and logging point.

## Part 5: Add identity

Separate:

- Human administrator access
- CI/CD deployment identity
- Kubernetes node identity
- Workload identity for applications

Use least privilege. A training job that reads one artifact bucket should not inherit platform-administrator credentials.

Verify access with one allowed and one denied operation.

## Part 6: Provision Kubernetes

Create the control plane and a small CPU node pool.

Configure:

- Private nodes
- Restricted API access
- Audit and control-plane logs
- Autoscaling limits
- Approved node image or version
- Labels and taints
- Upgrade policy

Output only what downstream automation needs, such as cluster name, region, and workload-identity details.

Run the basic exercises from [Workshop W16](W16-kubernetes-ai-operations.md) against this cluster.

## Part 7: Define the accelerator pool

Implement the module without enabling it.

Inputs should include:

- Accelerator type
- Node count or autoscaling range
- Reservation or capacity reference
- Placement or fabric requirement
- Local storage
- Driver strategy
- Labels and taints
- Health and repair settings

Add validation that rejects:

- Unsupported regions or shapes
- A public worker configuration
- Zero maximum capacity when enabled
- Missing fabric settings for multi-node training

If budget and capacity allow, enable one small accelerator node briefly, validate device access, then disable it.

## Part 8: Keep application delivery separate

Use Terraform for cloud resources and cluster bootstrap.

Use Helm or Kustomize for Kubernetes applications. Add GitOps if continuous reconciliation is part of the design.

Use Ansible only for mutable host configuration that cannot be handled by an image, operator, or cloud-init process. Document why it is needed and how repeated runs remain safe.

Do not put training commands or model deployment scripts inside Terraform provisioners.

## Part 9: Add delivery checks

The pull-request path should run:

1. Format
2. Validate
3. Static security scan
4. Policy checks
5. Plan
6. Human review of the plan
7. Apply of the reviewed plan
8. Platform acceptance tests

Write policies for at least:

- No public worker addresses
- Encryption required
- Required ownership and cost tags
- Approved regions
- Accelerator pool disabled unless explicitly enabled

Save the plan and test result as build artifacts.

## Part 10: Integrate with Python

Write a small Python command that:

- Reads stable Terraform outputs
- Connects to the cluster or cloud API
- Confirms the expected nodes and storage exist
- Runs one smoke workload
- Produces a machine-readable report

Use explicit timeouts, bounded retries, and clear exit codes. Do not parse human-oriented Terraform output when JSON output exists.

## Part 11: Practice drift and recovery

Make one harmless console change.

Run a scheduled-style plan and decide whether to:

- Restore the declared configuration
- Import the real resource
- Update the code intentionally

Then simulate one state-recovery scenario using a copy or disposable environment. Record the commands and safeguards.

## Part 12: Promote and hand off

Create a production root that reuses the same modules with:

- Separate state
- Stricter access
- Larger address space
- Protected resources
- Explicit capacity and recovery settings

Do not apply production. Produce and review the plan.

Give another engineer only the repository and README. They should be able to create the sandbox, run the tests, understand the cost, and remove it safely.

## What you should have

- Reusable Terraform modules
- Separate sandbox and production roots
- Protected remote state
- Network and identity design
- Managed Kubernetes with a CPU pool
- Optional accelerator-pool module
- Policy and CI checks
- Python acceptance command
- Drift and recovery exercise
- Handoff documentation and cost controls

## Go deeper

- [Terraform module development](https://developer.hashicorp.com/terraform/language/modules/develop)
- [Terraform test](https://developer.hashicorp.com/terraform/language/tests)
- [Terraform style guide](https://developer.hashicorp.com/terraform/language/style)
- [Open Policy Agent](https://www.openpolicyagent.org/docs/latest/)
