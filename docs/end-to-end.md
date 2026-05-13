# End-to-End Execution Example

## 1. End-to-End Commands

```bash
# Step 1: Preprocessing
# 1.1 BAM -> VCF
prismsnv bam2vcf \
  --outer-jobs 6 \
  --inner-threads 4 \
  --reference /path/to/genome.fa \
  --varscan-jar /path/to/VarScan.jar \
  --rna-edit-bed /path/to/RNA_editing.bed \
  --out-dir ./snv_call_out \
  --bam-files /path/to/sample1.bam /path/to/sample2.bam

# 1.2 VCF/BAM/CB -> merged barcode×SNV h5ad
prismsnv snv2barcode /path/to/snv2barcode_config.yaml

# Step 2: Training
# 2.1 Pretrain RNA backbone
prismsnv pre_train -y /path/to/train_config.yaml

# 2.2 Train and score SNV perturbation effects
prismsnv snv_effect -y /path/to/train_config.yaml
```

## 2. Checkpoints After Each Step

### After Step 1

- Each sample should have: `*.f1804q20.no_rna_editing.vcf`
- Expected output: `all_samples_merged_barcode_snv_matrix.h5ad`

### After Step 2

- Expected output: `finetune_aligned.h5ad`
- Expected output: `rna_backbone_pretrained.pt`
- Expected output: `snv_perturbation_model.pt`
- Expected output: attention/score CSV files

## 3. Common Bottlenecks

- Step 1 is slow: usually large BAM files or storage I/O bottlenecks; reduce `OUTER_JOBS` first.
- Step 1 is also where a very large SNV union or overly aggressive `threads` in `prismsnv snv2barcode` can slow the run down.
- Step 2 OOM: reduce `batch_size_train` and `pair_chunk` first.

Before running Step 2, confirm that `all_samples_merged_barcode_snv_matrix.h5ad` from preprocessing is available.
