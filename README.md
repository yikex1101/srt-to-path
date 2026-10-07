# Predicting Compound-Induced Pathway Responses from Molecular Structure

**Yike Xu | Independent Research Project | Summer 2026**

## Overview

Can molecular structure predict how a compound changes cellular signaling? This project investigates that question using LINCS L1000 gene-expression signatures from A549 human lung adenocarcinoma cells exposed to compounds at **10 µM for 24 hours**.

The analysis converts gene-expression signatures into transcriptionally inferred activity scores for 14 signaling pathways using PROGENy. After molecular-structure curation and adjustment for experimental block effects, Morgan fingerprints and pretrained CheMeleon embeddings are used to predict pathway responses for previously unseen molecular structures.

## Data

| Component | Description |
| --- | --- |
| Expression resource | LINCS Connectivity Map L1000, 2020 beta release, Level 5 |
| Cellular context | A549 human lung adenocarcinoma cells |
| Treatment condition | Nominal 10 µM dose, 24-hour exposure |
| Quality criteria | Small-molecule perturbations that passed quality control and were designated high quality |
| Final expression signatures | 2,355 |
| LINCS compound identifiers | 1,498 |
| Distinct standardized molecular structures | 1,494 |
| Prediction targets | 14 PROGENy pathway scores |

The pathways are Androgen, EGFR, Estrogen, Hypoxia, JAK–STAT, MAPK, NF-κB, PI3K, TGF-β, TNF-α, TRAIL, VEGF, WNT, and p53.

## Analysis Workflow

1. **Curate molecular structures.** Recover missing or incomplete SMILES annotations from earlier LINCS records and PubChem, standardize structures using RDKit and `chembl_structure_pipeline`, and audit stereochemical annotations. Assign a common structure identifier to records with identical standardized structures.
2. **Infer pathway responses.** Apply PROGENy through decoupler using the 100 highest-ranked responsive genes per pathway to produce a 14-dimensional response vector for each expression signature.
3. **Split at the structure level.** Allocate 1,045 structures to training, 224 to validation, and 225 to testing. Keep all observations of an identical standardized structure in the same partition.
4. **Adjust experimental block effects.** Average signatures within each structure–block combination and estimate pathway-specific block effects using a ridge-regularized additive model. Estimate adjustments and select shrinkage penalties using training data alone, freeze the adjustments, and average adjusted responses equally across available blocks for each structure.
5. **Encode molecular structures.** Generate chirality-aware Morgan fingerprints with radius 2 and 2,048 bits, and 2,048-dimensional embeddings from the frozen pretrained CheMeleon encoder.
6. **Fit and evaluate predictive models.** Compare linear and nonlinear regression methods using held-out performance. Standardize pathway responses and CheMeleon features using training-set statistics.

## Predictive Results

| Molecular representation | Model | Mean test R² | Test sRMSE |
| --- | --- | ---: | ---: |
| CheMeleon | RBF kernel ridge regression | **0.150** | **0.943** |
| CheMeleon | RBF support-vector regression | 0.131 | 0.952 |
| CheMeleon | Feedforward neural-network ensemble | 0.115 | 0.967 |
| Morgan | Tanimoto kernel ridge regression | 0.102 | 0.969 |
| Morgan | Ridge regression | 0.079 | 0.983 |
| CheMeleon | Ridge regression | 0.052 | 1.002 |

Mean test R² is the average across the 14 pathways. Standardized root mean squared error (sRMSE) scales prediction errors by each pathway's training-set standard deviation; lower values indicate better performance.

The best-performing approach combined CheMeleon embeddings with pathway-specific RBF kernel ridge regression. Its test R² varied across pathways, reaching **0.361 for MAPK**, **0.303 for EGFR**, and **0.222 for p53**. Overall performance was modest, and most response variability remained unexplained by the evaluated models.

## Response Reliability and Activity Cliffs

- **Cross-block reliability:** Conditional intraclass correlation coefficients (ICCs), with confidence intervals from 1,000 parametric-bootstrap repetitions, ranged from 0.413 for Androgen to 0.847 for MAPK. More reproducible pathways generally had better predictive performance, although high reliability did not guarantee accurate structure-based prediction.
- **Similarity–response analysis:** Among 237 structure pairs with Morgan Tanimoto similarity of at least 0.80, no pair exceeded the global response-distance threshold. However, 23 pairs exceeded a pathway-specific threshold for at least one pathway. These candidate activity cliffs indicate local response differences under the chosen criteria; they do not establish that activity cliffs explain the overall prediction difficulty.

These diagnostics used the pooled dataset to describe response properties and were separate from held-out predictive evaluation.

## Limitations and Future Directions

- Results apply to one cell line, dose category, and exposure duration.
- PROGENy scores are inferred from transcriptional responses rather than direct biochemical measurements of pathway activity.
- The project does not evaluate cell viability, growth inhibition, or anticancer efficacy.
- Evaluation uses a single structure-level split. Related molecular analogues may appear across partitions, and generalization to unfamiliar scaffolds or external datasets remains untested.
- Some compounds retain unresolved stereochemical annotations, introducing uncertainty in the molecular inputs.

Future work could examine repeated or nested evaluation, scaffold-based and external testing, learning curves, sensitivity to pathway-score construction, and additional information about compound–cell interactions.

## Selected References

- Subramanian et al. (2017). *A Next Generation Connectivity Map: L1000 Platform and the First 1,000,000 Profiles.* [Cell](https://doi.org/10.1016/j.cell.2017.10.049).
- Schubert et al. (2018). *Perturbation-response genes reveal signaling footprints in cancer gene expression.* [Nature Communications](https://doi.org/10.1038/s41467-017-02391-6).
- Bento et al. (2020). *An open source chemical structure curation pipeline using RDKit.* [Journal of Cheminformatics](https://doi.org/10.1186/s13321-020-00456-1).
- Burns et al. (2026). *Deep Learning Foundation Models from Classical Molecular Descriptors.* [arXiv](https://arxiv.org/abs/2506.15792).

See the full report for detailed methods, pathway-specific results, figures, and additional references.
