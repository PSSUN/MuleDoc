# Getting Started

## 1. Environment Requirements

### 1.1 Recommended Software Versions

| Component | Recommended Version | Notes |
|---|---|---|
| Python | 3.10+ | Keep versions consistent across team environments |
| samtools | 1.10+ | Used by `bam2vcf.sh` for filtering/indexing/mpileup |
| bedtools | 2.29+ | Used to remove known RNA editing sites |
| VarScan | 2.x | Matches `java -jar VarScan.jar` invocation |
| Java | 8+ | Required to run VarScan |

### 1.2 Python Environment Setup

```bash
python -m venv .venv
. .venv/Scripts/activate
pip install --upgrade pip
pip install -e .
```

### 1.3 External Tool Sanity Check

```bash
samtools --version
bedtools --version
java -version
```

Proceed once these commands return version information.

---

## 2. Required Inputs

The full workflow uses two groups of inputs.

### 2.1 Preprocessing Inputs

| Input | Purpose | Required |
|---|---|---|
| `reference.fa` | Reference genome | Yes |
| `RNA_editing.bed` | Known RNA editing sites | Yes |
| `*.bam` + `*.bai` | Per-sample alignment data | Yes |
| sample-level VCF | SNV source for `snv2barcode.py` | Yes |
| barcode file | Build barcode×SNV matrix | Yes |

### 2.2 Training Inputs

| Input | Source | Required |
|---|---|---|
| `all_samples_merged_barcode_snv_matrix.h5ad` | Output from `snv2barcode` | Yes (`snv_effect`) |
| `ann_csv` | SNV annotation table | Optional (`snv_effect`) |

Notes:

- `pre_train.py` consumes the raw RNA AnnData paths configured in YAML and generates the aligned training artifacts itself; `pretrain_adata.h5ad` and `finetune_adata.h5ad` should not be listed here as standalone user-prepared training inputs.
- `snv_effect.py` reads the RNA-side artifact from `result_folder/finetune_aligned.h5ad`, which is produced by `pre_train.py`.

Important for pretraining batch-aware behavior:

- Batch metadata should be stored in `pretrain_adata.obs[batch_key]`.
- Default key is `batch` (i.e., `pretrain_adata.obs["batch"]`).
- If you use another key, set `pre_train.training.batch_key` in YAML.

---

## 3. Recommended Directory Layout

```text
project-root/
├─ data/
│  ├─ bam/
│  ├─ reference/
│  ├─ barcode/
│  └─ anndata/
├─ result/
└─ config/
```

---

## 4. Minimal Execution Order

```bash
# Step 1: Preprocessing
# If you start from raw BAM files, run bam2vcf.sh first.
python src/preprocess/snv2barcode.py src/preprocess/snv2barcode_config.yaml

# Step 2: Training
# 2.1 Pretrain RNA backbone
python src/train/pre_train.py -y src/train/train_config.yaml

# 2.2 Train and score SNV perturbation effects
python src/train/snv_effect.py -y src/train/train_config.yaml
```

This is the full user-facing flow: one preprocessing step, then one training step.

---

## 5. Pre-run Checklist

- Every BAM has a matching index (`.bai`)
- Chromosome naming is consistent between `reference.fa` and `RNA_editing.bed` (`chr1` vs `1`)
- All paths in `train_config.yaml` are valid
- Output directories are writable

This checklist reduces first-run failures significantly.
