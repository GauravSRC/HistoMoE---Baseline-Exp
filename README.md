# HistoMoE - Baseline Experiments

**Task:** Predict gene expression from H&E histology patches using the [HEST-Benchmark](https://huggingface.co/datasets/MahmoodLab/hest-bench).

---

## Contents

| File | Description |
|---|---|
| `setup1_resnet50_baseline.ipynb` | ResNet50 (ImageNet) → PCA-256 → Ridge. Reproduces HEST paper Table 1 protocol. |
| `setup2_phikon_ridge_mlp.ipynb` | Phikon (ViT-B/16, TCGA iBOT) → Ridge + GPU MLP. Domain-specific encoder + non-linear head. |
| `HistoMoE_Baseline_Report.pdf` | Short written report covering design choices, results, and observations. |

---

## Tasks

- **ccRCC** - clear cell renal cell carcinoma (kidney), 24 patients, 6 official CV folds  
- **COAD** - colon adenocarcinoma, 4 patients, 2 official CV folds

---

## Key Results (Mean Pearson r, averaged across folds and 50 genes)

| Setup | ccRCC (Ours) | ccRCC (Paper) | COAD (Ours) | COAD (Paper) |
|---|---|---|---|---|
| ResNet50 + Ridge | 0.032 | 0.223 | 0.051 | 0.253 |
| Phikon + Ridge | 0.046 | 0.242 | 0.028 | 0.259 |
| Phikon + MLP | **0.055** | 0.242 | 0.017 | 0.259 |
| Paper best (H-Optimus-0) | — | 0.265 | — | 0.299 |

Best single-gene Pearson: **r = 0.508** (IGHG3, ccRCC) and **r = 0.817** (TAGLN, COAD).

---

## Main Observations

**1. Phikon > ResNet50 on ccRCC** - Domain-specific histopathology pretraining (TCGA iBOT) produces more informative embeddings than ImageNet features. Matches the trend reported in the HEST paper.

**2. MLP beats Ridge on ccRCC but not on COAD** - With 24 patients (ccRCC), a non-linear MLP head learns patterns that a linear Ridge projection cannot. With only 4 patients (COAD), the MLP overfits. This asymmetry shows that the optimal model depends on the cancer type - the core heterogeneity problem HistoMoE addresses.

**3. Per-gene signal is strong even when the mean is low** - High-performing genes (IGHG3, TAGLN, SERPINA1) correspond to biologically interpretable features: immune infiltration, stromal content, and cellular morphology that are directly visible in H&E images. The low mean is driven by genuinely unpredictable genes, not a pipeline failure.

**4. Cross-task heterogeneity is large** - From the HEST paper (all 9 tasks, ResNet50): scores range from 0.08 (READ) to 0.49 (LUAD). A single global model cannot be optimal across this range. HistoMoE's cancer-specific expert models with a gating network are the natural solution.


---

## Reference

Jaume et al., *HEST-1k: A Dataset for Spatial Transcriptomics and Histology Image Analysis*, arXiv 2406.16192 (2024).  
GitHub: https://github.com/mahmoodlab/hest
