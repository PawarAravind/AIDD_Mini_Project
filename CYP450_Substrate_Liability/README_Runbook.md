# CYP450 Substrate Liability — README / Runbook

## Goal
Classify substrate/non-substrate labels for CYP1A2, CYP2C19, CYP2C9, CYP2D6 and CYP3A4 using Deep-PK CYP substrate data. Compare single-task models with shared multi-task learning under global cold-compound and scaffold-disjoint evaluation.

## Important note
This runbook is based on the reported project structure and result summary. Verify exact local notebook names, dependency versions and file paths before submission. This documentation ZIP contains reports and figures, not the full local source code, raw data, notebooks or checkpoints.

## Setup (Windows PowerShell)
1. Open a terminal at the project root.
2. `py -m venv .venv`
3. ` .\.venv\Scripts\Activate.ps1`
4. `python -m pip install --upgrade pip`
5. `pip install -r requirements.txt` (use the project’s existing requirements/lock file; do not blindly upgrade packages).
6. Verify imports: `python -c "import rdkit, pandas, sklearn, torch; print(rdkit.__version__, pandas.__version__, sklearn.__version__, torch.__version__)"`

## Notebook order
1. `01_data_audit.ipynb` — inspect data, schema, class counts and invalid SMILES.
2. `02_cleaning_and_splits.ipynb` — canonicalize valid structures, preserve missing labels, create cold/scaffold splits.
3. `03_baseline_models.ipynb` — train Logistic Regression and Random Forest baselines.
4. `04_multitask_model_plan.ipynb` — review the shared model and label-mask strategy.
5. `05_multitask_model.ipynb` — train/evaluate the shared MTL model if this notebook exists in the local version.

Restart the kernel and use “Run All” for each notebook in order. Run from project root so relative paths resolve. Do not run later notebooks against stale intermediate files.

## Data logic to preserve
- Keep raw Deep-PK files unchanged.
- Log invalid SMILES and exclude them from modeling rather than silently repairing them.
- Canonicalize structures with RDKit.
- Missing isoform labels are unobserved, not negative; mask them in multi-task loss.
- Cold split: each canonical SMILES belongs to one partition across all isoforms.
- Scaffold split: no Bemis–Murcko group crosses partitions; document handling of acyclic structures.

## Smoke tests before submission
- Import test passes in the clean environment.
- Every included SMILES parses; invalid-row audit matches exclusions.
- Labels remain binary and missing labels are not filled with zero.
- Cold split has zero exact canonical-SMILES overlap across train/validation/test.
- Scaffold split has zero shared scaffold groups across partitions.
- Each isoform has ROC-AUC, PR-AUC, MCC and F1 for both protocols.
- Macro ROC-AUC equals the arithmetic mean of five valid isoform AUCs.
- Checkpoints load with matching architecture and preprocessing.
- Full notebook run regenerates outputs without stale kernel state.

## Reported cold-compound macro metrics
| Model | ROC-AUC | PR-AUC | MCC | F1 | Accuracy |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.7643 | 0.7090 | 0.4023 | 0.6352 | 0.7007 |
| Random Forest | 0.7799 | 0.7400 | 0.3709 | 0.5828 | 0.7009 |
| Original multi-task MLP | 0.7551 | 0.7125 | 0.3966 | 0.6343 | 0.7087 |
| Task-balanced MTL | 0.7433 | 0.6839 | 0.3382 | 0.5927 | 0.6826 |

## Reported scaffold-split macro metrics
| Model | ROC-AUC | PR-AUC | MCC | F1 |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.7379 | 0.6679 | 0.3664 | 0.6515 |
| Random Forest | 0.7637 | 0.7360 | 0.3132 | 0.5226 |
| Multi-task MLP | 0.7122 | 0.6508 | 0.3234 | 0.6197 |

Recompute these values from actual metric/prediction CSVs before final submission.

## References
- Bender et al. (2026), *Nature Reviews Drug Discovery*: https://doi.org/10.1038/s41573-026-01496-2. Official 2025 JIF: 91.2, https://www.nature.com/nrd/journal-impact
- Fang et al. (2024), *Chemical Research in Toxicology*: https://doi.org/10.1021/acs.chemrestox.4c00199
- Deep-PK data portal: https://biosig.lab.uq.edu.au/deeppk/data/

## Limitations
No independent external validation or wet-lab CYP assay validation is documented in the current results summary. Fingerprint importance is not causal; Tanimoto similarity is not calibrated uncertainty. Do not present predictions as clinical advice or validated drug-interaction risk.
