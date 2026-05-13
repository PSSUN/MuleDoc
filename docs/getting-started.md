# Getting Started

## 1. Environment Requirements

### 1.1 Recommended Software Versions

| Component | Recommended Version | Notes |
|---|---|---|
| Python | 3.10+ | Keep versions consistent across team environments |
| Bash | system/Git Bash/WSL | Required by `prismsnv bam2vcf` |
| samtools | 1.10+ | Used by `prismsnv bam2vcf` for filtering/indexing/mpileup |
| bedtools | 2.29+ | Used to remove known RNA editing sites |
| VarScan | 2.x | Matches `java -jar VarScan.jar` invocation |
| Java | 8+ | Required to run VarScan |
| awk/gawk | system/gawk | Used by the BAM-to-VCF shell pipeline |

### 1.2 Python Environment Setup

```bash
conda create -n prismsnv python=3.10 -y
conda activate prismsnv
pip install --upgrade pip
pip install -e .
```

Run the install command from the PrismSNV repository root. Use `pip install .` instead of `pip install -e .` if you want a non-editable install.

`prismsnv bam2vcf` also needs command-line tools available in `PATH`:

```bash
conda install -c conda-forge -c bioconda bash samtools bedtools openjdk gawk -y
```

You also need a VarScan JAR file and should pass it at runtime with `--varscan-jar`.

After installation, confirm that the package command is available:

```bash
prismsnv --help
prismsnv bam2vcf --help
prismsnv snv2barcode --help
prismsnv pre_train --help
prismsnv snv_effect --help
```

### 1.3 External Tool Sanity Check

```bash
bash --version
samtools --version
bedtools --version
java -version
awk --version
```

Proceed once these commands return version information.

---

## 2. Required Inputs

The full workflow uses two groups of inputs.

### 2.1 Preprocessing Inputs

| Input | Purpose | Required |
|---|---|---|
| `reference.fa` | Reference genome | Yes |
| `RNA_editing.bed` | Known RNA editing sites; you can download [here](https://doi.org/10.6084/m9.figshare.30460229) | Yes |
| `*.bam` + `*.bai` | Per-sample alignment data | Yes |
| sample-level VCF | SNV source for `prismsnv snv2barcode` | Yes |
| barcode file | Build barcode×SNV matrix | Yes |

### 2.2 Training Inputs

| Input | Source | Required |
|---|---|---|
| `all_samples_merged_barcode_snv_matrix.h5ad` | Output from `prismsnv snv2barcode` | Yes (`prismsnv snv_effect`) |
| `ann_csv` | SNV annotation table | Optional (`prismsnv snv_effect`) |

Notes:

- `prismsnv pre_train` consumes the raw RNA AnnData paths configured in YAML and generates the aligned training artifacts itself; `pretrain_adata.h5ad` and `finetune_adata.h5ad` should not be listed here as standalone user-prepared training inputs.
- `prismsnv snv_effect` reads the RNA-side artifact from `result_folder/finetune_aligned.h5ad`, which is produced by `prismsnv pre_train`.

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
# If you start from raw BAM files, run BAM-to-VCF calling first.
prismsnv bam2vcf \
  --outer-jobs 6 \
  --inner-threads 4 \
  --reference /path/to/genome.fa \
  --varscan-jar /path/to/VarScan.jar \
  --rna-edit-bed /path/to/RNA_editing.bed \
  --out-dir ./snv_call_out \
  --bam-files /path/to/sample1.bam /path/to/sample2.bam

prismsnv snv2barcode /path/to/snv2barcode_config.yaml

# Step 2: Training
# 2.1 Pretrain RNA backbone
prismsnv pre_train -y /path/to/train_config.yaml

# 2.2 Train and score SNV perturbation effects
prismsnv snv_effect -y /path/to/train_config.yaml
```

This is the full user-facing flow: preprocessing first, then training.

On Windows, run `prismsnv bam2vcf` in an environment where Bash can access the input files, such as WSL or Git Bash.

---

## 5. Pre-run Checklist

- Every BAM has a matching index (`.bai`)
- Chromosome naming is consistent between `reference.fa` and `RNA_editing.bed` (`chr1` vs `1`)
- All paths in `train_config.yaml` are valid
- Output directories are writable

This checklist reduces first-run failures significantly.
