# ML Algorithms for Architects

You do not need to implement machine-learning algorithms from scratch to design an ML platform.

You do need to understand what problem each algorithm family solves, what data it needs, how success is measured, and what it asks from the infrastructure.

## Algorithm, model, and training

These words are related but not interchangeable.

An **algorithm** is the method used to learn patterns. Gradient boosting and logistic regression are algorithms.

A **model** is the learned artifact produced by training. It contains the parameters the algorithm found.

**Training** is the process of giving the algorithm examples and adjusting the model to reduce its errors.

**Inference** uses the trained model to make a prediction on new data.

The same use case can often use several algorithms. The right starting point is usually the simplest one that meets the quality, latency, cost, and explainability needs.

## Start with the shape of the answer

Before choosing an algorithm, ask what the system must produce.

| Problem | Output | Common examples |
|---|---|---|
| Regression | A number | Demand, price, duration, risk score |
| Classification | One or more labels | Fraud or not, ticket category, defect type |
| Ranking | An ordered list | Search results, recommendations, candidate matches |
| Clustering | Groups without known labels | Customer segments, document groups |
| Anomaly detection | An unusualness score | Equipment failure, suspicious traffic |
| Forecasting | Future values over time | Capacity, inventory, energy demand |
| Generation | New text, image, audio, or code | Summaries, assistants, content creation |

If the desired output is unclear, the ML problem is not ready for architecture.

## Supervised learning

**Supervised learning** trains from examples that already include the correct answer, called the **label**.

### Regression

Regression predicts a number.

Common starting points:

- Linear regression when relationships are simple and explainability matters
- Decision trees or random forests for mixed tabular data
- Gradient-boosted trees for strong performance on structured business data
- Neural networks when the data or relationship is large and complex

Useful metrics:

- **MAE**, mean absolute error: the average size of the mistake
- **RMSE**, root mean squared error: penalizes large mistakes more heavily
- Business error, such as dollars lost or capacity missed

Always compare with a simple baseline. A forecast that cannot beat "use last week's value" is not useful because it uses ML.

### Classification

Classification predicts a category or probability.

Common options:

- Logistic regression for a fast, explainable baseline
- Decision trees, random forests, or boosted trees for tabular data
- Neural networks for images, audio, text, or very large datasets
- An LLM when the input is unstructured language and labels require semantic judgment

Accuracy can be misleading when one class is rare.

Use:

- **Precision:** when false positives are expensive
- **Recall:** when missing a positive case is expensive
- **F1:** a balance between precision and recall
- **ROC-AUC or PR-AUC:** how well scores separate classes across thresholds
- A confusion matrix to see which mistakes the model makes

The business usually cares about a particular error, not an abstract metric.

## Unsupervised learning

**Unsupervised learning** looks for structure without known labels.

### Clustering

Clustering groups similar examples.

- K-means is simple and works best for roughly compact groups.
- DBSCAN can find irregular groups and mark isolated points as noise.
- Hierarchical clustering gives a tree of groupings.

The hard part is proving the groups are useful. A technically clean cluster that changes no decision has little value.

### Dimensionality reduction

Dimensionality reduction compresses many features into fewer dimensions.

PCA is useful for compression and broad structure. UMAP and t-SNE are often used for visualization, but their plots can make weak patterns look stronger than they are.

Embeddings are also a learned compressed representation. They support semantic search, clustering, recommendation, and retrieval.

### Anomaly detection

Anomaly detection finds examples unlike the normal pattern.

Options include statistical thresholds, isolation forests, one-class models, and autoencoders. The operational problem is usually false alarms. A model that finds every anomaly but pages someone all day will be disabled.

## Ranking and recommendations

Ranking predicts which items should appear first.

Common approaches:

- Rules and popularity baselines
- Collaborative filtering from user-item interactions
- Matrix factorization
- Two-tower neural models that embed users and items
- Learning-to-rank models using many features

Offline metrics such as precision at K, recall at K, and NDCG measure ordering quality. Online experiments measure whether the ranking changes real behavior.

Recommendations create feedback loops: what the system shows affects what users click, which becomes future training data.

## Forecasting

Time-series data must be split by time, not randomly. Training on future information is data leakage.

Start with:

- Last value
- Moving average
- Seasonal baseline

Then consider statistical models, boosted trees with time-based features, or neural sequence models.

Backtesting means repeatedly training on the past and testing on the next time window. One train/test split is rarely enough.

## Deep learning

Deep learning uses neural networks with many layers.

Common families:

- Convolutional networks and vision transformers for images
- Transformers for language and increasingly vision, audio, and time series
- Autoencoders for representation learning and anomaly detection
- Multi-tower networks for recommendation and retrieval

Deep learning becomes attractive when the input is unstructured, the dataset is large, or pretrained models can be adapted. It also brings higher compute, data, and operational cost.

## Conventional ML or an LLM?

Use conventional ML when:

- The output is a stable label, number, or score
- You have good structured features and labels
- Millisecond latency and low unit cost matter
- Explainability or deterministic behavior matters
- A small model already meets the goal

Use an LLM when:

- The input or output is natural language
- The task needs broad semantic knowledge
- The labels are hard to express as fixed rules
- Generation or multi-step language reasoning is part of the product

The two can work together. A conventional model can route requests, score risk, or rank candidates before an LLM handles the language-heavy part.

## Training concepts you need

**Features** are the inputs used by the model. **Labels** are the answers used during supervised training.

Split data into:

- Training data for learning parameters
- Validation data for choosing settings and models
- Test data for the final unbiased check

**Overfitting** means the model learned the training examples too specifically and performs poorly on new data.

**Regularization** discourages unnecessary complexity. Early stopping ends training when validation results stop improving.

**Data leakage** means training had access to information that would not exist when the prediction is made. Leakage creates impressive offline results and a broken production model.

**Class imbalance** means one label is much rarer than another. Use suitable sampling, thresholds, and metrics rather than trusting accuracy.

## What the code usually looks like

Libraries implement the algorithm. Application code defines the data, model settings, training, and evaluation:

```python
from sklearn.ensemble import HistGradientBoostingClassifier

model = HistGradientBoostingClassifier(max_depth=6)
model.fit(train_features, train_labels)
probabilities = model.predict_proba(test_features)
```

The important architecture questions are outside these three lines:

- Where did the features and labels come from?
- Can the dataset and environment be reproduced?
- Which metric chooses the model?
- Where is the artifact stored?
- How is it promoted and deployed?
- How will production quality be measured?

Those are MLOps questions.

## Infrastructure implications

| Workload | Typical pressure |
|---|---|
| Linear models and small trees | CPU, memory, and data preparation |
| Large boosted-tree training | CPU or GPU compute, memory, distributed data movement |
| Deep-learning training | GPU memory and compute, cluster network, checkpoints |
| Online tabular prediction | CPU latency, feature availability, consistency |
| Large-model inference | GPU memory, HBM bandwidth, KV cache, batching |
| Embedding pipelines | Batch throughput, vector storage, data refresh |

Do not prescribe a GPU because the project is called AI.

## Questions to ask an ML team

- What decision or user outcome does the model change?
- What is the simplest baseline?
- Where do labels come from, and how reliable are they?
- Which error is most expensive?
- How large is the dataset, and how often does it change?
- Is inference online, batch, or both?
- Which feature or data dependencies must exist at prediction time?
- How will production labels arrive?
- What triggers retraining?
- What must be reproducible for audit or debugging?

## Things to watch for

**Choosing an algorithm before defining the metric.** The metric is part of the problem.

**Using deep learning for small structured data by default.** Boosted trees are often a stronger and cheaper baseline.

**Treating a high offline score as production proof.** Data drift, latency, and feedback loops still matter.

**Randomly splitting time-series or grouped data.** Related examples leak across the split.

**Building custom algorithm code.** Use established libraries unless the algorithm itself is the product.

## Go deeper

- [Google Machine Learning Crash Course](https://developers.google.com/machine-learning/crash-course)
- [scikit-learn supervised learning guide](https://scikit-learn.org/stable/supervised_learning.html)
- [scikit-learn unsupervised learning guide](https://scikit-learn.org/stable/unsupervised_learning.html)
- [PyTorch beginner fundamentals](https://pytorch.org/tutorials/beginner/basics/intro.html)
- [Workshop W15 — End-to-End MLOps](../05-workshops/W15-end-to-end-mlops.md)

---

[Next: Distributed Training Systems →](22-distributed-training.md)
