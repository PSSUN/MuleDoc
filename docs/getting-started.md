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

If you only want to build documentation:

```bash
pip install -r docs/requirements.txt
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
| `pretrain_adata.h5ad` | RNA pretraining data | Yes (`pre_train`) |
| `finetune_adata.h5ad` | RNA finetuning data | Yes (`pre_train`) |
| `all_samples_merged_barcode_snv_matrix.h5ad` | Output from `snv2barcode` | Yes (`snv_effect`) |
| `ann_csv` | SNV annotation table | Yes (`snv_effect`) |

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
# 1) Preprocess and build merged SNV matrix
python src/preprocess/snv2barcode.py src/preprocess/snv2barcode_config.yaml

# 2) Pretrain RNA backbone
python src/train/pre_train.py -y src/train/train_config.yaml

# 3) Train and score SNV perturbation effects
python src/train/snv_effect.py -y src/train/train_config.yaml
```

If you start from raw BAM files, run `bam2vcf.sh` before step 1.

---

## 5. Pre-run Checklist

- Every BAM has a matching index (`.bai`)
- Chromosome naming is consistent between `reference.fa` and `RNA_editing.bed` (`chr1` vs `1`)
- All paths in `train_config.yaml` are valid
- Output directories are writable

This checklist reduces first-run failures significantly.
