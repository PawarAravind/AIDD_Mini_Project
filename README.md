# CYP450 Substrate Liability Prediction

### Machine Learning for ADMET and Drug Metabolism

**Predicting whether small-molecule compounds are substrates of major Cytochrome P450 enzymes using molecular fingerprints and machine learning.**

[![Python](https://img.shields.io/badge/Python-3.12%2B-blue)](https://www.python.org/)
[![RDKit](https://img.shields.io/badge/Cheminformatics-RDKit-green)](https://www.rdkit.org/)
[![PyTorch](https://img.shields.io/badge/Deep%20Learning-PyTorch-orange)](https://pytorch.org/)
[![Domain](https://img.shields.io/badge/Domain-ADMET-purple)](https://en.wikipedia.org/wiki/ADME)

---

## 1. Project Overview

Cytochrome P450 (CYP450) enzymes play a central role in drug metabolism. Whether a compound acts as a substrate of a particular CYP enzyme can influence its metabolic clearance, systemic exposure, and potential for drug–drug interactions.

This project develops a machine-learning pipeline to predict **substrate versus non-substrate classification** for five major human CYP450 isoforms:

- CYP1A2
- CYP2C19
- CYP2C9
- CYP2D6
- CYP3A4

The project investigates traditional machine-learning baselines and multitask neural networks using molecular fingerprints. It also evaluates generalization to compounds and molecular scaffolds not observed during training.

**Primary objective:** Build and evaluate a reproducible computational workflow for CYP450 substrate-liability prediction while identifying the limitations of fingerprint-based models and multitask learning.

> **Scope:** This is a computational research project. Predictions are intended for research and educational use, not as a substitute for experimental metabolism assays or clinical decision-making.

## 2. Problem Statement

Given a molecule represented by its SMILES string, predict whether it is a substrate or non-substrate of a selected CYP450 isoform.

Formally, for a molecule \(x\) and enzyme \(t\), the model estimates:

\[
\hat{y}_{x,t} = f(x,t)
\]

where:

- \(x\) represents the molecular structure.
- \(t\) identifies the CYP450 isoform.
- \(y_{x,t}\in\{0,1\}\) is the observed binary label.
- \(\hat{y}_{x,t}\) represents the model's predicted score or class.

The project addresses three related questions:

1. How well can molecular fingerprints predict CYP450 substrate liability?
2. How do Logistic Regression, Random Forest, and multitask neural networks compare?
3. How does model performance change when evaluation uses scaffold-disjoint rather than compound-disjoint splits?

## 3. CYP450 Isoforms and Dataset

The project uses the Deep-PK CYP Substrates dataset, organized into train, validation, and test CSV files for the five CYP450 isoforms.

| CYP450 isoform | Compound–isoform records |
|---|---:|
| CYP1A2 | 366 |
| CYP2C19 | 261 |
| CYP2C9 | 1,033 |
| CYP2D6 | 916 |
| CYP3A4 | 1,551 |
| **Total** | **4,127 valid records** |

The audited source files contained 4,129 records before invalid structures were excluded. Two invalid SMILES entries were identified and excluded from modeling.

### Data preparation

The preprocessing workflow includes:

- Loading and auditing the source CSV files.
- Parsing molecular structures using RDKit.
- Removing records with invalid molecular representations from modeling.
- Canonicalizing valid SMILES for consistent compound identification.
- Preserving the original binary labels.
- Maintaining the distinction between observed labels and unmeasured CYP–compound combinations.
- Constructing compound-disjoint and scaffold-disjoint evaluation splits.

**Important:** An unmeasured CYP–compound combination is not automatically treated as a negative example. Missing labels must remain unobserved rather than being converted into negative labels.

The audited dataset contains **2,141 unique canonical compounds** across the five isoforms.

## 4. Project Architecture

The pipeline is organized into distinct stages:

```text
CYP450 Substrate Dataset
          |
          v
Data Audit and Validation
          |
          v
SMILES Parsing and Canonicalization
          |
          v
Morgan Fingerprint Generation
          |
          v
Compound / Scaffold-Based Splitting
          |
          v
+-------------------------------+
| Logistic Regression           |
| Random Forest                 |
| Multitask Neural Network      |
+-------------------------------+
          |
          v
Validation and Model Selection
          |
          v
Independent Test Evaluation
          |
          v
Metrics, Error Analysis,
and Applicability-Domain Analysis
```

This separation makes it easier to inspect data quality, compare models consistently, and evaluate the effect of different splitting strategies.

## 5. Molecular Representation

### Morgan fingerprints

Molecules are converted into fixed-length Morgan fingerprints using RDKit.

| Parameter | Configuration |
|---|---|
| Fingerprint type | Morgan / circular fingerprint |
| Radius | 2 |
| Fingerprint length | 2,048 bits |
| Input | Valid molecular SMILES |
| Output | Fixed-length binary molecular representation |

Morgan fingerprints encode local molecular environments and provide a computationally efficient representation for classical machine-learning models.

The same fingerprint configuration is used across the principal model comparisons to keep the representation consistent.

## 6. Models Evaluated

### 6.1 Logistic Regression

A linear classification baseline trained on molecular fingerprints.

**Why it is included:**
- Provides a computationally efficient baseline.
- Helps assess whether approximately linear decision boundaries are informative.
- Offers a useful reference for evaluating more complex models.

### 6.2 Random Forest

An ensemble of decision trees trained on the fingerprint features.

**Why it is included:**
- Captures nonlinear relationships between molecular substructures and labels.
- Provides an interpretable baseline for model comparison through feature and error analysis.
- Requires relatively little neural-network-specific tuning.

### 6.3 Multitask Neural Network

A shared-representation neural network designed to learn from multiple CYP450 prediction tasks.

Conceptually, the architecture consists of:

1. **Input layer:** 2,048-bit molecular fingerprint.
2. **Shared representation:** Learns molecular features used across CYP450 tasks.
3. **Task-specific outputs:** Predict substrate liability for the five isoforms.
4. **Masked training objective:** Computes the loss only for observed compound–isoform labels.

Multitask learning is motivated by the possibility that different CYP450 enzymes share useful structure–activity information.

However, shared learning does not guarantee improved generalization. The model must be compared against single-task and classical baselines under identical evaluation conditions.

### 6.4 Task-Balanced Multitask Learning

A task-balanced variant was also evaluated to investigate whether differences in dataset size across isoforms influence the learning objective.

This comparison is particularly relevant because the dataset contains substantially different numbers of records for different CYP450 enzymes.

## 7. Evaluation Strategy

Randomly splitting individual rows can place the same compound—or structurally related compounds—on both sides of an evaluation split. This can lead to overly optimistic estimates of generalization.

The project therefore examines two split strategies.

### Compound-disjoint (cold-compound) split

Compounds are assigned to partitions so that the same canonical compound does not appear in multiple partitions.

| Partition | Unique compounds |
|---|---:|
| Training | 1,498 |
| Validation | 321 |
| Test | 322 |
| **Total** | **2,141** |

Exact canonical-SMILES overlap between the partitions is zero.

### Scaffold-disjoint split

Compounds are grouped by molecular scaffold before assignment, reducing the chance that closely related scaffold families appear across training, validation, and test sets.

| Partition | Unique compounds |
|---|---:|
| Training | 1,584 |
| Validation | 274 |
| Test | 283 |
| **Total** | **2,141** |

The audited split contains 1,363 scaffold groups, including 141 acyclic compounds. Scaffold overlap between the partitions is zero.

Scaffold-disjoint evaluation is generally more challenging because the test set can contain molecular frameworks that are absent from training.

### Evaluation metrics

The project uses complementary metrics rather than relying on accuracy alone.

| Metric | Purpose |
|---|---|
| ROC-AUC | Measures ranking performance across classification thresholds |
| PR-AUC | Evaluates the precision–recall trade-off, particularly useful with class imbalance |
| F1-score | Balances precision and recall at a selected threshold |
| Matthews Correlation Coefficient (MCC) | Summarizes binary classification quality while accounting for all confusion-matrix outcomes |
| Accuracy | Measures the proportion of correctly classified examples |

ROC-AUC and PR-AUC evaluate score-based ranking, while F1 and MCC depend on classification decisions and the selected threshold. Metric definitions and threshold-selection procedures should therefore be considered when comparing results.

## 8. Results

The following results summarize the audited **compound-disjoint test evaluation**. Values are reported as recorded in the project experiments.

### Compound-disjoint test results

| Model | ROC-AUC | PR-AUC | MCC | F1-score | Accuracy |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.7643 | 0.7090 | **0.4023** | **0.6352** | 0.7007 |
| Random Forest | **0.7799** | **0.7400** | 0.3709 | 0.5828 | 0.7009 |
| Multitask Neural Network | 0.7551 | 0.7125 | 0.3966 | 0.6343 | **0.7087** |
| Task-Balanced Multitask Network | 0.7433 | 0.6839 | 0.3382 | 0.5927 | 0.6826 |

Bold values identify the best result within each metric column.

### Scaffold-disjoint test results

| Model | ROC-AUC | PR-AUC | MCC | F1-score |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.7379 | 0.6679 | **0.3664** | **0.6515** |
| Random Forest | **0.7637** | **0.7360** | 0.3132 | 0.5226 |
| Multitask Neural Network | 0.7122 | 0.6508 | 0.3234 | 0.6197 |

The reported scaffold-disjoint multitask accuracy was 0.6732.

### Key findings

1. **Random Forest achieved the highest macro ROC-AUC and PR-AUC** among the compared models in both reported split settings.
2. **Logistic Regression achieved the highest macro MCC and F1-score** among the reported compound-disjoint baselines and the highest reported scaffold-disjoint MCC and F1-score.
3. The original multitask network did not consistently outperform the classical baselines.
4. Task balancing did not improve aggregate performance in the reported compound-disjoint comparison.
5. Performance declined under scaffold-disjoint evaluation for the reported ROC-AUC comparisons, highlighting the challenge of generalizing to structurally different molecules.

These findings suggest that more complex architectures should be justified by measured gains rather than assumed to outperform simpler models.

**Interpretation caveat:** Aggregate results can conceal substantial differences between isoforms. In particular, the scaffold-disjoint CYP2C19 test subset contains only 23 compounds, making its individual evaluation results uncertain. Per-isoform metrics and sample counts should be consulted alongside macro averages.

## 9. Error Analysis and Applicability Domain

Beyond aggregate metrics, the workflow includes error analysis and a structural-familiarity analysis based on molecular fingerprints.

### Error analysis

Useful follow-up analyses include:

- False-positive and false-negative inspection.
- Per-isoform performance comparison.
- Class-distribution analysis.
- Examination of performance under compound-disjoint versus scaffold-disjoint evaluation.

Such analyses can reveal whether model errors are concentrated in particular CYP450 tasks or in compounds that differ substantially from the training data.

### Applicability-domain analysis

Maximum Tanimoto similarity to training compounds can be used as a simple structural-familiarity indicator.

For a test molecule \(x\), the maximum similarity is:

\[
S_{\max}(x)=\max_{z\in D_{\mathrm{train}}}
T(x,z)
\]

where \(T(x,z)\) is the Tanimoto similarity between molecular fingerprints.

Lower maximum similarity indicates that a molecule is less structurally similar to the available training compounds under the chosen representation.

**This score is not a calibrated uncertainty estimate.** It should not be interpreted as a probability that a prediction is correct. A reliable uncertainty framework would require additional validation.

## 10. Engineering Decisions

Several design choices were made to improve the reliability and interpretability of the experiments.

| Decision | Rationale |
|---|---|
| Canonicalize SMILES | Provides a consistent representation for compound-level grouping |
| Exclude invalid structures from modeling | Prevents invalid molecular representations from entering feature generation |
| Preserve missing labels | Avoids incorrectly converting unmeasured tasks into negative examples |
| Use fixed-length Morgan fingerprints | Enables consistent comparisons across classical and neural models |
| Compare multiple model families | Establishes whether added model complexity provides measurable value |
| Use compound-disjoint evaluation | Tests generalization to previously unseen compounds |
| Use scaffold-disjoint evaluation | Probes generalization to unseen structural frameworks |
| Report multiple metrics | Reduces dependence on a single performance measure |
| Inspect structural familiarity | Helps identify test compounds with limited similarity to training data |

## 11. Technology Stack

- **Python** — Data processing and experimentation.
- **RDKit** — Molecular parsing, canonicalization, and Morgan fingerprints.
- **scikit-learn** — Classical machine-learning models and evaluation metrics.
- **PyTorch** — Multitask neural-network experiments.
- **pandas / NumPy** — Data manipulation and numerical processing.
- **Jupyter Notebook** — Exploratory analysis and experiment documentation.
- **Matplotlib** — Visualization of results and evaluation metrics.

Only packages and versions actually used by the final implementation should be listed in the repository's dependency file.

## 12. Repository Structure

The GitHub repository may contain documentation and selected supporting artifacts without distributing the entire dataset, trained checkpoints, notebooks, or implementation.

A suggested documentation-focused layout is:

```text
CYP450_Substrate_Liability/
├── README.md
├── CYP450_Substrate_Liability_Detailed_Report.docx
├── Reference_Paper_Review_CYP450.docx
├── SUBMISSION_CHECKLIST.md
├── requirements.txt
├── docs/
│   ├── methodology.md
│   └── results_summary.md
├── results/
│   └── figures/
└── LICENSE
```

This is a suggested layout, not a claim that all listed files are currently present. Keep only files that you actually add. Do not include a dependency file or license until its contents are accurate.

## 13. Reproducibility and Setup

The complete experiment depends on the implementation, dataset files, and compatible Python packages. If these are not included in this public repository, the README documents the experimental setup but does not make the full pipeline independently executable.

### Environment setup

Create and activate a virtual environment:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies from the repository's dependency file:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

A compatible RDKit distribution, scikit-learn, PyTorch, pandas, and NumPy are needed for the corresponding stages of the pipeline.

### Running the project

The exact training commands depend on which implementation files are published. If notebooks and training scripts are intentionally withheld, readers cannot reproduce the reported metrics from this repository alone.

For a fully reproducible release, include:

- Data acquisition instructions and dataset provenance.
- Preprocessing and split-generation code.
- Model-training scripts or notebooks.
- Configuration files and random seeds.
- Dependency versions.
- Evaluation scripts and saved metric tables.

Do not invent execution commands or filenames for scripts that are not part of the published repository.

## 14. Limitations

This project has several important limitations:

- **Dataset coverage:** The number of labeled examples differs substantially across CYP450 isoforms.
- **Structural generalization:** Performance on scaffold-disjoint data is lower for several reported metrics than performance on compound-disjoint data.
- **Multitask performance:** The evaluated multitask networks did not consistently improve over classical baselines.
- **Small evaluation subsets:** Some isoform-specific test subsets, including scaffold-disjoint CYP2C19, are too small for strong conclusions.
- **Applicability domain:** Fingerprint similarity indicates structural familiarity, not calibrated predictive uncertainty.
- **External validation:** No independent external or experimental validation has been completed in the reported experiments.
- **Experimental uncertainty:** Binary substrate labels do not fully represent enzyme kinetics, metabolic rates, assay variability, or clinical exposure.
- **Reproducibility:** If data, code, or model checkpoints are withheld, readers cannot independently reproduce the reported results from the README alone.

These limitations motivate additional validation before the models can be used for practical compound prioritization.

## 15. Future Work

Potential extensions include:

1. Conducting external validation on an independently sourced CYP450 dataset.
2. Reporting confidence intervals and repeated-split variability.
3. Investigating class imbalance and task-specific threshold selection.
4. Improving per-isoform evaluation, particularly for tasks with limited test examples.
5. Comparing alternative molecular representations and graph-based neural networks.
6. Evaluating calibrated uncertainty and applicability-domain methods.
7. Adding explainability analyses to identify molecular substructures associated with predictions.
8. Testing whether multitask learning provides benefits under controlled, matched experimental conditions.

## 16. Scientific References

**1. CYP450 substrate prediction and multitask learning**

Fang et al. (2024). *Prediction of Cytochrome P450 Substrates Using the Explainable Multitask Deep Learning Models.* Chemical Research in Toxicology, 37(9), 1535–1548.

- [Publisher page and article details](https://pubs.acs.org/doi/10.1021/acs.chemrestox.4c00199)
- [DOI](https://doi.org/10.1021/acs.chemrestox.4c00199)

This work provides CYP450-specific context for multitask learning and explainable substrate prediction. Its results should not be compared directly with this project's metrics without accounting for dataset composition, split methodology, and evaluation protocol.

**2. Broader artificial intelligence in drug discovery**

Bender et al. (2026). *Artificial intelligence in drug discovery — what it is, where we stand and the path forward.* Nature Reviews Drug Discovery.

- [Publisher page](https://www.nature.com/articles/s41573-026-01496-2)
- [DOI](https://doi.org/10.1038/s41573-026-01496-2)

This review provides broader context for the opportunities and limitations of AI-driven drug discovery.

**3. Cheminformatics toolkit**

RDKit. *Open-source cheminformatics software.*

- [Official documentation](https://www.rdkit.org/docs/)

## 17. Project Takeaway

This project demonstrates an end-to-end approach to CYP450 substrate-liability modeling, from molecular representation and data validation to baseline comparisons and scaffold-aware evaluation.

The central engineering lesson is that **evaluation design matters as much as model architecture**. Random or compound-disjoint performance alone does not establish generalization to new chemical frameworks, and a more complex multitask model is not necessarily a better model.

By comparing multiple algorithms, preserving missing-label semantics, and evaluating both compound-disjoint and scaffold-disjoint splits, the project provides a foundation for more rigorous computational ADMET modeling.

---

**Disclaimer:** This project is for academic and research purposes only. It does not provide medical advice, establish clinical safety, or replace experimental drug-metabolism studies.
