# Infrastructure as Code for AI Platforms

A useful proof of concept should be small, not disposable.

Infrastructure as Code, or IaC, lets the same design grow from a few GPUs into a controlled platform without rebuilding it by hand.

## Split the layers

A practical repository separates:

```text
modules/
  network/
  identity/
  gpu-cluster/
  kubernetes/
  storage/
  observability/

environments/
  sandbox/
  staging/
  production/
```

Modules should be opinionated enough to be safe and small enough to understand. Environment roots should mostly select versions, size, regions, and policy.

Use separate state and access boundaries for production. Terraform workspaces can be useful for identical temporary copies, but they are not a strong isolation boundary.

## State is sensitive

Terraform state maps code to real resource IDs. It may also contain sensitive values.

For a team:

- Store it remotely
- Enable locking
- Encrypt it
- Restrict access
- Keep versioned backups
- Never commit it

If state is damaged, restore the backup first. Then use `plan`, `state` commands, and import blocks to reconcile. Hand-editing should be the last move, with a copy preserved.

## A safe delivery flow

On every change:

1. Format and validate.
2. Run static and policy checks.
3. Produce a plan.
4. Review the plan with the code.
5. Apply the exact reviewed plan through automation.
6. Run platform acceptance tests.

Scheduled plans can detect drift. When drift appears, either restore the declared state or update the code deliberately.

## What belongs where

| Tool | Best responsibility |
|---|---|
| Terraform or OpenTofu | Cloud networks, clusters, node groups, IAM, storage |
| Image builder | Drivers and host packages that should be immutable |
| Helm or Kustomize | Kubernetes applications |
| Argo CD or Flux | Continuous reconciliation of cluster applications |
| Ansible | Host configuration where immutable images or operators do not fit |

Do not use Terraform as a general-purpose remote shell.

## GPU-specific inputs

An AI platform module often needs inputs for:

- GPU type and node count
- Fabric or placement domain
- Reservation or capacity policy
- Driver image or driver version
- Local disks and shared storage
- Scheduler and queue configuration
- Topology labels
- Health checks and observability
- Public endpoint policy

Validate these inputs. A typo that silently creates CPU nodes is annoying. A typo that creates public GPU nodes is a security problem.

## Policy as code

Good defaults reduce review load:

- No public IPs on worker nodes
- Control-plane access limited to known networks
- Encryption enabled
- Required tags or labels
- Approved GPU types and regions
- Protected stateful resources
- Audit logging enabled

Use policy checks in CI, but also enforce critical controls in the cloud and cluster. A skipped pipeline should not remove the security boundary.

## From PoC to production

The PoC can omit some operational depth. It should not omit the shape of production.

Build the PoC with:

- The real modules at a smaller size
- A real remote state backend
- Reproducible benchmark jobs
- Basic identity and network controls
- Outputs that make handoff easy

Later phases add federation, stronger isolation, reservations, policy, backup, incident response, and multi-environment promotion.

## Things to watch for

**Local state for a team.** It creates collisions and puts recovery on one laptop.

**One giant module.** Every change becomes risky and reuse becomes mostly theoretical.

**Installing drivers at every boot.** Startup becomes slow and unreliable. Prefer validated images or an operator.

**Using `ignore_changes` to silence real drift.** Ignore only fields another controller intentionally owns.

**Embedding benchmark commands in a wiki.** Put acceptance tests in versioned code and run them after changes.

## Go deeper

- [Terraform documentation](https://developer.hashicorp.com/terraform/docs)
- [Terraform module design](https://developer.hashicorp.com/terraform/language/modules/develop)
- [Open Policy Agent](https://www.openpolicyagent.org/docs/latest/)
- [Argo CD documentation](https://argo-cd.readthedocs.io/)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#infrastructure-as-code)

---

[← Previous: Kubernetes and Slurm](27-kubernetes-and-slurm.md) · [Next: Reliability and Performance →](29-reliability-and-performance.md)
