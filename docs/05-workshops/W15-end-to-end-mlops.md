# Workshop W15 — End-to-End MLOps

**Goal:** Train a small model, track the experiment, register and validate it, deploy it, monitor it, then promote or roll it back.

**Time:** One weekend

**Prerequisites:** Python, Docker, and [The MLOps Lifecycle](../05-ai-workloads/25-mlops-lifecycle.md)

No GPU is required. The point is the lifecycle, not model complexity.

---

## What you're building

Use scikit-learn's built-in wine-classification dataset. Train two small models and manage them through this path:

```text
data validation
  → training
  → MLflow tracking
  → model registry
  → validation gate
  → local API
  → monitoring
  → promotion or rollback
```

Your repository should end with:

```text
mlops-workshop/
  src/
    validate_data.py
    train.py
    validate_model.py
    app.py
    monitor.py
  tests/
  pipeline.py
  Dockerfile
  requirements.txt
  README.md
```

## Part 1: Define the contract

Write down:

- Input feature names and types
- Allowed ranges
- Target classes
- Required quality metric
- Maximum prediction latency
- What makes a model eligible for promotion

Use macro F1 as the main metric so each wine class matters.

Example gates:

```yaml
quality:
  macro_f1_min: 0.90
  no_class_recall_below: 0.80
service:
  p95_latency_ms_max: 100
```

These numbers are workshop targets, not universal standards.

## Part 2: Validate and split the data

Load `sklearn.datasets.load_wine`.

Check:

- Expected columns
- Numeric types
- Missing values
- Class counts
- Duplicate rows
- Feature ranges

Create train, validation, and test splits. Use stratification so every split contains each class.

Save:

- Row counts
- Class distribution
- Split seed
- A content hash or manifest

The goal is to reproduce which data produced a model without committing a large dataset.

## Part 3: Track two experiments

Train:

1. Logistic regression as the simple baseline
2. A tree-based model such as histogram gradient boosting

For each MLflow run, log:

- Algorithm and hyperparameters
- Data manifest and split seed
- Source commit
- Package versions
- Training duration
- Macro F1
- Per-class precision and recall
- Confusion matrix
- Serialized model
- Input signature

Use a clear experiment and run-naming scheme.

Do not choose the more complex model automatically. Choose the model that passes the contract with the best trade-off.

## Part 4: Register a candidate

Register the selected model in MLflow Model Registry.

Add:

- Description
- Owner
- Dataset hash
- Task metrics
- Tags for algorithm and source commit
- The alias `candidate`

Use aliases rather than MLflow's deprecated fixed stages.

Your validation code should load:

```text
models:/wine-quality@candidate
```

Do not copy a local model path into the deployment step.

## Part 5: Build a validation gate

`validate_model.py` should:

1. Load the candidate from the registry.
2. Run the untouched test set.
3. Check macro F1.
4. Check recall for every class.
5. Validate the input and output signature.
6. Measure prediction latency.
7. Write a machine-readable report.
8. Return a non-zero exit code on failure.

Add tests for:

- A passing candidate
- A quality regression
- A missing feature
- A schema mismatch

The pipeline must stop before deployment when the gate fails.

## Part 6: Deploy behind an API

Build a small FastAPI service that loads:

```text
models:/wine-quality@champion
```

Expose:

- `POST /predict`
- `GET /health`
- `GET /model`
- `GET /metrics`

`/model` should return the registered model name, version, alias, and source run ID.

Build a container. Run the same container in tests and local deployment.

Do not reload the model on every request.

## Part 7: Promote the model

If the candidate passes:

1. Record the current champion version.
2. Assign the `champion` alias to the candidate.
3. Restart or reload the service through a controlled deployment.
4. Run smoke tests.
5. Save the promotion event and validation report.

Aliases change independently from application code. The deployment still needs to record which concrete version it loaded.

## Part 8: Monitor service and model behavior

Collect:

- Request count and errors
- p50 and p95 latency
- Schema failures
- Model version
- Prediction count by class
- Input-feature summaries

If simulated labels arrive later, calculate rolling macro F1 and per-class recall.

Create shifted input data by changing one or two feature distributions. Confirm that the monitoring report notices the distribution change.

Do not call the shift a quality regression unless labels prove it.

## Part 9: Roll back

Create a deliberately weak candidate or change the input distribution enough to fail a production check.

Rollback should:

1. Reassign `champion` to the previous version.
2. Redeploy or reload safely.
3. Run smoke tests.
4. Record the reason and affected versions.

Practice the rollback before calling the pipeline production-ready.

## Part 10: Make the lifecycle repeatable

Create `pipeline.py` or a Makefile with these commands:

```text
validate-data
train
register
validate-model
promote
serve
monitor
rollback
```

Then run the full lifecycle from a clean checkout.

The local orchestrator can be simple. The important part is that every step has declared inputs, outputs, and failure behavior.

## Optional: move the pipeline to Kubernetes

Convert the steps into Kubeflow Pipeline components:

- One container per step
- Object storage for artifacts
- Secrets through workload identity or a secret manager
- A CPU request for data validation
- A larger CPU or GPU request for training
- Cached steps only where inputs fully describe the result

Compile the pipeline, run it twice, and explain which steps can safely reuse cached output.

Do not put model-serving deployment inside the training container. Promotion should call a separate deployment path.

## Optional: adapt it to deep learning

Replace the scikit-learn model with a small PyTorch network.

Log:

- Epoch metrics
- Learning rate
- Checkpoints
- Device and training time
- Final model signature

The MLOps lifecycle stays the same. The infrastructure needs change.

## Review questions

- Can you reproduce the champion from code, data, and environment references?
- Can a model pass overall F1 while failing one class?
- What prevents a bad dataset from triggering a deployment?
- Which production signals need labels?
- How long does rollback take?
- Who owns each gate?
- What changes when training uses eight GPUs instead of a CPU?

## What you should have

- Versioned training and validation code
- At least two tracked runs
- A registered candidate and champion
- Automated quality and schema gates
- A containerized prediction API
- Service and model-behavior metrics
- A successful promotion
- A tested rollback
- A short architecture and ownership note

## Go deeper

- [MLflow Tracking](https://mlflow.org/docs/latest/ml/tracking/)
- [MLflow Model Registry workflow](https://mlflow.org/docs/latest/ml/model-registry/workflow/)
- [Kubeflow Pipelines](https://www.kubeflow.org/docs/components/pipelines/overview/)
- [scikit-learn model evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html)
