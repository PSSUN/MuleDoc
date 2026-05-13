# Workflow Overview

## Data Preparation

Only `BAM files` from single-cell RNA-seq read alignment are required to run the full workflow on your own private data. Whenever possible, use full-length sequencing data so that more genomic regions are covered and more SNV information can be obtained.

## Pipeline Stages (User-Facing 2-Step View)

1. **Step 1: Preprocessing**
   - `prismsnv bam2vcf`: call SNVs from multiple BAM files and remove known RNA editing sites.
   - `prismsnv snv2barcode`: build per-sample and merged barcode×SNV matrices.
2. **Step 2: Training**
   - `prismsnv pre_train`: pretrain RNA backbone and prepare aligned finetuning data.
   - `prismsnv snv_effect`: train SNV perturbation model and export attention/score outputs.

## Stage Handoffs (Critical)

| Upstream Stage | Key Output | Downstream Consumer |
|---|---|---|
| `bam2vcf` | `{sample}.f1804q20.no_rna_editing.vcf` | `snv2barcode` |
| `snv2barcode` | `all_samples_merged_barcode_snv_matrix.h5ad` | `snv_effect` |
| `pre_train` | `finetune_aligned.h5ad`, `rna_backbone_pretrained.pt` | `snv_effect` |

## Core Bridge Files

- `all_samples_merged_barcode_snv_matrix.h5ad` (from `snv2barcode`, consumed by `snv_effect`)
- `finetune_aligned.h5ad` (from `pre_train`, auto-loaded by `snv_effect`)
- `rna_backbone_pretrained.pt` (from `pre_train`, optionally loaded by `snv_effect`)
