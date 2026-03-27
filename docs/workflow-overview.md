# Workflow Overview

## Pipeline Stages (Logical Order)

1. `src/preprocess/bam2vcf.sh`: call SNVs from multiple BAM files and remove known RNA editing sites.
2. `src/preprocess/snv2barcode.py`: build per-sample and merged barcode×SNV matrices.
3. `src/train/pre_train.py`: pretrain RNA backbone and prepare aligned finetuning data.
4. `src/train/snv_eff.py`: train SNV perturbation model and export attention/score outputs.

## Stage Handoffs (Critical)

| Upstream Stage | Key Output | Downstream Consumer |
|---|---|---|
| `bam2vcf` | `{sample}.f1804q20.no_rna_editing.vcf` | `snv2barcode` |
| `snv2barcode` | `all_samples_merged_barcode_snv_matrix.h5ad` | `snv_eff` |
| `pre_train` | `finetune_aligned.h5ad`, `rna_backbone_pretrained.pt` | `snv_eff` |

## Core Bridge Files

- `all_samples_merged_barcode_snv_matrix.h5ad` (from `snv2barcode`, consumed by `snv_eff`)
- `finetune_aligned.h5ad` (from `pre_train`, auto-loaded by `snv_eff`)
- `rna_backbone_pretrained.pt` (from `pre_train`, optionally loaded by `snv_eff`)
